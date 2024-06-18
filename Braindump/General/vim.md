#terminal #linux 

## Normal mode

### Navigation

| **Description**                           | **Key**                                                     |
| ----------------------------------------- | ----------------------------------------------------------- |
| Move left                                 | h                                                           |
| Move right                                | l                                                           |
| Move up                                   | k                                                           |
| Move down                                 | j                                                           |
| Move word forward                         | w                                                           |
| Move word backward                        | b                                                           |
| Move word end                             | e                                                           |
| Move WORD forward                         | W                                                           |
| Move WORD backward                        | B                                                           |
| Move to end of WORD                       | E                                                           |
| Move to end of line                       | $                                                           |
| Move to beginning of line                 | 0                                                           |
| Move to first non empty character of line | ^                                                           |
| Move to specific character on line        | f + character <br>`f.` jump to the next dot on the line     |
| Move to previous character on line        | F + character<br>`F.` jumps to the previous dot on the line |
| Move sentence up                          | (                                                           |
| Move sentence down                        | )                                                           |
| Move paragraph up                         | {                                                           |
| Move paragraph down                       | }                                                           |
| Move half page up                         | ctrl + u                                                    |
| Move half page down                       | ctrl + d                                                    |
| Move full page up                         | ctrl + u                                                    |
| Move full page down                       | ctrl + b                                                    |
| Move to start of page                     | gg                                                          |
| Move to end of page                       | G                                                           |

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

### Switching modes

| **Description**                       | **Key** |
| ------------------------------------- | ------- |
| Insert mode before cursor             | i       |
| Insert mode after cursor              | a       |
| Insert mode at beginning of line      | I       |
| Insert mode at end of line            | A       |
| Insert mode after current line        | o       |
| Insert mode before current line       | O       |
| Change word and switch to insert mode | c       |


## Insert mode

## Visual mode

## Command mode


## References

[Vim Motions for absolute beginners](https://www.youtube.com/watch?v=lWTzqPfy1gE)
[Intermediate Vim Motions](https://www.youtube.com/watch?v=nBjEzQlJLHE)