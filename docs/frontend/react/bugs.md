# options.allowedHosts...

🐞

```sh
Invalid options object. Dev Server has been initialized using an options object that does not match the API schema.
- options.allowedHosts[0] should be a non-empty string.
```

🍃
I stumbled on this error when updating to node 18.0.0, it works normally with node 17.0.0. Solution: in `.env` file, add:

```sh
DANGEROUSLY_DISABLE_HOST_CHECK=true
```
