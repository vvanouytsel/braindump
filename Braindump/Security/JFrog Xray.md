---
tags:
  - sbom
  - software-composition-analysis
  - security
---

JFrog has a tool named `xray` which can be used to perform software composition analysis.
JFrog has its own vulnerability database that it uses to interact with Xray.

## Concepts

### Policies

Policies define security and license compliance behavior specifications. Policies enable you to create a set of [[#Rules|rules]], in which each rule defines a license/security criteria, with a corresponding set of automatic actions according to your needs.

Policies are enforced when they are applied to [[#Watches|watches]].

### Rules

A rule defines a range of [CVSS values](https://www.sans.org/blog/what-is-cvss/).
A rule defines what action has to be taken one a vulnerability is detected. There is a multitude of actions you can take:
* Trigger a webhook
* Notify people
* Block downloads
* Fail builds
* Create Jira tickets
* ...

### Watches

A watch is the link between a set of resources and a set of policies.
These resources could be repositories, builds and bundles.

A watch can be triggered manually, upon update of the vulnerability database or if the content of the resources are updated.

A watch violation is created whenever a violation of the policies attached to the watch occurs.

