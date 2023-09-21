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

## General rules for all the Hooks

1. Hooks can ONLY be used either in **React Functional Component** or **other custom Hooks**
2. Hooks can ONLY be used in **the root level**: you can not use Hook in some nested function, for example:

```js
const MyComponent = () => {
  const submitHandler = () => {
    const [name, setName] = useState("Martin"); // ❌ wrapped in a function
  };

  if (true) {
    const [name, setName] = useState("Martin"); // ❌ wrapped in a if statement
  }
};
export default MyComponent;
```

## 1. State Hooks

### 🪝`useState`


`useState(initialState)`declares a state variable in component that you can update directly. `initialState` is the initial value of the state, it can be:

- a variable in any type (could also be an `object`)
- a pure function, which takes 0 arguments, and returns a value in any type


!!! warning "State"
    The State that created by `useState` decides when the Component (re)renders! Every time the State changes, the Component will rerender!

!!! note "State Batching"
    The `setXXX()` functions in the same synchronous execution cycle (scope) will be batched together, and cause ONLY one re-render.
    ```js
    setName("Max");
    setAge("5")
    ```
    The updates in both `name` and `age` will be applied **simultaneously**! That's why the following code will NOT work:
    ```js
    console.log(name); // prints name state, e.g. 'Manu'
    setName('Max');
    console.log(name); // ??? what gets printed? Still 'Manu'?
    ```

Example:

```js
import { useState } from "react";

function MyComponent() {
  // multiple States
  const [name, setName] = useState("Taylor");
  const [age, setAge] = useState(42);

  // ... or put multiple States in 1 object:
  const [person, setPerson] = useState({ name: "Taylor", age: 42 });
}
```

!!! note "Array Destruction"
The way we write the LEFT-HAND side `[name, setName]` is called "Array Destruction"

A classic use case for useState as "Controlled Component":

```jsx
<form>
  <div>
    <label htmlFor="title">Name</label>
    <input
      type="text"
      id="title"
      value={name}
      //   onChange={(event) => setName(event.target.value)}
      onChange={(event) => {
        // ⚠️ This new variable `newName` solve the "wrong object" error caused by how React handles the state
        const newName = event.target.value;
        setName((prevState) => ({
          name: newName,
          age: prevState.age,
        }));
      }}
    ></input>
  </div>
</form>
```

`setXXX()` only affects what useState will return starting from the next render. Therefore this will not work:

```js
function handleClick() {
  setName("Robin");
  console.log(name); // Still "Taylor"!
}
```

!!! warning
    if a variable depends on its previous value, to make sure the value is updated. We pass an **updater function** instead of a **value** to the `setXXX()`:

    ```js
    // by convention, we use `a` or `prevAge` to represent the previous/old/pending state
    setAge(age + 1); // ❌
    setAge((age) => age + 1); // ✅ pass a updater function as parameter!
    ```

### 🪝`useReducer`

`useReducer(reducer, initialArg, init?)` sources out multiple `useState()` to keep the code cleaner. Ususally we use it when there are several actions on one State. The defined `reducer` is outside of the Component. `useReducer()` has 3 input parameters:

1. `reducer`: the `function reducer(state, action)` returns the next state. State and action can be any types.
2. `initialArg`: initial state, can be any type.
3. `[OPTIONAL]init`: The initializer function that should return the initial state. If exists, then `init(initialArg)` will replace the `initialArg`.

Example:

```js
import { useReducer } from "react";

// define a `reducer`
function reducer(state, action) {
  switch (action.type) {
    case 'incremented_age':
      return {
        age: state.age + state.amount,
      };
    case 'decreased_age':
      return {
        age: state.age - state.amount,
      };
  }

  throw Error("Unknown action.");
}

export default function Counter() {
  /*
   * [variable, function] =
   */
  const [state, dispatch] = useReducer(reducer, { age: 42 });

  return (
    <>
      <button
        onClick={() => {
          dispatch({ type: "incremented_age", amount: 2 });  {/* 👉 `dispatch()` with input parameter */}
        }}
      >
        Increase
      </button>
      <p>Hello! You are {state.age}.</p>    {/* 👉 `state` */}
    </>
  );
}
```

## 2. Context Hooks

### 🪝`useContext`

`useContext(SomeContext)` is used to manage global variable in the App, such as theme, authentication. It lets a component receive information from distant parents without passing the props through the tree in three steps:

1. Define Context and ContextProvider
For convention we put the Context in a separate folder `/context`. The `createContext()` function includes initial values.

```js
// ./context/theme-context.jsx
import { createContext } from 'react';

// create Context
export const ThemeContext = React.createContext({
  darkTheme: false;
  setDarkTheme: () => {}
});

// create ContextProvider, and wrap all the given Children
export const ThemeContextProvider = props => {
  const [darkTheme, setDarkTheme] = useState(true);

  const darkThemeHandler = () => {
    setDarkTheme(true)
  }

  return (
    {/*⚠️ value中有两个花括号，里面那个花括号表示Object */}
    <ThemeContext.Provider value={{ darkTheme: darkTheme, setDarkTheme: darkThemeHandler }}>
      {props.children}
    </ThemeContext.Provider>
  )
}


```

<!-- === "./context/ThemeContext.tsx:"

    ```tsx
    import { createContext } from 'react';

        interface ThemeContextProps {
        	darkTheme: boolean;
        	setDarkTheme: React.Dispatch<React.SetStateAction<boolean>>;
        }

        export const ThemeContext = React.createContext<ThemeContextProps>({} as ThemeContextProps);
    ``` -->

