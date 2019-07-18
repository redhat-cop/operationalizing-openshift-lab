# How to Apply Infrastructure as Code Principals to OpenShift 4 Operationalization

In this lab we are going to develop an iterative and declarative approach to taking a new OpenShift Cluster from vanilla to operationalized. This repo should act as a resource for cluster administrators to begin the process of defining clusters as code.

In OpenShift 4, every cluster starts out the same way -- a bootstrapped control plane with 3 masters, and 3 compute nodes. But in order for this cluster to provide value to an organization, it needs further configuration. Using this repository as a starting point we are going to establish an automated strategy for managing configuration of an operationalized cluster, including things like:

* Enabling Cluster Autoscaling
* Configuring authentication and authorization
* Managing namespace creation
* Setting up quotas & limits for applications
* Applying automated certificate management for applications
* Deploying some initial workloads
* Deploying some custom dashboards and setting up alerts

Note that the purpose of this lab is not to teach how each of the components works individually, but to show how one can employ a common automation strategy to a myriad of different configurations.

Let's get started...
