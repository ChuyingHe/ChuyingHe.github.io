## Materials
- [Udemy Course](https://ibm-learning.udemy.com/course/react-the-complete-guide-incl-redux/learn/lecture/35734628#questions)
To continue, I'm here: https://ibm-learning.udemy.com/course/react-the-complete-guide-incl-redux/learn/lecture/15700342#overview

- [React official tutorial](https://react.dev/learn)
- [git repo](https://github.com/academind/react-complete-guide-code)



<!-- ## Agenda
### Foundation
[] Components
[] Props
[] State & Events
[] Outputting Content
[] Styling
[] Hooks
[] Debugging

### Advanced
[] Refs & Portals
[] Side effects
[] Advanced hooks
[] Custom hooks
[] Behind the Scenes
[] Context API
[] Redux
[] Routing
[] HTTP Requests
[] Authentication
[] Unit Testing
[] NextJS -->

## Running Project for practise
### Running local React project - MS's way
1. install nodeJS
2. download `*.zip` file
3. open the zip file in VSCode
4. run `npm install` and `npm start`

### Creating local new project
using `create-react-app`(or `vite`) toolkit

### Creating online new project
by type in `https://react.new/` in the browser in sandbox platform


## React vs VanillaJS
- **React** is declarative, you define the target UI, not the steps to get there. 
- **Pure Vanilla JavaScript** is imperative, it defines the steps, not the target.

## Component
A component is a piece of the UI (user interface) that has its own **logic** and **appearance**.

Some rules about Component:

- name of a React component is **capitalized**
- A `*.jsx` file can contains multiple components!
- `export default` specify the main component in current file. 
- JSX is stricter than HTML. You have to close tags like <br />.
- one component can only return one JSX tag. Wrap the returning content in `<></>`
- use `className` instead of `class`

## Functional Component
```js
const myComponent = (props) => {
	// props.name
	// props.age
	// props.setStatus(true)
}
```

or using **object destruction** for the props:

```js
const myComponent = ({name, age, setStatus}) => {
	// name
	// age
	// setStatus(true)
}
```

## Displaying data
Displaying data in jsx using curly brackets: 
```js
<h1>{user.name}</h1>
``` 

or in attribute 
```js
<img src={user.imageUrl} />
```

### String concatenation
You could also use **string concatenation**:
```js
<img
    className="avatar"
    alt={'Photo of ' + user.name}
  />
```

### Using object
Using **object** in JSX: we use a regular `{}`object inside the `style={ }` JSX curly braces. 
```js
<img
    className="avatar"
    style={{
      width: user.imageSize,
      height: user.imageSize
    }}
  />
```

### Rendering list
```js
const products = [
  { title: 'Cabbage', id: 1 },
  { title: 'Garlic', id: 2 },
  { title: 'Apple', id: 3 },
];

const listItems = products.map(product =>
  <li key={product.id}>
    {product.title}
  </li>
);

return (
  <ul>{listItems}</ul>
);
```

## Conditional rendering
"outsourced" mode:
```js
let content;
if (isLoggedIn) {
  content = <AdminPanel />;
} else {
  content = <LoginForm />;
}
return (
  <div>
    {content}
  </div>
);
```

"compact" mode:
```js
<div>
  {isLoggedIn ? (
    <AdminPanel />
  ) : (
    <LoginForm />
  )}
</div>
```

"compact" mode without `else` branch:
```js
<div>
  {isLoggedIn && <AdminPanel />}
</div>
```

## Event
```js
function MyButton() {
  function handleClick() {
    alert('You clicked me!');
  }

  return (
    <button onClick={handleClick}>
      Click me
    </button>
  );
}
```
!!! warning "warning"
	:warning: `onClick={handleClick}` has no parentheses at the end! If there is a `()`, the function will be called IMMEDIATELY!, without `()` we ONLY pass it through!


## Passing State across Components
Example:
```js
const [currentCars, setCurrentCars] = useState([]);

const addNewCar = car => {
	setCurrentCars(prev => [..prev, car])
}

// Passing variable & function_referece to another Component
<carList cars={currentCars} onAddNewCar={addNewCar}>

```

## `prop` vs `state`
- `prop` has the following use cases:
	- when the data won't change
	- when its passed from a parent
	- can be computed based on existing state/prop
- `state` is reserved only for _interactivity_, that is, data that changes over time


<!-- for refreshing and upgrading your present react knowledge -->
## Hooks
[Hooks in react](./hooks.md)