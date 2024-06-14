#linux #utilities  

- [autojump](https://github.com/wting/autojump "https://github.com/wting/autojump"): easily jump to frequently used directories
- [powerlevel10k](https://github.com/romkatv/powerlevel10k "https://github.com/romkatv/powerlevel10k"): theme for zsh
- [starship](https://starship.rs/ "https://starship.rs/"): powerlevel10k seems to no longer be maintained, this seems to be an alternative
- [zsh-autosuggestions](https://github.com/zsh-users/zsh-autosuggestions "https://github.com/zsh-users/zsh-autosuggestions"): suggestions based on history
- [zsh-autocomplete](https://github.com/marlonrichert/zsh-autocomplete "https://github.com/marlonrichert/zsh-autocomplete"): autocomplete
- [zsh-syntax-highlighting](https://github.com/zsh-users/zsh-syntax-highlighting "https://github.com/zsh-users/zsh-syntax-highlighting"): syntax highlighting
- [awesome-zsh-plugins](https://github.com/unixorn/awesome-zsh-plugins "https://github.com/unixorn/awesome-zsh-plugins"): list of awesome plugins
- [kazam](https://github.com/henrywoo/kazam "https://github.com/henrywoo/kazam"): screen recording

## Pbcopy and Pbpaste
#alias

We can copy and paste using `xsel --input --clipboard` and `xsel --output --clipboard`, but it can be a lot quicker to just write an alias around it.

Install the `xsel` package
```bash
apt-get install -y xsel
```

Add an alias
```bash
vim ~/.zshrc

# add the following
alias pbcopy="xsel --input --clipboard"                                          
alias pbpaste="xsel --output --clipboard"
```

Reload the `zshrc` shell.
```bash
source ~/.zshrc
```

Verify that it works
```bash
❯ echo "hello world" | pbcopy
❯ pbpaste
hello world
```
