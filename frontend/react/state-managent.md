### 1. Minimal states, minimal render

React renders a component and its children when a state update occurs. For the sake of better performance, it is essential to minimize these state updates.

One way to achieve minimal state updates is by minimizing the number of states used in a component.

Many of the rules listed here are some way to achieve minimal state, thus minimal render. Keep rule #1 in mind if you don't understand why the following rules are designed to optimize performance.

### 2. Independent child, careless parent

Reusability is one of the most important aspects of component design. To maximize reusability in various environments, you should minimize external dependencies like props and contexts. Such rule can be implemented using the "independent child" principle.

- States that can be managed within a child should be managed within a child
  - If not, consider [lifting the state](https://reactjs.org/docs/lifting-state-up.html)
- Minimize number of props
- Expose states as props if and only if the parent must interfere with the children's internal states
  - ex) An input component with a `value` state exposes its state and state changes (`value`, `setValue`) using `value` and `onChange` props
  - It may sound like we are creating two separate states through such design, but this ultimately creates a flow where you could independently write business logic in the parent component while using the same children

> Parents can't always tell what their children should do. Sometimes it's necessary for children to learn what they can do on their own.

If a state update/management can be done at a lower level, meaning that the state which you are trying to update is irrelevant to the higher parent, it should be done at a lower level. Component updates at a higher level tend to be more expensive since it requires more render computations. It could either result in fewer renders or cheaper operation costs.

If you find yourself writing child components with too many `set...` props which are meant to update the parent's state, see if your code fits well with rule #2.

> Sidenote: Please do not ever name your prop `set...`. It directly means that you are modifying the parent's state while also forcing a state variable name. We don't want this to happen because in fact the parent component could be using a different state name / modifying the retrieved value and writing it to a different state. Rather, you should name it like an event callback(`on...`) to indicate that a state "changed" and the parent should do something(or not) about it.


### 3. Derived states should be derived from states

> "Single Source of Truth"

This may sound like a far digression, but let's take a look at how we design databases. Relational databases exist to serve one of the most important purposes of database design: the single source of truth rule. This means that data(and data that derives from another data) should only refer to a single source, so that there is only one way to retrieve the original data. For instance, if you want to return the age of a user, you wouldn't have separate columns for age and birthday, but rather use birthday as a single source of truth to dynamically calculate the age of the user, and this is because *age is derived from birthday*.

The point is: a derived state must not be a separate state.

If you are calling `setState` multiple times within a single call, or within a `useEffect` hook, you should double check your state dependencies.

**BAD** (derived state modification through consecutive state update)
```typescript
const Component: React.FC = () => {
	const [first, setFirst] = useState<number>(0)
	const [second, setSecond] = useState<number>(0)
	const [result, setResult] = useState<number>(0)
	return <>
		<input type="number" value={first} onChange={(e) => {
			setFirst(e.target.value)
			setResult(e.target.value + second)
		}} />
		+
		<input type="number" value={second} onChange={(e) => {
			setSecond(e.target.value)
			setResult(e.target.value + first)
		}} />
		=
		{result}
	</>
}
```

**BAD** (derived state modification through `useEffect`)
```typescript
const Component: React.FC = () => {
	const [first, setFirst] = useState<number>(0)
	const [second, setSecond] = useState<number>(0)
	const [result, setResult] = useState<number>(0)
	
	useEffect(() => setResult(first + second), [first, second])
	
	return <>
		<input type="number" value={first} onChange={(e) => setFirst(e.target.value)} />
		+
		<input type="number" value={second} onChange={(e) => setSecond(e.target.value)} />
		=
		{result}
	</>
}
```

**GOOD**
```typescript
const Component: React.FC = () => {
	const [first, setFirst] = useState<number>(0)
	const [second, setSecond] = useState<number>(0)
	return <>
		<input type="number" value={first} onChange={(e) => setFirst(e.target.value)} />
		+
		<input type="number" value={second} onChange={(e) => setSecond(e.target.value)} />
		=
		{first + second}
	</>
}
```

Some may ask: "Then how am I supposed to implement caching?"

You should `useMemo`.