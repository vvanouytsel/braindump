---
tags:
  - devops
---

# Branching strategies

## Feature based

A developer works isolated from the main branch until the feature is complete. Once development is done, the branch is merged into the main branch.

| Benefits                                            | Challenges                                            |
| --------------------------------------------------- | ----------------------------------------------------- |
| Completely isolated                                 | Complexity the longer the branch lives                |
| Multiple features can be developed at the same time | Integration only happens as soon as you merge to main |
| Stable main branch                                  |                                                       |

## Trunk based

A developer pulls and commits to the main branch multiple times per day. Either no, or very short lived branches are used.

| Benefits                                                            | Challenges                                                                 |
| ------------------------------------------------------------------- | -------------------------------------------------------------------------- |
| Small feedback loop due to frequent integration                     | Discipline of developers is needed to integrate often and keep main stable |
| Merge conflicts are easier to solve as the code changes are smaller | Automated testing is required to guarantee quality of the main branch      |
| Everyone is using the latest version of the codebase                | Mindset to immediately fix main in case of failing builds is required      |

