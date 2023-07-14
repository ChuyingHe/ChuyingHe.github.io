# see log

```sh
git log origin/main..HEAD
```

# reset to previous version

```sh
git reset —soft HEAD~
git restore --staged // Removes the file from the Staging Area, but leaves its actual modifications untouched
```

# revert what has been `git add xxx`

If this has been done:

```sh
git add .gitignore
```

This will be taken back from "added file":

```sh
git reset .gitignore
```
