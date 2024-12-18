---
tags:
  - IDE
  - code
---

## Debugging

Debugging in vscode is done by creating a `.vscode/launch.json` file in your repository.

```json
{
  // Use IntelliSense to learn about possible attributes.
  // Hover to view descriptions of existing attributes.
  // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
  "version": "0.2.0",
  "configurations": [
    {
      "name": "clusters proxy",
      "type": "go",
      "request": "launch",
      "mode": "debug",
      "program": "cmd/tm",
      "args": ["cluster", "proxy"], //  parameters to pass to the program
      "console": "integratedTerminal" //  uses the terminal instead of the debug console, this allows the use of arrow keys
    }
  ]
}
```

## Handy Keybinds

| Description              | Keybind                                                        |
| ------------------------ | -------------------------------------------------------------- |
| Go to last edit location | Control + K, Control + Q                                       |
| Go to matching bracket   | Control + Shift + \                                            |
| Go to definition         | F12                                                            |
| Go to type definition    | Not bound by default, check `editor.action.goToTypeDefinition` |
| Go to implementation     | Ctrl + F12                                                     |
| SmartSelect grow         | Shift + Alt + → (MacOS: **^** + **⌘** + **⌥** + **→**)         |
| SmartSelect shrink       | Shift + Alt + ← (MacOS: **^** + **⌘** + **⌥** + **←**)         |
