# Routes
```js
import { BrowserRouter, Route, Routes } from 'react-router-dom';

...

return (
	 <BrowserRouter>
	 	<Routes>
	 		<Route path="/cases_list" element={<TablePage />} />
	 		<Route path="/cases/:case_id" element={<DetailPage />} />
	 	</Routes>
	 </BrowserRouter>
	)
```

For the detailPage we use `useParameter` to get the id of the page:
```js
import { useParams } from 'react-router-dom';

const DetailPage = () => {
  const { case_id } = useParams();

  return (
    <>
       <h1>Detail Page: {case_id}</h1>
    </>
  );
};

export default DetailPage;

```


# String with format
In the React Componnent attribute where usually its string, you can do this:

```js

<MyComponent
	labelDescription={
		<>
			Zeile 1<br />
			Zeile 2
		</>
	}
/>
```
