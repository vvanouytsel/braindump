You can use string concatenation with multiple variables

```
{{ my_variable ~ '/' ~ item }}
```

When dealing with multi-line strings, the indent filter can help.
This indents every line of your variable with 4 spaces/tabs. If `first` is set to `True`, the first line is also indented.

```yaml
private_key: |
{{ lookup('community.hashi_vault.hashi_vault', 'secret=secret/example:private_key') | indent(width=4, first=True) }}
```