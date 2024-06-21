#terminal #linux 

So you want to be a hipster, I mean be efficient and learn vim keybindings?
As with everything, the beginning will be hard, but all it takes is practice and muscle memory.

### Getting help

You can't learn this by simply reading. Make sure to practice often and use the available manuals and tutorials, they are actually quite good.

Use `vimtutor` as an excellent beginner tutorial.
```
vimtutor
```

Read up on a specific topic.
```
# Seriously, you can get help about everything
:help word
:help WORD
:help d
:help $
```

## Normal mode

### Navigation

| **Description**                                           | **Key**                                                     |
| --------------------------------------------------------- | ----------------------------------------------------------- |
| Move left                                                 | h                                                           |
| Move right                                                | l                                                           |
| Move up                                                   | k                                                           |
| Move down                                                 | j                                                           |
| Move word forward                                         | w                                                           |
| Move word backward                                        | b                                                           |
| Move word end                                             | e                                                           |
| Move WORD forward                                         | W                                                           |
| Move WORD backward                                        | B                                                           |
| Move to end of WORD                                       | E                                                           |
| Move to end of line                                       | $                                                           |
| Move to beginning of line                                 | 0                                                           |
| Move to first non empty character of line                 | ^                                                           |
| Move to specific character on line                        | f + character <br>`f.` jump to the next dot on the line     |
| Move to previous character on line                        | F + character<br>`F.` jumps to the previous dot on the line |
| Move sentence up                                          | (                                                           |
| Move sentence down                                        | )                                                           |
| Move paragraph up                                         | {                                                           |
| Move paragraph down                                       | }                                                           |
| Move half page up                                         | ctrl + u                                                    |
| Move half page down                                       | ctrl + d                                                    |
| Move full page up                                         | ctrl + u                                                    |
| Move full page down                                       | ctrl + b                                                    |
| Move to start of page                                     | gg                                                          |
| Move to end of page                                       | G                                                           |
| Move to next curly braces, square brackets or parenthesis | %                                                           |

> [!info]
> Keys can be prefixed with an number. For example `20 k` moves you up 20 times.


> [!info]
> `WORD` is not the same as `word`.
> `WORD` is always delimited by whitespace.
> * `ansible.builtin.import_role:` is a single `WORD`.
>
>`word` is delimited by non-keyword characters.
>All of these are separate `words`: 
> * `ansible`
> * `.`
> * `builtin`
> * `.`
> * `import_role`
> * `:`

### Manipulating text

| **Description**                       | **Key**                                                                                    |
| ------------------------------------- | ------------------------------------------------------------------------------------------ |
| Copy full line                        | yy                                                                                         |
| Paste full line                       | p                                                                                          |
| Undo                                  | u                                                                                          |
| Undo all changes on a line            | U                                                                                          |
| Redo                                  | ctrl + r                                                                                   |
| Dele full line                        | dd                                                                                         |
| Delete a word                         | d + i  + w                                                                                 |
| Delete a WORD                         | d + i + W                                                                                  |
| Delete paragraph                      | d + i + p                                                                                  |
| Delete sentence                       | d + i + s                                                                                  |
| Delete up to specific character       | d + t + character<br>`dt{` deletes up to next `{`                                          |
| Change word                           | c + i + w                                                                                  |
| Repeat previous operation             | .                                                                                          |
| Search forward for character          | / + character<br>`/atlantis` moves to first iteration of the word atlantis from the cursor |
| Iterate forward                       | n                                                                                          |
| Iterate backward                      | N                                                                                          |
| Search forward for word under cursor  | *                                                                                          |
| Search backword for word under cursor | #                                                                                          |
| Create markpoint                      | m + character<br>`ma` creates markpoint named `a`                                          |
| Move to markpoint                     | \` + char<br>\`a moves to markpoint `a`                                                    |
| Move to previous location             | \` + \`                                                                                    |
| Move to last edited location          | \` + .                                                                                     |
| Join next 3 lines together            | 3 + J                                                                                      |
| Make word uppercase                   | g + U + w                                                                                  |
| Make world lowercase                  | g + u + w                                                                                  |
| Make line lowercase                   | g + u + u                                                                                  |
| Make line uppercase                   | g + U + U                                                                                  |
| Increment a number                    | ctrl + a                                                                                   |
| Decrement a number                    | ctrl + x                                                                                   |
| Replace character on current position | r + character<br>r + x replaces the current character with `x`                             |
|                                       |                                                                                            |

### Folding

| Description                  | Key   |
| ---------------------------- | ----- |
| Fold all code blocks         | z + M |
| Unfold all code blocks       | z + R |
| Fold individual code block   | z + c |
| Unfold individual code block | z + o |
|                              |       |

### Advanced


| Description                    | Key                                                                               |
| ------------------------------ | --------------------------------------------------------------------------------- |
| Start recording                | q + character<br>q + a stored recording as a                                      |
| Stop recording                 | q                                                                                 |
| Apply recording                | @ + character<br>@ + a applies recording a<br>15 + @ + a applies macro a 15 times |
| Enable relative line numbering | :set relativenumber                                                               |


### Switching modes

| **Description**                           | **Key** |
| ----------------------------------------- | ------- |
| Insert mode before cursor                 | i       |
| Insert mode after cursor                  | a       |
| Insert mode at beginning of line          | I       |
| Insert mode at end of line                | A       |
| Insert mode after current line            | o       |
| Insert mode before current line           | O       |
| Change word and switch to insert mode     | c + w   |
| Change full line and sitch to insert mode | c + c   |


## Insert mode

## Visual mode

## Command mode

| Description                                | Key                                                                                              |
| ------------------------------------------ | ------------------------------------------------------------------------------------------------ |
| Sort and keep unique entries               | :sort u                                                                                          |
| Sort                                       | :sort                                                                                            |
| Sort reversed order                        | :sort!                                                                                           |
| Load external file to cursor position      | :read filename<br>:read /tmp/x.txt reads the file and places the contents at cursor position     |
| Load output from STDOUT to cursor position | :read !command<br>`:read !ls -ltr` parses the output of `ls -ltr` to the current cursor position |
| Substitute on line                         | :s/old/new/g                                                                                     |
| Subtitute on file                          | :%s/old/new/g                                                                                    |
| Substitute between line numbers            | :#,#s/old/new/g<br>`:10,20s/hi/hello/g` substitutes `hi` to `hello` between line `10` and `20`   |
| Run commands                               | :!command<br>`:!ls -ltr` runs the `ls -ltr` command<br>                                          |
| Write to specific file                     | :w FILENAME<br>`:w /tmp/test.txt`                                                                |

## Registers

Vim uses a set of registers. You can use these register using the `"` key.

> [!info]
> If you do not have a system (`+`) register, make sure to install `vim-gtk3`.
> ```
> apt install vim-gtk3
> ```


| Description             | Key                                                                                                                                                 |
| ----------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------- |
| List all register       | :reg                                                                                                                                                |
| Use a specific register | " + register + operation<br>`"+y`  yanks the current line to the `+` register, which is the clipboard<br>`"3p` pastes the text stored in register 3 |

## References

[Vim Motions for absolute beginners](https://www.youtube.com/watch?v=lWTzqPfy1gE)
[Intermediate Vim Motions](https://www.youtube.com/watch?v=nBjEzQlJLHE)