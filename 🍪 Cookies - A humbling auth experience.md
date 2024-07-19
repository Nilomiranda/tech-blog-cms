
While working on a NextJS 14 project, I had to implement the user flow to refresh an authentication token using the refresh token.


There were some overlooked concepts that made my life harder when I could've solved the problem much faster if I paid attention to the fundamentals of web development and **specially how cookie works**.


As it's a NextJS 14 project, I was making use of the latest and shiniest techniques available within the stack, such as middleware, server actions and route handlers.


My middleware file was already handling the logic to check if a valid session was available and would redirect the user accordingly or just let him access the route he was intending to access.


However, there was this important step of refreshing the authentication token if a valid refresh token was available.


At first, looks like a simple solution was on the way. Just implement a server action to handle the logic to obtain a new auth token and call this server action, if needed, from my middleware file.


**The problem** is that Next's middleware runs on the [edge runtime](https://nextjs.org/docs/pages/api-reference/edge) and since server actions will inherit the runtime from which they are used on, this means that my server action responsible for taking care of the logic to refresh the authorization token would also be running under the same runtime.


The problem is, that I was using the `jsonwebtoken` package to handle token signing and validation, and some internal code from that package relies on APIs available only in the node runtime.


I could've used other jwt packages, such as [jose](https://github.com/panva/jose) which has operability in [Nextjs's edge runtime](https://github.com/panva/jose?tab=readme-ov-file#supported-runtimes) but I was using another package built on top of the RPC framework which also relied on node so that wouldn't solve my problem.


**The solution** I came up with was that I could maintain my server action, but then implement a [route handler](https://nextjs.org/docs/app/building-your-application/routing/route-handlers) to call it and handle the response back. That would solve the problem of runtimes since [**by default** route handlers run in the node runtime](https://nextjs.org/docs/app/building-your-application/routing/route-handlers#edge-and-nodejs-runtimes), although you can change that if you want to.


That's what I did, implemented the call from a route handlers and inside my middleware I would send a fetch request to that same route and handle the response I get from the API call.


All seemed to be working well at first, the request was going through, I could log the old tokens and the new ones, to ensure that the refresh flow was working, but no matter how many times I refresh the auth token, the cookies that were supposed to hold the auth token were never updated, they were always missing.


This is where I was overlooking an essential web fundamental concept. 


Next middleware runs on the server, I was sending the request to refresh the auth token and getting the new token, while also setting the new cookie, all from within the context of running on the server, **but never on the client**.


This means that in between the middleware handler sending the request to update the auth token, receiving the response and redirecting, or allowing the user to access a given route, the new auth token and the set of the cookie would be lost, cause that was never happening on the client. Basically, **the instructions to set the cookie stopped on the server side and never reached the client**.


The solution was now obvious, of course, after banging the head for "hours", at the end of the day, the solution will always look obvious.


Instead of leaving the middleware to handle that request to refresh the token, I would pass the responsibility to a component high in the components tree so the instruction coming from the response header to set the cookie would be available in the browser's context, correctly setting and updating the cookies as I needed. 

