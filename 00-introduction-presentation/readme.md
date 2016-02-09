class: center, middle
# Introduction to Kubernetes

Andreas Elmertoft ([@elmertoft](https://twitter.com/elmertoft)  
Kevin Simper [@kevinsimper](https://twitter.com/kevinsimper)

[http://kubernetes.dkrcph.com](http://kubernetes.dkrcph.com)

---

# Concepts

- pods
- replication controllers
- services
---
## Pod

A Pod is one or more containers that must be scheduled onto the same host.

Communicates on the same network. That means that they can communicate on localhost.
---
## Replication Controller  

A Replicationcontroller is despite its name a simple program that ensures that a container is replicated a number of times.
---
## Service

A service is a load-balancer that routes traffic to pods.

It works across nodes.
