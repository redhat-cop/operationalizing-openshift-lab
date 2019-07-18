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

## Iteration 1: Initial rollout of cluster configuration

When OpenShift gets installed, a _cluster id_ is generated from the `name` passed in the install-config.yaml, as well as a randomized uid that the installer generates. Several resources we are going to apply need that cluster id value, so we'll grab it and set it to a local variable.

    cluster_id=$(oc get machinesets -n openshift-machine-api -o jsonpath='{.items[0].metadata.labels.machine\.openshift\.io\/cluster-api-cluster}')

OK, now let's roll out our first config. We're going to do this using OpenShift Applier.

    git clone <this repo>
    cd operationalizing-openshift-lab
    ansible-galaxy install -r requirements.yml -p galaxy
    ansible-playbook -i .applier/ galaxy/openshift-applier/playbooks/openshift-cluster-seed.yml -e clusterid=${cluster_id}
