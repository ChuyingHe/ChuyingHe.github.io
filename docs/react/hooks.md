[TOC]

<!-- Reference: 
	https://ibm-learning.udemy.com/course/react-the-complete-guide-incl-redux
	Section 28 -->

**React hook** is a complementary function for React Functional Component. In the functionality perspective, we could understand as:
```bash
Hook + Functional Component = Class Component
```

# 1. React Hooks
**React Hooks** are special functions that can only be used in Functional Component. They are have the format of `useXXX()`. In React@18.20.0, there are 6 types of built-in hooks, plus your own hook - the customized hook.

<img src='../hook-1.svg' width=600>

!!! note "Data type in hook"
	the data that saved in hook is an **object**! Means it can be everything
## 1. State Hooks
### useState(initialState)
`useState(initialState)`declares a state variable in component that you can update directly. `initialState` is the initial value of the state, it can be:

- a variable in any type
- a pure function, which takes 0 arguments, and returns a value in any type


Example:
```jsx
import { useState } from 'react';

function MyComponent() {
	const [age, setAge] = useState(42);
	const [name, setName] = useState('Taylor');
// ...
}
```

`setXXX()` only affects what useState will return starting from the next render. Therefore this will not work:
	
``` js
function handleClick() {
  setName('Robin');
  console.log(name); // Still "Taylor"!
}
```

!!! warning
	if a variable depends on its previous value, to make sure the value is updated. We pass an **updater function** instead of a **value** to the `setXXX()`:
```jsx
// by convention, we use `a` or `prevAge` to represent the previous/old/pending state
setAge(age + 1); // wrong
setAge(age => age + 1);	// correct: pass a updater function as parameter!
```


### useReducer(reducer, initialArg, init?)
`useReducer()` sources out multiple `useState()` to keep the code cleaner. `useReducer()` has 3 input parameters:

1. `reducer`: the `function reducer(state, action)` returns the next state. State and action can be any types.
2. `initialArg`: initial state, can be any type.
3. `[OPTIONAL]init`: The initializer function that should return the initial state. If exists, then `init(initialArg)` will replace the `initialArg`.

Example:
```jsx
import { useReducer } from 'react';

// define a `reducer`
function reducer(state, action) {
	if (action.type === 'incremented_age') {
		return {
			age: state.age + 1
		};
	}
	throw Error('Unknown action.');
}

export default function Counter() {
	/* define:
	* 1. variable `state`
	* 2. function `dispatch()`
	*/
	const [state, dispatch] = useReducer(reducer, { age: 42 });

	return (
		<>
			{/* 👉 use function `dispatch()` with input parameter */}
			<button onClick={() => { dispatch({ type: 'incremented_age' }) }}>
				Increment age
			</button>
			{/* 👉 use variable `state` */}
			<p>Hello! You are {state.age}.</p>
		</>
	);
}
```



## 2. Context Hooks
### useContext()
`useContext()` lets a component receive information from distant parents without passing the props through the tree.


!!! warning
	`useContext()` call in a component is not affected by providers returned from the same component. The corresponding `<Context.Provider>` needs to be above the component doing the `useContext()` call.

## 3. Ref Hooks
### useRef()
`useRef()`
### useImperativeHandle
`useImperativeHandle`

# Customized Hook