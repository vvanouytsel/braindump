[Quartz](https://quartz.jzhao.xyz/) is a fast, batteries-included static-site generator that transforms Markdown content into fully functional websites. Thousands of students, developers, and teachers are [already using Quartz](https://quartz.jzhao.xyz/showcase) to publish personal notes, websites, and [digital gardens](https://jzhao.xyz/posts/networked-thought) to the web.

[How to publish your notes for free with Quartz - YouTube](https://www.youtube.com/watch?v=6s6DT1yN4dw)

## Installation

```bash
git clone https://github.com/jackyzha0/quartz.git
cd quartz
npm i
npx quartz create
```

The `quartz` git submodule lives in the same repository as your documentation. Your markdown files should be stored in the `/content` directory. However, this will have the effect that your Obsidian vault will also the `content` directory. You can instead place your markdown files in a separate directory and point `quartz` to that directory instead.

```bash
# Markdown files are stored in /Braindump directory
npx quartz build --directory=Braindump --serve
```

Syncing your changes to git is done via the `sync` command. This will commit all you changes and push it to your git repository.

```bash
npx quartz sync --directory=Braindump
```