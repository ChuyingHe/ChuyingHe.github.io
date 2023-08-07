# VSCode

## .vscode/launch.json
Example for python fastapi runtime:
```json
{
  // Use IntelliSense to learn about possible attributes.
  // Hover to view descriptions of existing attributes.
  // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Python: FastAPI",
      "type": "python",
      "request": "launch",
      "module": "uvicorn",
      "args": ["app.main:app", "--port", "4000", "--reload"],
      "jinja": true,
      "justMyCode": true,
      "env": {
        "secret_url": "https://pretty-little-liars.com",
        "secret_api_key": "oneliarsonedies",
      }
    }
  ]
}
```

## Great plugins


## Preferences: Open User Settings (JSON)
```json
	"editor.formatOnSave": true,
	// Sort imports and automatically remove unused imports when pressing CTRL + S
	"editor.codeActionsOnSave": {
		"source.organizeImports": true
	},
```