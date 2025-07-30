Delete all local branches that are already removed upstream:

```bash
git fetch -p && for branch in `git branch -vv | grep ': gone]' | awk '{print $1}'`; do git branch -D $branch; done
```

Update list of branches with remote

```sh
git remote update origin --prune
git branch -a
```

Find the last commit that did something with a specific file, this also includes removing the file

```sh
git log --all -1 -- plays/infra-dns.yml
```

Use a specific SSH key when cloning.

```bash
GIT_SSH_COMMAND="ssh -i ~/.ssh/id_ed25519_company_github.pub" git clone git@github.com:Company/example.git
```

Specify your email address at a global level.

```bash
git config user.email "your_email@example.com"
```

Specify your email address per repository.

```bash
git config user.email "your_email@example.com"
```

Use a template for you commit messages.

```bash
git config --global commit.template ~/.gitmessage 
```

# Using Multiple Git Accounts and SSH Keys

Assume you have a company specific GitHub account and a personal GitHub account.

By default you cannot use both accounts via SSH because a private key can only belong to a single account. If you attempt to upload to a second account, GitHub will complain.

You need to use two SSH keys instead. You can alter your SSH config to use a specific SSH key for your personal projects and use the default SSH key for all other projects.

* Add the following to your `~/.ssh/config` files

```text
Host gitaspersonal
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_ed25519_personal
  IdentitiesOnly yes
```

* Change the git remote of your personal projects to reflect the `gitaspersonal` host

```bash
git remote set-url origin git@gitaspersonal:vvanouytsel/myproject.git 
```

Happy pushing!

# Changing a commit message of an older commit

If for some reason you need to change a commit message of an older commit, you can do by rewriting your history. You should only do this on branches and never on your main branch.

## Without merge commits

If you do not use 'merge' commits but instead use rebase to merge changes from main into your branch, it is very easy to change any previous commit message.

```sh
$ git rebase -i HEAD~3

  1 pick 9e16d8c fix: change me
  2 pick f5ed786 fix: some message    
  3 pick edd3248 fix: this is the latest commit
```

The above command starts an interactive rebase of the last 3 commits. The commit at the bottom is the latest commit.

Using interactive rebasing, you can use a multitude of keywords for each commit. The ones I use the most are listed below.

| Identifier | Keyword | Description                                                                                                                                                                   |
| ---------- | ------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| p          | pick    | use the commit                                                                                                                                                                |
| r          | reword  | use the commit  but change the commit message                                                                                                                                 |
| s          | squash  | use the commit but meld it into  the previous commit                                                                                                                          |
| f          | fixup   | like `squash` but keep only the previous commit's log message. Unless `-C` is used, in which case keep only this commit's message; `-c` is same as `-C` but opens the editor. |

Once done, you can force push your changes to your branch.

## With merge commits

When using merge commits, it becomes a little bit more complex.
If that is the case you need to follow the steps below.

* Find the commit SHA of the commit that occurred right before the commit you want to change.
* Start an interactive rebase using `--rebase-merges` of the commit SHA right before the commit you want to change.

```bash
$ git rebase -i --rebase-merges aa3e0ef024d70b0a3470f75984bc9244caf26aa8
```

* Find the commit you want to change, use `fixup` and change `-c` to `-C`
* Change the commit message in the opened editor
* Solve any merge conflicts that might occur and continue with `git rebase --continue`, possibly resulting in more merge conflicts. Keep solving them and continuing.
* Force push your changes to your branch
