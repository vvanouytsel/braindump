---
tags:
  - linux
  - tools
---

Often times you have a project where you quickly want to do something. This can be running tests, building a container image, generating documentation or something else.

One option is to use scripting to execute these tasks, another option is to use a 'command runner'.

# Make

[Make](https://www.make.com/en) uses a `Makefile` file. It is traditionally used as a build tool, but one of the features is to run commands.

# Just

[Just](https://github.com/casey/just) uses a `justfile`where you define 'recipes'. These recipes are the commands or sets of commands that you want to run together. The syntax is inspired by `make`, [but much of its complexity is removed](https://github.com/casey/just#what-are-the-idiosyncrasies-of-make-that-just-avoids).

## Usage

> [!info]
> There is a [handy VSCode extension](https://marketplace.visualstudio.com/items?itemName=nefrob.vscode-just-syntax)that can be used for syntax-highlighting.

Download the precompiled binary via the [releases](https://github.com/casey/just/releases) page or install it via your favorite package manager.

```bash
$ dnf install just
```

Create a `justfile` in the root of your project.

```bash
# justfile
run-container:
  podman run -ti --rm ubuntu:latest /bin/bash
```

Run the `justfile` using the `just` command and by passing the `recipe` as argument.

```bash
$ just run-container
podman run -ti --rm ubuntu:latest /bin/bash
root@f15adf97df0e:/# 
```

# Task

[Task](https://taskfile.dev/) uses a `Taskfile.yml` file in the root of your project.
