class: center, middle
# Introduction to Kubernetes

Andreas Elmertoft ([@elmertoft](https://twitter.com/elmertoft))  
Kevin Simper ([@kevinsimper](https://twitter.com/kevinsimper))

[http://kubernetes.dkrcph.com](http://kubernetes.dkrcph.com)

---
class: center, middle
# what we represent
---
class: center, middle
# What is Kubernetes?
"an open-source platform for automating deployment, scaling, and operations of application containers across clusters of hosts"
---
class: middle
# History

- Published in 2014
- Builds upon more than 10 years experience of managing containers within Google
---
class: center, middle
# who is the target?
## developers, devops, ops?
---
class: center, middle
![](http://i.imgur.com/wWbzzAR.png)
---
class: middle
# what are we going to learn?
- 3 very easy task
- 1 very complicated
---
class: center, middle
![](http://i.imgur.com/4xOVeut.png)
---
class: center, middle
# how are we going to do it?
---
class: middle
# Concepts

- pods
- replication controllers
- services
---
class: middle
## Pods

A Pod is one or more containers that must be scheduled onto the same host.

Communicates on the same network. That means that they can communicate on localhost.
---
class: middle
## Replication Controllers

A Replication controller is despite its name a simple program that ensures that a container is replicated a number of times.
---
class: middle
## Services

A service is a load-balancer that routes traffic to pods.

It works across nodes.
