---
tags:
  - development
  - containers
---

[Devcontainers](<https://containers.dev/overview>](<https://code.visualstudio.com/docs/devcontainers/containers>), or development containers allow you to work together on the same project without having to keep track of all development dependencies needed to run your code.

The only requirement that you need is to have `docker` installed locally. You define a `.devcontainer/devcontainer.json` file where you describe whatever dependencies (and base image) that your application should use. Using the [Dev Container VSCode extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers) you can run your code directly from VSCode in that development container.

The in the `devcontainer.json` file you can specify the ports that need to be forwarded to your local device, which base images should be used, which extensions need to be installed and much more. The default microsoft devcontainer base images already come with a set of tools installed (including CoPilot).

Below is an example of such a `devcontainer.json` configuration file.

```json
// For format details, see https://aka.ms/devcontainer.json. For config options, see the
// README at: https://github.com/devcontainers/templates/tree/main/src/go
{
"name": "Go",
// Or use a Dockerfile or Docker Compose file. More info: https://containers.dev/guide/dockerfile
"image": "mcr.microsoft.com/devcontainers/go:1-1.23-bookworm",

// Features to add to the dev container. More info: https://containers.dev/features.

// "features": {},

// Use 'forwardPorts' to make a list of ports inside the container available locally.
"forwardPorts": [7070]

// Use 'postCreateCommand' to run commands after the container is created.
// "postCreateCommand": "go version",


// Configure tool-specific properties.
// "customizations": {},

// Uncomment to connect as root instead. More info: https://aka.ms/dev-containers-non-root.

// "remoteUser": "root"
}
```

The main benefit is that you do not have to think about local development dependencies as these are all stored in the container. The fact that you can work with VSCode like it is in your local environment is also nice. You can set breakpoints, run debugging, ...
