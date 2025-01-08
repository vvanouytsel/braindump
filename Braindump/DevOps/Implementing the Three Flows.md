---
tags:
  - devops
---

In order to bring the [[Principles|theory]] to practice we will need to build implementations.

# [[Principles#The First Way The principles of flow|The First Way]]

## Enable On-Demand creation of environments

The sooner a developer can use a production-like system to test his code on, the better. This allows us to detect issues early in the chain instead of during deployment to production.

Creation of these environments should be seamless and completely automated. These environments should be considered as cattle and not as pets. All configuration should be stored in git. If an environment is in a broken state, you simply shoot it down and spin up a new one. All of this should be possible without Ops intervention.

## The Deployment Pipeline

A pipeline needs to be put in place that is automatically started with every commit. The pipeline should be considered holy and the whole engineering department should treat them as such. It is completely fine to break them, they exist to prevent defects to reach downstream, it is only logical that it will eventually break. However, **whenever it breaks, the teams should stop their work and make sure the build gets fixed.** This is a cultural thing.

The pipeline should include the following steps:
* Building the artifact
* Automated unit tests
* Static code analysis
* Duplication and test coverage analysis
* Style checking
* Deployment into production like environment
* Automated acceptance tests

Whatever the artifact is that you build, you only build it once and store it in an artifact repository. You will re-use the same artifact in any other later stages of the deployment pipeline.

## Fast and Reliable Automated Test Suite

Tests are a necessity. There are different type of tests that can be written. All of them have value but are used for different purposes.

* **Unit tests:** Tests a single method, class or function. Checks if the code does what the developer intended it to do.
* **Acceptance tests:** A functional test to verify that the application does what the user/customer expects it to do.
* **Integration tests:** Ensures that the application correctly interacts with other applications.
* ...

Unit tests are extremely quick and lightweight to run. You want a lot of these and you want to run them very early in your [[Implementing the Three Flows#The Deployment Pipeline|pipeline]]. Integration tests are also required, but are much more expensive to run.

Whenever you find out that an integration test fails, you should think critically and investigate if it is possible to have found the same defect using a unit test. If the answer is yes, write the unit test.

More extensive tests such as performance or load tests can be run either continually or on a schedule. These should not be part of your [[Implementing the Three Flows#The Deployment Pipeline|pipeline]] by default, as they often take a long time to complete.
