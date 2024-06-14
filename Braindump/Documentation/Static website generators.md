#documentation

The documentation source is stored in Git. This works, but when consulting the documentation on other devices I will have to constantly pull and push from the git repository. This is not ideal. A better solution would be to have the pipeline convert the markdown files to HTML files and serve these HTML files directly instead. We can use static website generators for that.

You can find a lot of these via [Jamstack](https://jamstack.org/generators/).
## Jekyl

[Jekyl](https://jekyllrb.com/) is a [Ruby](https://www.ruby-lang.org/en/) based library to create static websites.

## Quartz

[Quartz](https://quartz.jzhao.xyz/) is a fast, batteries-included static-site generator that transforms Markdown content into fully functional websites. It is written in. It requires [Node.js](https://nodejs.org/en) to run. We've got a whole [[Quartz|page]] dedicated how to set this up.

## MkDocs

[MkDocs](https://www.mkdocs.org/) is a **fast**, **simple** and **downright gorgeous** static site generator that's geared towards building project documentation. We are already using this for [jetspotter](https://vvanouytsel.github.io/jetspotter/).

## Docusaurus
[Docusaurus](https://docusaurus.io/) is running on [Node.js](https://nodejs.org/en) and also looks nice.

