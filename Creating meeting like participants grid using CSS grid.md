
Let's imagine a situation where you are creating a tool that could serve as a virtual meeting space, like google-meet or zoom, or if you just need to create a small grid of participants. 

This grid will have a have a variable number of participants and you want to render them an even columns, displaying each participant card in a nice 16:9 aspect ratio.

For this POC, we will have three main files.

- App.tsx
	- The application main's component
	
- Participants.tsx
	- The component responsible for rendering the list of participants
	
- Participant.tsx
	- The component responsible for rendering a single participant


With a little less importance, as you can style your application as you see fit, we will have three style files for each component

- styles.css
- participants.css
- participant.css


Since the main `App` component is just there to be the wrapper of our application, let's no focus to much on it. 

Here's the component structure:

```jsx
import Participants from "./components/Participants";
import "./styles.css";

export default function App() {
	const participants = [
		{ name: "Danilo" },
		{ name: "Danilo" },
		{ name: "Danilo" },
		{ name: "Danilo" },
		{ name: "Danilo" },
		{ name: "Danilo" },
		{ name: "Danilo" },
		{ name: "Danilo" },
	]; // Just a simple list of participants

	return (
		<div className="App">
			<h1>Participants</h1>	
			<Participants participants={participants} />
		</div>
	);
}
```

And here are the styles for `styles.css`

```css
.App {
	font-family: sans-serif;
	text-align: center;
	display: flex;
	flex-direction: column;
	align-items: center;
	width: 100%;
}
```

The `Participant` component, which will be responsible for rendering a single participant, is also pretty straightforward:

```jsx
import "./participant.css";

type Participant = {
	name: string;
};
  
export default function Participant({ name }: Participant) {
	return (
		<div className="participant-card">
			<strong>{name}</strong>
		</div>
	);
}
```

A few styles on `participant.css` to give a nice look:

```css
.participant-card strong {
	color: #fff;
	background: lightblue;
	padding: 10px 20px;
	aspect-ratio: 16 / 9;
	border-radius: 4px;
	display: flex;
	align-items: center;
	justify-content: center;
}
```

Inside `Participants` component, responsible for rendering the list of participants, is what we are interested in:

```jsx
import Participant from "./Participant";
import "./participants.css";

const MAX_COLUMNS = 3;

type ParticipantsProps = {
	participants: { name: string }[];
};

export default function Participants({ participants }: ParticipantsProps) {
	const participantsCount = participants.length;
	const gridMaxColumns = participantsCount > MAX_COLUMNS ? MAX_COLUMNS : participantsCount;
	  
	const styleConfig = {
		gridTemplateColumns: `repeat(${gridMaxColumns}, 1fr)`,
	};
	  
	return (
		<div style={styleConfig} className="participants-grid">
			{participants.map((participant, index) => (
				<Participant key={index} name={participant.name} />
			))}
		</div>
	);
}
```

Not a lot going on `participants.css` too, just some basic styling for illustration purposes:

```css
div.participants-grid {
	display: grid;
	width: 100%;
	gap: 8px;
	padding: 4px;
}
```

The important property here would be `grid`, setting `participants-grid` to be a `grid` component. `width`, `gap` and `padding` doesn't really matter in this case, and you can to whatever value you may need.

Now let's go back to `Participants` component.

Inside the component's definition, we start by getting the participants count and determining the max number of columns we will have in our grid?

```javascript
const participantsCount = participants.length;
const gridMaxColumns = participantsCount > MAX_COLUMNS ? MAX_COLUMNS : participantsCount;
```

Remember, we set our  `MAX_COLUMNS` to `3` and what we are doing above is checking if the number of participants is greater than the max columns allowed.  If it is, then we set the grid columns to have a maximum of 3 columns, if it isn't we set our grid columns to the number of participants.

Then, we just created a `style` object that we will use with the `style` prop of our `div`.

```jsx
const styleConfig = {
	gridTemplateColumns: `repeat(${gridMaxColumns}, 1fr)`,
};

return (
	<div style={styleConfig} className="participants-grid">
		{participants.map((participant, index) => (
			<Participant key={index} name={participant.name} />
		))}
	</div>
)
```

And we end up with something like this:

![Participants grid](https://github.com/Nilomiranda/tech-blog-cms/assets/25915040/0975b685-f941-49f2-b410-c408b7323406)

See that if we have different number of participants, the grid adjusts accordingly, while keeping the aspect ratio of 16:9

![CleanShot 2024-06-02 at 14 45 44](https://github.com/Nilomiranda/tech-blog-cms/assets/25915040/ff2f4dca-14f1-4a4e-8669-6c5bb1df2807)