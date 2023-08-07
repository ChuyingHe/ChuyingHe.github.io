# Create React App (CRA)
In CRA, the src directory is the root of the project, and by default, you cannot import files from outside the src directory due to security reasons. Therefore this will not work:

```
import DE from '../public/languages/german.json';
```

!!! error
	The `public` directory is served as-is and not processed by webpack!

The solution will be:
```
// Move the file to src/languages/german.json
import GM from './languages/german.json';
```
