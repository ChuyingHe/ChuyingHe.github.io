# Things to repeat & remember

## Remove an Element from an Array by ID
```js
function removeObjectWithId(arr, id) {
  return arr.filter((obj) => obj.id !== id);
}
const arr = [
  { id: 1, name: 'John' },
  { id: 2, name: 'Kate' },
  { id: 3, name: 'Peter' },
];
const newArr = removeObjectWithId(arr, 2);
console.log(newArr);
```
Result:
```bash
[ { id: 1, name: 'John' }, { id: 3, name: 'Peter' } ]
```

The original array is not modified
```js
console.log(arr);
```

```bash
[
  { id: 1, name: 'John' },
  { id: 2, name: 'Kate' },
  { id: 3, name: 'Peter' }
]
```

## HTTP Request
### built-in fetch API 
This `fetch()` function is a browser function, inrelevant to React.
```js
fetch('https://postman-echo.com', {
  method: 'POST',
  body: JSON.stringify(["apple", "banana"]),
  headers: {
    "Content-Type": "application/json"
  }
}).then(response => {   // Do stuffs after receiving the result from the request - its async
  // Parse the RESPONSE, get the real DATA from the RESPONSE
  return response.json()
}).then(data => {
  // data
  console.log(data)
}).catch(error => {
  // handle the error
  console.log(error.message)
})
```