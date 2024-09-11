#linux

## Tmux

Tmux is a tool such as `screen`. It allows you to set up a `tty` and de-attach, re-attach to it.

```bash
# Create a tty
$ tmux

# CTRL+b + d to detach

# List sessions
$ tmux list-sessions
0: 1 windows (created Mon Mar  7 09:22:28 2022)

# Re-attach to session
$ tmux attach [-t session_id]
```

## Terminator

- Fix double characters in Ubuntu 22 on broadcast This seems to be related to `ibus`.

```bash
# https://github.com/gnome-terminator/terminator/issues/596
ibus restart
```

## Kitty

[Kitty](https://sw.kovidgoyal.net/kitty/overview/ "https://sw.kovidgoyal.net/kitty/overview/") is a very cool terminal, a replacement for terminator that I used before.  
A cool font to use is [Victor Mono](https://fonts.google.com/specimen/Victor+Mono "https://fonts.google.com/specimen/Victor+Mono")

Kitty doesn't work properly with SSH due to some terminal stuff. You can use `kitten` to fix that.

```bash
kitten ssh my-server
```

You can also just create an alias for this.

```bash
# In your ~/.zshrc file
alias s="kitten ssh"
```

[Reference](https://sw.kovidgoyal.net/kitty/faq/#i-get-errors-about-the-terminal-being-unknown-or-opening-the-terminal-failing-or-functional-keys-like-arrow-keys-don-t-work)

### Custom configuration

Edit `~/.config/kitty/kitty.conf`

```text
# BEGIN_KITTY_THEME
# Adwaita dark
include current-theme.conf
# END_KITTY_THEME

# Custom
enable_audio_bell no

map ctrl+left neighboring_window left
map ctrl+right neighboring_window right
map shift+left move_window right
map ctrl+down neighboring_window down
map shift+down move_window up

# Font
font_family VictorMono NFM Medium
font_size 13
```

### Keybindings

| Description                         | Keybind                |
| ----------------------------------- | ---------------------- |
| Create new tab in same screen       | Ctrl + Shift + Enter   |
| Create new tab in new screen        | Ctrl + Shift + T       |
| Go to next layout                   | Ctrl + Shift + L       |
| Swap current tab with tab of choice | Ctrl + Shift + F8      |
| Resize current tab                  | Ctrl + Shift + R       |
| Navigate to screen left/right       | Ctrl + Shift + <- / -> |
| Navigate to tab left/right/up/down  | Ctrl + <- / -> / ↑ / ↓  |

## Warp

Warp is a modern, Rust-based terminal with AI built in so you and your team can build great software, faster.

[Warp: Your terminal, reimagined](https://www.warp.dev)

## Zelij

[Zelij](https://zellij.dev/)is a terminal workspace with batteries included.

