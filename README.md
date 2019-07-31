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



To grab your cluster cloud region and set it to a local variable run the following:

    cloud_region=$(oc get machinesets -n openshift-machine-api -o jsonpath='{.items[0].spec.template.spec.providerSpec.value.placement.region}')



OK, now let's roll out our first config. We're going to do this using OpenShift Applier.

    git clone <this repo>
    cd operationalizing-openshift-lab
    ansible-galaxy install -r requirements.yml -p galaxy
    ansible-playbook -i .applier/ galaxy/openshift-applier/playbooks/openshift-cluster-seed.yml -e clusterid=${cluster_id} -e cloudregion=${cloud_region}

## Iteration 2: LDAP Authentication

OpenShift by default is installed with a default administrative user (called `kubeadmin`). [Identity Providers](https://docs.openshift.com/container-platform/4.1/authentication/understanding-identity-provider.html) can be configured in order to allow other users the ability to login to the cluster. Many organizations manage users using LDAP and OpenShift supports leveraging users defined in LDAP systems through an [LDAP identity provider](https://docs.openshift.com/container-platform/4.1/authentication/identity_providers/configuring-ldap-identity-provider.html).

Security is paramount and OpenShift can be configured to communicate with the LDAP server via secure mechanisms. In order for the OpenShift to trust the LDAP server, a CA certificate must be provided. Obtain the certificate and place it in a file called _ldap.ca_ as it will be referenced later in this section.

Obtain the following values for the LDAP server:

* LDAP Search URL
    * Used to specify how to connect to the LDAP server as well as define the Base DN to start searching the LDAP tree as well as any LDAP filters.
* Bind User
    * User to authenticate against the LDAP instance
* Bind Password
    * Password for the user to authenticate against the LDAP instance
* CA Certificate
    * Contents of the CA certificate for the LDAP instance

Let's use the OpenShift applier once again to apply the configurations to the cluster:

    ansible-playbook -i .applier/ galaxy/openshift-applier/playbooks/openshift-cluster-seed.yml -e "ldap_ca='$(cat ldap.ca)'" -e ldap_bind_password='${ldap_bind_password}' -e ldap_bind_dn="${ldap_bind_dn}" -e ldap_search_url="${ldap_search_url}" -e include_tags=ldap_auth

_Note: THe use of the `filter_tags` variable allows for a subset of the Applier inventory to be executed._

With the configurations applied, attempt to login with a user defined in the LDAP instance

    oc login -u <user> -p <password>

To login to the web console, locate the URL of the OpenShift web console and navigate to the URL presented

    oc get routes -n openshift-console console -o jsonpath="https://{.spec.host}"

Two login options are displayed. Select _LDAP_ and login with a valid username and password

If you are authenticated using both methods, the confgurations were LDAP authentication was successful.