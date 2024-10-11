# Kubernetes FEDeRATED Node

Helm scripts for deploying FEDeRATED node on a Kubernetes cluster with/without GraphDB and/or Corda Client API

## Requirements

- Kubernetes cluster (currently tested with Azure Kubernetes Engine)
    - Preferably set up a Nginx Ingress, certmanager, public IP Address and DNS name along with your cluster, so that HTTPS access can be made to the deployed client node.
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

- Preferred way is to create a values override file for a specific purpose. Then run helm upgrade using two value files. For example: 
```
helm upgrade --install -n DEPLOYMENT_NAMESPACE -f values.yaml -f specific-values.yaml DEPLOYMENT_NAME .
```
- (For Azure) For the Corda Node, use the Public IP Address created above.
- Enable the GraphDB and/or Spring API deployment as needed, based on the key 'enabled' for each block in the `values.yaml` file in the root directory.
- The Corda Client API Ingress hostname must be configured as required. If the LetsEncrypt ClusterIssuer has been deployed, then TLS will automatically be generated and configured. (See template for details). By default, it is enabled for easy viewing.
- If needed Change the user(name) and password values in the federated-corda-node (in the security section) and the 
  federated-node-api (in the federated section) but keep them matched to ensure access from the API to the Corda 
  node.  
 



### Installation

- Run the following command to cache all charts as needed.
```
helm dep update
```
 
- Run the following to install the marked components.

```
helm upgrade --install -n DEPLOYMENT_NAMESPACE --create-namespace DEPLOYMENT_NAME .
```
For example: 

```
helm upgrade --install -n federated --create-namespace demo-node1 .
```

- Use Port-Forwarding with `kubectl` to open a connection to the GraphDB database.
For example, the port-forwarding command might look like (run in Command prompt in the correct kube-context):

```
kubectl -n DEPLOYMENT_NAMESPACE port-forward svc/DEPLOYMENT_NAME-graphdb 7200:7200
```
A repository must be initiated in the GraphDB container, with the name defined in the `corda-node.graphdb.repository` key in the `values.yaml` file and default config parameters.

### Uninstallation 
- Run the following script to clear all deployed resources. 
```
helm uninstall -n DEPLOYMENT_NAMESPACE DEPLOYMENT_NAME
``` 
For example, 
```
helm uninstall -n federated demo-node1
```
- For a clean uninstall, it may help to also delete the namespace entirely.

### Testing the FEDeRATED Node endpoints
Please refer to [this chapter in the Docker version of the BDI node](https://github.com/Federated-BDI/Docker-BDI-Node#testing-the-bdi-api).


## Note
- The provided Helm scripts are provided in a rough state and may change.
- Currently the steps to designate a node as a Notary are manually done through the command-line as described in the official Corda node documentation. Due to limitations of Kubernetes, we do not provide an automated way to set up a Notary.
