---
tags:
  - devops
---

# The Three Ways

The Three Ways are a set of principles that all DevOps processes derive from.
![[Pasted image 20241223120341.png]]

## The First Way: The principles of flow

The goal of the First Way is to improve the flow of work from development towards delivering value to the customer.

Flow can be increased by the following steps:
* Make the work visible
* Limit the amount of work in progress
* Reduce the batch sizes
* Reduce the amount of hand offs

### Implementations

#### Feature toggles

By using feature toggles you can commit often and thus introduce smaller changes to the system. Once the feature is complete, the feature toggle can be enabled.

## The Second Way: The principles of feedback

The Second Way is all about shortening the feedback loops. This allows employees to quickly discover any defects early in the chain.

To shorten the feedback loop we should:
* See problems as they occur by running tests
* Swarm and solve problems to contain these before they can spread
* Push quality closer to the source

### Implementations

#### Continuous Integration

By introducing CI systems we can continually evaluate the quality of our created code. These systems should be designed to give us a short feedback loop. If these builds fail, the feedback should reach the responsible teams and effort should be taken to swarm the problem to reduce it from spreading further.

## The Third Way: The principles of continual learning and experimentation

The Third Way mentions the creation of a culture of continual learning and experimentation. Individual knowledge needs to be turned into team and organizational knowledge. Constantly reiterate the current processes and look for improvements.

We can do this by:
* Making time to clean up technical debt
* Share individual or team learning across the organization
* Keep seeking to reduce lead times
* Introduce controlled failures in systems to make them more resilient
* Never stop learning

### Implementations

#### Blameless postmortems

Failures will always happen. Instead of fearing them, we should embrace them and learn from them. In the event of a failure, a blameless postmortem can be written. The goal of these is to spread knowledge and think about possible solutions to prevents these from happening in the future.
