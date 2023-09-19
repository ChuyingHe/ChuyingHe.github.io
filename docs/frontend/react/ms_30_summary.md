<!-- for refreshing and upgrading your present react knowledge -->
## Hooks
[Hooks in react](./hooks.md)


## Passing State across Components
Example:
```jsx
const [currentCars, setCurrentCars] = useState([]);

const addNewCar = car => {
	setCurrentCars(prev => [..prev, car])
}

// Passing function reference to another Component
<button onAddNewCar={addNewCar}>

// Passing variable to another Component
<carList cars={currentCars}>

```

## Functional Component
```jsx
const myComponent = (props) => {
	// props.name
	// props.age
	// props.setStatus(true)
}
```

or using **object destruction** for the props:

```jsx
const myComponent = ({name, age, setStatus}) => {
	// name
	// age
	// setStatus(true)
}
```

To continue, I'm here: https://ibm-learning.udemy.com/course/react-the-complete-guide-incl-redux/learn/lecture/15700342#overview