# How to Apply Infrastructure as Code Principles to OpenShift 4 Operationalization

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

## Setting up a Git Ops Workflow

_Git Ops_ is a form of Infrastructure as Code practice where all of the configurations that define a system are managed in a git repository, and automatically and idempotently applied to that system each time commits are made to the repository. For this lab, we are going to enable such a workflow using an Operator called [Eunomia](https://github.com/KohlsTechnology/eunomia) and [OpenShift Applier](https://github.com/redhat-cop/openshift-applier), and automation framework for OpenShift. Eunomia provides a workflow for watching git repos, triggering actions when new commits are detected. The action, in this case, will be a [Kubernetes Job](https://kubernetes.io/docs/tasks/job/) that executes Applier.

Let's go ahead and deploy Eunomia. We can do that via provided helm charts.

    git clone https://github.com/KohlsTechnology/eunomia.git
    helm template eunomia/deploy/helm/prereqs/ | oc apply -f -
    helm template eunomia/deploy/helm/operator/ --set openshift.route.enabled=true | oc apply -f -

Now we can set up eunomia to monitor a repo of configs, which in turn will apply configs to set up our cluster. Eunomia provides a CR called a `GitOpsConfig` to set up a monitor on a repository. You can examine the one we're going to use at [templates/cluster-gitops.yaml](templates/cluster-gitops.yaml). Let's use Applier to roll out the config.

    git clone https://github.com/redhat-cop/operationalizing-openshift-lab
    cd operationalizing-openshift-lab
    ansible-galaxy install -r requirements.yml -p galaxy
    ansible-playbook -i .applier/ galaxy/openshift-applier/playbooks/openshift-cluster-seed.yml -e include_tags=gitops

This run resulted in a namespace called `cluster-config` with our `GitOpsConfig` in it. From this, Eunomia spins up a job. Let's wait for that job to complete, and then see what we have.

    oc get jobs -n cluster-config

You should see one job created in this namespace. To follow the progress of the job, we need to find the pod running the job.

    oc get pods -n cluster-config

We should see that there is a pod running. Let's tail the logs of that pod.

    oc logs -f gitopsconfig-cluster-config-2bromv-wks8d -n cluster-config

From this we should see that there is an OpenShift Applier playbook running. This will roll out the rest of the configurations stored in this repo. Once that job completes successfully, we can explore what has been set up for us.

## Cluster Exploration

### MachineSets and AutoScalers

When OpenShift gets installed, a _cluster id_ is generated from the `name` passed in the install-config.yaml, as well as a randomized uid that the installer generates. Several resources we are going to apply need that cluster id value, so we'll grab it and set it to a local variable.

TO BE CONTINUED...

### Authentication and Authorization

### LDAP Authentication

OpenShift provides the capability to authenticate users stored in LDAP systems using the [LDAP identity provider](https://docs.openshift.com/container-platform/4.1/authentication/identity_providers/configuring-ldap-identity-provider.html).

Obtain the following values for the LDAP server:

* LDAP Search URL
    * Used to specify how to connect to the LDAP server as well as define the Base DN to start searching the LDAP tree as well as any LDAP filters.
* Bind User (DN)
    * User to authenticate against the LDAP instance

Apply the configurations to the cluster:

    ansible-playbook -i .applier/ galaxy/openshift-applier/playbooks/openshift-cluster-seed.yml -e ldap_bind_dn="${ldap_bind_dn}" -e ldap_search_url="${ldap_search_url}" -e include_tags=ldap_auth

With the configurations applied, attempt to login with a user defined in the LDAP instance

    oc login -u <user> -p <password>

To login to the web console, locate the URL of the OpenShift web console and navigate to the URL presented

    oc whoami --show-console

Two login options are displayed. Select _LDAP_ and login with a valid username and password

If you are authenticated using both methods, the confgurations were LDAP authentication was successful.
