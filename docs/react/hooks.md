[TOC]

<!-- Reference: 
	https://ibm-learning.udemy.com/course/react-the-complete-guide-incl-redux
	Section 28 -->

**React hook** is a complementary function for React Functional Component. In the functionality perspective, we could understand as:
```bash
Hook + Functional Component = Class Component
```

# What is React Hook
**React Hooks** are special functions that can only be used in Functional Component. They are have the format of `useXXX()`. In React@18.20.0, there are 6 types of built-in hooks, plus your own hook - the customized hook.

<img src='../hook-1.svg' width=600>

!!! note "Data type in hook"
	the data that saved in hook is an **object**! Means it can be everything
## State Hooks
### useState(initialState)
`useState()`declares a state variable that you can update directly.

```jsx
import { useState } from 'react';

function MyComponent() {
	const [age, setAge] = useState(42);
	const [name, setName] = useState('Taylor');
// ...
}
```

!!! warning
	`setXXX()` only affects what useState will return starting from the next render. Therefore this will not work:
	
``` js
function handleClick() {
  setName('Robin');
  console.log(name); // Still "Taylor"!
}
```

!!! warning
	if a variable depends on its previous value, to make sure the value is updated. We pass an **updater function** instead of a **value** to the `setXXX()`:

	\* by convention, we use `a` or `prevAge` to represent the previous/old/pending state

```jsx
setAge(age + 1); // wrong

setAge(age => age + 1);	// correct: pass a updater function as parameter!
```

**Use cases:**

- declare state variable in component

### useReducer(reducer, initialArg, init?)
`useReducer()` sources out multiple `useState()` to keep the code cleaner. Example:
```jsx
import { useReducer } from 'react';

function reducer(state, action) {
  if (action.type === 'incremented_age') {
    return {
      age: state.age + 1
    };
  }
  throw Error('Unknown action.');
}

export default function Counter() {
  const [state, dispatch] = useReducer(reducer, { age: 42 });

  return (
    <>
      <button onClick={() => {
        dispatch({ type: 'incremented_age' })
      }}>
        Increment age
      </button>
      <p>Hello! You are {state.age}.</p>
    </>
  );
}
```

**Use cases:**

- declare many state variable in component


## Context Hooks
### useContext()
`useContext()` lets a component receive information from distant parents without passing it as props. 

## Ref Hooks
### useRef()
`useRef()`
### useImperativeHandle
`useImperativeHandle`

# Customized Hook