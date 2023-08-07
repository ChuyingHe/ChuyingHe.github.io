<!-- Reference: 
	https://ibm-learning.udemy.com/course/react-the-complete-guide-incl-redux
	Section 28 -->
# Hooks in React

**React hook** is a complementary function for React Functional Component. In the functionality perspective, we could understand as:
```bash
Hook + Functional Component = Class Component
```

**React Hooks** are special functions that can only be used in Functional Component. They are have the format of `useXXX()`. In **React@18.20.0**, there are 6 types of built-in hooks, plus your own hook - the customized hook.

<img src='../hook-1.svg' width=600>

!!! note "Data type in hook"
	the data that saved in hook is an **object**! Means it can be everything
## 1. State Hooks
### useState(initialState)
`useState(initialState)`declares a state variable in component that you can update directly. `initialState` is the initial value of the state, it can be:

- a variable in any type
- a pure function, which takes 0 arguments, and returns a value in any type


Example:
```js
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
```js
// by convention, we use `a` or `prevAge` to represent the previous/old/pending state
setAge(age + 1);		// wrong
setAge(age => age + 1);		// correct: pass a updater function as parameter!
```


### useReducer(reducer, initialArg, init?)
`useReducer()` sources out multiple `useState()` to keep the code cleaner. `useReducer()` has 3 input parameters:

1. `reducer`: the `function reducer(state, action)` returns the next state. State and action can be any types.
2. `initialArg`: initial state, can be any type.
3. `[OPTIONAL]init`: The initializer function that should return the initial state. If exists, then `init(initialArg)` will replace the `initialArg`.

Example:
```js
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
	/*
	* [variable, function] = 
	* 
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



# 2. Context Hooks
## useContext(SomeContext)
`useContext()` lets a component receive information from distant parents without passing the props through the tree in 3 steps:

1.Create Context


=== "ThemeContext.jsx"
	```js
	import { createContext } from 'react';

	export const ThemeContext = React.createContext<>();
	```
=== "ThemeContext.tsx"
	```tsx
	import { createContext } from 'react';

	interface ThemeContextProps {
		darkTheme: boolean;
		setDarkTheme: React.Dispatch<React.SetStateAction<boolean>>;
	}

	export const ThemeContext = React.createContext<ThemeContextProps>({} as ThemeContextProps);
	```

2.Create a **Context Provider** as a "wrapper": all the childeren elements that want to use the Context is wrapped in this **Provider**
```js
// App.tsx
import ThemeContext from './ThemeContext';

const App = () => {
	const [darkTheme, setDarkTheme] = useState(true)

	const toggleTheme = () => {
		setDarkTheme(prevDarkTheme => !prevDarkTheme)
	}

	return (
			<>
				{/*⚠️ value中有两个花括号，里面那个花括号表示Object */}
				<ThemeContext.Provider value={{ darkTheme, setDarkTheme }} >
					<button onClick={toggleTheme}>Toggle Theme</button>

					<ComponentClass />
					<ComponentFunctional />
				</ThemeContext.Provider>
			</>
		)
};

export default App;
```

该例子中，`<ThemeContext.Provider>`存在于最顶层的App组件中，这意味着，所有App组件的子组件都能访问到该Context的值。


<!-- TODO: content tabs not working: https://squidfunk.github.io/mkdocs-material/reference/content-tabs/#ordered-list -->

3.Use the Context
```js
import { useContext } from 'react';
import { ThemeContext } from './App';

const ComponentFunctional = () => {
	// Now we have the Context available!
	const themeContext = useContext(ThemeContext);

	const themeStyle = {
		backgroundColor: themeContext.darkTheme? "#333":"#CCC";
		color: themeContext.darkTheme?"white":"black";
	}

	return (
		<div style={themeStyle}>Functional Component Style</div>
		<button onClick={()=>themeContext.setDarkTheme(false)}>Turn off Dark Theme</button>
	)

};

export default ComponentFunctional;
```

<!-- TODO: Example: use the Context in Class Component -->

!!! warning
	`useContext()` call in a component is not affected by providers returned from the same component. The corresponding `<Context.Provider>` needs to be above the component doing the `useContext()` call.

# 3. Ref Hooks
## useRef(initialValue)
`useRef()` is a React Hook that lets you reference a value that’s **not needed for rendering**. --> The ref value is independent from render times!

```js
import { useRef } from 'react';

const MyComponent = () => {
  let count_ref = useRef(0);

  const handleClick = () => {
    count_ref.current = count_ref.current + 1;
    alert('You clicked ' + count_ref.current + ' times!');
  }

  return (
    <button onClick={handleClick}>
      Click me!
    </button>
  );
}
```

!!! note
		useRef returns an object with a single property `current`, to use the ref, you always need to write `myRef.current`


To change the value in ref, you can assign new value:
```js
count_ref.current = 5;
```

⚠️ Do NOT write/read your Ref during the **component rendering**:
```js
function MyComponent() {
  // ...
  // 🚩 Don't write a ref during rendering
  myRef.current = 123;
  // ...
  // 🚩 Don't read a ref during rendering
  return <h1>{myOtherRef.current}</h1>;
}
```
Instead, ONLY read or write refs from **event handlers or effects** instead.
```js
```

## useImperativeHandle
`useImperativeHandle`

# Effect Hooks 
## useEffect()
`useEffect(setup, dependencies?)`

The second parameter `dependencies` has 3 possibilities:
1. `[a, b]`: a dependency array --> runs after the **component initial render** and after re-renders of variable a AND b
2. `[]`: an empty array --> only run after the **component initial render**
3. ` `: empty --> runs after EVERY single render (and re-render)


## useLayoutEffect()
`useLayoutEffect(setup, dependencies?)` is a version of `useEffect(setup, dependencies?)` that fires before the browser repaints the screen.


## useInsertionEffect()
`useInsertionEffect(setup, dependencies?)` is a version of `useEffect(setup, dependencies?)` that fires before any DOM mutations.



# Performance Hooks
## useMemo()
`useMemo(calculateValue, dependencies)` is a React Hook that lets you cache the **result of a calculation** between re-renders.

```js
import { useMemo } from 'react';

function TodoList({ todos, tab }) {
  const visibleTodos = useMemo(
    () => filterTodos(todos, tab),
    [todos, tab]
  );
}
```

## useCallback()
`useCallback(fn, dependencies)` is a React Hook that lets you cache a **function definition** between re-renders.

```js
import { useCallback } from 'react';

export default function ProductPage({ productId, referrer, theme }) {
  const handleSubmit = useCallback((orderDetails) => {
    post('/product/' + productId + '/buy', {
      referrer,
      orderDetails,
    });
  }, [productId, referrer]);
```


## useTransition()
`useTransition()` is a React Hook that lets you update the state without blocking the UI, it does NOT take any parameters.

## useDeferredValue()

# Other Hooks
## useDebugValue()
## useId()
## useSyncExternalStore()


# Customized Hook