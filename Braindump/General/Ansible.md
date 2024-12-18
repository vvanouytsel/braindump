---
tags:
  - iac
  - ansible
---

## Working with Strings

You can use string concatenation with multiple variables

```bash
{{ my_variable ~ '/' ~ item }}
```

When dealing with multi-line strings, the indent filter can help.

This indents every line of your variable with 4 spaces/tabs. If `first` is set to `True`, the first line is also indented.

```yaml
private_key: |
{{ lookup('community.hashi_vault.hashi_vault', 'secret=secret/example:private_key') | indent(width=4, first=True) }}
```

## Omitting Module Parameters

You can omit a module parameter if you do not want it to be set.

For example, notice that the `hostname` module allows you to set the `use` variable as optional. If not set the module tries to autodetect which method to use.

```yaml
- name: Set a hostname
  ansible.builtin.hostname:
    name: web01

- name: Set a hostname specifying strategy
  ansible.builtin.hostname:
    name: web01
    use: systemd #alpine, debian, macosx, solaris, ...
```

You can make this more generic by setting the method via a variable. If the variable is not defined, the whole `use` parameter will be omitted.

```yaml
- name: Set a hostname specifying strategy
  ansible.builtin.hostname:
    name: web01
    use: "{{ method }} | default(omit)"
```

## Troubleshooting

You can use the `--step` flag to make troubleshooting easier. This makes Ansible wait for your confirmation before executing a step. That allows you to troubleshoot the state of the system between tasks.

```bash
$ ansible-playbook plays/myplay.yml --step

Perform task: TASK: example/role1 : Execute task1
(N)o/(y)es/(c)ontinue: y

Perform task: TASK: example/role1 : Execute task1 
(N)o/(y)es/(c)ontinue: **********************************************************************************************************************************************************************************
```
