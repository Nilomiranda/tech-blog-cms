When working with NextJS, there is a high chance that at some point you'll be writing some small server side code, if not extensively, depending on how your application is structure.

When going through the build process, you may come across an error that looks like this:

```bash
Dynamic server usage: Route /api/google-auth couldn't be rendered statically...
```

that will probably be followed by a reason, like:

```
...because it accessed `nextUrl.searchParams`
```

While NextJS [documentation covers the reasons](https://nextjs.org/docs/messages/dynamic-server-error) to why this error is thrown at build time and possible ways to fix it, it may not cover some specific "use-cases" or why exactly the error is being thrown.

In my case, I had a route handler calling a function that looks like this:

```typescript 
import { redirect } from 'next/navigation'  
import { NextResponse, NextRequest } from 'next/server'  
  
export async function handleAuthRedirect(req: NextRequest) {  
  try {  
    const state = req.nextUrl.searchParams.get('state')  
    const token = req.nextUrl.searchParams.get('token')  

	// [...rest of the code...]
	
    return response  
  } catch (error) {  
    captureError(error as Error)  
    redirect('/login?errorMessage="We encountered an issue while logging you in"')  
  }  
}
```


The code looks fine, but something with the usage of `nextUrl.searchParams` was causing this.

What is happening, according to [this SO answer](https://stackoverflow.com/a/78010468) is that Nextjs will go through this file and notice the call to `nextUrl.searchParams` and determine that now this needs to by dynamically rendered since the code is dependant on URL params, which can't be determined ahead of time.

Internally, to opt into dynamic render, Next will throw an error when a dynamic value is needed, catch it and the handle the logic for dynamic rendering.

So what's causing the problem here, is that I'm accessing `nextUrl.searchParams` inside a `try...catch` block but in the `catch` I'm not re-throwing the error anymore, removing Nextjs's capability of determining if I'm relying on dynamic values.

Two possible solutions:

You can just re-throw the error in your `catch` block:

```typescript
try {
	// ...code
} catch (err) {
	// ... do whatever you need to
	throw err;
}
```

Or, you can simply remove such access to dynamic values from the `try...catch` block.

```typescript
const state = req.nextUrl.searchParams.get('state')  
const token = req.nextUrl.searchParams.get('token')

try {
	// ...code
} catch (err) {
	// ... do whatever you need to
}
```
