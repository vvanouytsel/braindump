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