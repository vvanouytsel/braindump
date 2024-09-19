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

## Using Multiple Git Accounts and SSH Keys

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