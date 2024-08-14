#linux #utilities  

- [autojump](https://github.com/wting/autojump "https://github.com/wting/autojump"): easily jump to frequently used directories
- [powerlevel10k](https://github.com/romkatv/powerlevel10k "https://github.com/romkatv/powerlevel10k"): theme for [[zsh]]
- [starship](https://starship.rs/ "https://starship.rs/"): powerlevel10k seems to no longer be maintained, this seems to be an alternative
- [zsh-autosuggestions](https://github.com/zsh-users/zsh-autosuggestions "https://github.com/zsh-users/zsh-autosuggestions"): suggestions based on history for [[zsh]]
- [zsh-autocomplete](https://github.com/marlonrichert/zsh-autocomplete "https://github.com/marlonrichert/zsh-autocomplete"): autocomplete for [[zsh]]
- [zsh-syntax-highlighting](https://github.com/zsh-users/zsh-syntax-highlighting "https://github.com/zsh-users/zsh-syntax-highlighting"): syntax highlighting for [[zsh]]
- [awesome-zsh-plugins](https://github.com/unixorn/awesome-zsh-plugins "https://github.com/unixorn/awesome-zsh-plugins"): list of awesome plugins for [[zsh]]
- [fzf-tab-completion](https://github.com/lincheney/fzf-tab-completion): fuzzy find with tab completion for [[zsh]]
- [fzf-tab](https://github.com/Aloxaf/fzf-tab): a better version of fzf-tab-completion imho for [[zsh]]
- [kazam](https://github.com/henrywoo/kazam "https://github.com/henrywoo/kazam"): screen recording
- [fzf-history-search](https://github.com/joshskidmore/zsh-fzf-history-search): search better in history for [[zsh]]

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
