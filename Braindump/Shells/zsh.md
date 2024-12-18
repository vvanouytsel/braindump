---
tags:
  - linux
  - zsh
  - shell
  - macos
---

You can use [Oh My Zsh](https://ohmyz.sh/) as a framework for you zsh.
There are a set of handy [[Utilities]] that you can use.

## Installation

<https://ohmyz.sh/>

## Configuration

Configuration of your zsh is done via the `~/.zshrc` file. This is loaded whenever you open up a new shell. Whenever you make any changes in this file, you either need to source the file or reopen your shell.

```bash
source ~/.zshrc
```

By default, some keys that you might be used to, simply don't work. This can be fixed by adding the following bindkeys to your `~/.zshrc` file.

```bash
# Make HOME, END, DELETE, CTRl+LEFT and CTRL+RIGHT work
bindkey "^[[H" beginning-of-line
bindkey "^[[F" end-of-line
bindkey "^[[3~" delete-char
bindkey "^[[1;3C" forward-word
bindkey "^[[1;3D" backward-word
```

You also might want to have your history survive sessions. To do so, add the following to your `~/.zshrc` file.

```bash
HISTFILE=~/.histfile
HISTSIZE=1000
SAVEHIST=1000
alias history="history 0" # to show you to full history whenever you type 'history'
```