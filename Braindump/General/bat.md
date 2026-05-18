
`bat` is a `cat` replacement with built-in syntax highlighting. It makes reading files in the terminal a lot more pleasant, especially for config files, YAML, JSON, and code.

## Install

* Install `bat`

```bash
$ dnf install -y bat
```

> On some distros the binary is called `batcat`. Add an alias if needed: `alias bat=batcat`

## Usage

* Display a file with syntax highlighting

```bash
$ bat file.yaml
```

* Pipe output into it (specify language explicitly when piping)

```bash
$ kubectl get cm my-config -o yaml | bat -l yaml
```

## Configure

By default `bat` shows line numbers, a file header and borders. Use `--style=plain` to get clean output without decorations.

* Add an alias to `~/.bashrc` or `~/.zshrc`

```bash
alias bat='bat --style=plain'
```

* Reload your shell config

```bash
$ source ~/.bashrc
```

To make `less` automatically use `bat` for syntax highlighting, add this to your shell config as well:

```bash
export LESSOPEN="|bat --color=always --style=plain %s"
export LESS="-R"
```

After this, opening any file with `less file.yaml` will automatically show it with colors.