2.Create a **Context Provider** as a "wrapper": The `Provider` has the `value` which is the implementation and overwrites the initial values. All the childeren elements that want to use the Context are wrapped in this `Provider`.

```jsx
// index.jsx
import React from 'react';
import ReactDOM from 'react-dom';

import './index.css';
import App from './App';
import { ThemeContextProvider } from "./context/theme-context";


ReactDOM.render(
  <ThemeContextProvider>
    <App />
  </ThemeContextProvider>, 
  document.getElementById('root')
);

```

该例子中，`<ThemeContextProvider>`被用于于最顶层的 `index.js` 中，包裹着`<App/>`。这意味着，所有 App 组件的子组件都能访问到该 Context 的值。


3.Use/Change the Context
Here with the `useContext` you can actually apply the change: in this case, change the theme type.

```js
import { useContext } from 'react';
import { ThemeContext } from './context/ThemeContext';

const ThemeChanger = () => {
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

export default ThemeChanger;
```

!!! warning
    `useContext()` call in a component is not affected by providers returned from the same component. The corresponding `<Context.Provider>` needs to be above the component doing the `useContext()` call.

## 3. Ref Hooks

### 🪝`useRef`

`useRef(initialValue)` is a React Hook that lets you reference a value that’s **not needed for rendering**. --> The ref value is independent from render times!

```js
import { useRef } from "react";

const MyComponent = () => {
  let count_ref = useRef(0);

  const handleClick = () => {
    count_ref.current = count_ref.current + 1;
    alert("You clicked " + count_ref.current + " times!");
  };

  return <button onClick={handleClick}>Click me!</button>;
};
```

Another example:
```jsx
const input_ref = useRef()

return <>
  <input 
  type="text"
  ref={input_ref}
  />
</>
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

### 🪝`useImperativeHandle`

`useImperativeHandle`

## 4. Effect Hooks

### 🪝`useEffect`

`useEffect(setup, dependencies?)`

The second parameter `dependencies` has 3 possibilities:

1. `[a, b]`: a dependency array --> runs after the **component initial render** and after re-renders of variable a AND b
2. `[]`: an empty array --> only run after the **component initial render** --> similar to `componentDidMount()`
3. ` `: empty --> runs after EVERY single render (and re-render when State changes)

#### Clean up
Sometimes we need to **clean up** the content in `useEffect`, we do so by returning a function in the end, this function will be executed before the next round executes:
```js
useEffect(()=>{
  // do sth...
  const timer = setTimeout(()=>{
    //...
  }, 500)

  // clean up
  return () => {
    clearTimeout(timer)
  };
},[name])
```
* if `dependencies` is `[]`, then the **clean up** happens when the Component gets unmounted!

### 🪝`useLayoutEffect`

`useLayoutEffect(setup, dependencies?)` is a version of `useEffect(setup, dependencies?)` that fires before the browser repaints the screen.

### 🪝`useInsertionEffect`

`useInsertionEffect(setup, dependencies?)` is a version of `useEffect(setup, dependencies?)` that fires before any DOM mutations.

## 5. Performance Hooks

### 🪝`useMemo`

`useMemo(calculateValue, dependencies)` is a React Hook that lets you **cache a value/component** between re-renders.

Example - cache a value:
```js
import { useMemo } from "react";

function TodoList({ todos, tab }) {
  // the function filterTodos returns a value - and this value is to be cached
  const visibleTodos = useMemo(() => filterTodos(todos, tab), [todos, tab]);
}
```

Example - cache a component:
```js
// Ingredients.jsx
import { useMemo } from "react";
const Ingredients = () => {

  // cache the component <IngredientList />
  const ingredientList = useMemo(() => {
    return (
      <IngredientList
        ingredients={userIngredients}
        onRemoveItem={removeIngredientHandler}
      />
    );
  }, [userIngredients, removeIngredientHandler]);


  return (
    <section>
      <Search onLoadIngredients={filteredIngredientsHandler} />
      {ingredientList}
    </section>
  )
}
```

!!! note "`useMemo` vs `useCallback`"
    - `useMemo` cache a value/component
    - `useCallback` cache a function

### 🪝`useCallback`

`useCallback(fn, dependencies)` is a React Hook that lets you **cache a function** between re-renders. This could avoid unnecessary re-render. The dependency array indicates that when the function ONLY re-renders when the dependency changes, if the array is empty, then it means this function should NEVER be re-rendered.

```js
  // original function:
  const handleSubmit = (orderDetails) => {
    post('/product/' + productId + '/buy', {
      referrer,
      orderDetails,
    });
  }

  // function in useCallback:
  const handleSubmit = useCallback((orderDetails) => {
    post('/product/' + productId + '/buy', {
      referrer,
      orderDetails,
    });
  }, [productId, referrer]);
```

### 🪝`useTransition`

`useTransition()` is a React Hook that lets you update the state without blocking the UI, it does NOT take any parameters.

### 🪝`useDeferredValue`

## 6. Other Hooks

### 🪝`useDebugValue`

### 🪝`useId`

### 🪝`useSyncExternalStore`

## 7. Customized Hook
Name convention is `useSOMETHING`, and its actually just a function. Whenever this customized hook is called, a new snapshot of this hook will be created. So the logic of this hook can be shared between different component, but not the data of this hook.

TODO: https://ibm-learning.udemy.com/course/react-the-complete-guide-incl-redux/learn/lecture/25599852#overview
