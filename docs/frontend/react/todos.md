# Default export vs named export
in `components.jsx` there are 2 components:

```js
// Named export
export const Car = () => {

}

const Bike = () => {

}

// Default export
export default Bike;
```

usage:
```js
// Named export
import { Car } from './components'

// Default export
import Bike from "./components"

```