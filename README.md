# Kubernetes BDI Node (In progress)

Helm scripts for deploying BDI node on a Kubernetes cluster with/without GraphDB and/or Corda Client API

## Requirements

- Kubernetes cluster (currently tested with Azure Kubernetes Engine)
    - Preferably set up an Nginx Ingress, certmanager, public IP Address and DNS name along with your cluster, so that HTTPS access can be made to the deployed client node.
- Kubectl (configured with access to the cluster)
- Helm

## Overview of Deployed Components
Three nodes are deployed as required.

- Corda Node (with developed CorDapps)
- GraphDB
- Spring Boot Client Server

## Instructions

The Corda nodes communicate over TCP, which is not supported through an Nginx ingress in a Kubernetes cluster. 
There are different approaches that exist to direct Level 4 traffic towards the container, but for the purposes of this deployment, we use Load Balancers automatically set up in the Azure cloud by the Azure Kubernetes Engine.

### Azure-specific Pre-requisites
- Create a Public IP Address in the Azure Portal (in the same resouce group as the Kubernetes cluster). Note the IP Address for later.
- Once created, go to IAM for the generated Public IP Address resource, and give Network Contributor role access to the Kubernetes Cluster (find by name).
- This configuration allows the Kubernetes cluster to set up a Load Balancer resource that connects to this IP Address.
- It is then possible to optionally set up a DNS entry that points towards this IP Address.

### Configuration

- Edit values.yaml as per the documentation provided. Replace all the marked values with appropriate
    - (For Azure) For the Corda Node, use the Public IP Address created above.
    - Enable the GraphDB and/or Spring API deployment as needed, based on the key 'enabled' for each block in the `values.yaml` file in the root directory.

### Installation

- Run `helm dep update` to cache all charts as needed.
- Run `helm upgrade --install -n DEPLOYMENT_NAMESPACE --create-namespace DEPLOYMENT_NAME .` to install the marked components. For example, `helm upgrade --install -n bdi-node --create-namespace bdi-node .`.

## Future updates
- Support ingress support for Client API, currently unsupported.
- Support deployment of node as Notary

## Note
The provided Helm scripts are provided in a rough state.