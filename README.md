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
    - The Corda Client API Ingress hostname must be configured as required. If the LetsEncrypt ClusterIssuer has been deployed, then TLS will automatically be generated and configured. (See template for details). By default, it is enabled for easy viewing.

### Installation

- Run the following command to cache all charts as needed.
```
helm dep update
```
 
- Run the following to install the marked components.

```
helm upgrade --install -n DEPLOYMENT_NAMESPACE --create-namespace DEPLOYMENT_NAME .
```
For example, 

```
helm upgrade --install -n bdi-node --create-namespace bdi-node .
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
helm uninstall -n bdi-node bdi-node
```
- For a clean uninstall, it may help to also delete the namespace entirely.

### Sample call

Either use the configured `HOSTNAME` from the `values.yaml` with a deployed Ingress resource, or use kubectl port-forwarding on the Corda Client API to a localhost port of your choice. For example, let's assume that we did deploy an Ingress with hostname `corda-node.example`.


Open `http://corda-node.example/swagger-ui.html` in your browser. A swagger UI should appear.
Under Corda details, one can query the Corda node what nodes it knows. It should know at least one notary (GET `/node/notaries`) and a few other nodes (GET `/node/peers`).


Play around with the `/events` calls too. In case you are prompted for an access token, you can use your iShare instance – make sure you configured it under `database.properties`, together with GraphDB, in case – or enter `Bearer doitanyway` to skip this.


Try to submit the following new events.


```json
{
"fullEvent": "@base <http:\/\/example.com\/base\/> . @prefix pi: <https:\/\/ontology.tno.nl\/logistics\/federated\/PhysicalInfrastructure#> . @prefix classifications: <https:\/\/ontology.tno.nl\/logistics\/federated\/Classifications#> . @prefix dcterms: <http:\/\/purl.org\/dc\/terms\/> . @prefix LogisticsRoles: <https:\/\/ontology.tno.nl\/logistics\/federated\/LogisticsRoles#> . @prefix rdfs: <http:\/\/www.w3.org\/2000\/01\/rdf-schema#> . @prefix owl: <http:\/\/www.w3.org\/2002\/07\/owl#> . @prefix Event: <https:\/\/ontology.tno.nl\/logistics\/federated\/Event#> . @prefix ReusableTags: <https:\/\/ontology.tno.nl\/logistics\/federated\/ReusableTags#> . @prefix businessService: <https:\/\/ontology.tno.nl\/logistics\/federated\/BusinessService#> . @prefix DigitalTwin: <https:\/\/ontology.tno.nl\/logistics\/federated\/DigitalTwin#> . @prefix skos: <http:\/\/www.w3.org\/2004\/02\/skos\/core#> . @prefix xsd: <http:\/\/www.w3.org\/2001\/XMLSchema#> . @prefix ex: <http:\/\/example.com\/base#> . @prefix time: <http:\/\/www.w3.org\/2006\/time#> . @prefix dc: <http:\/\/purl.org\/dc\/elements\/1.1\/> . @prefix era: <http:\/\/era.europa.eu\/ns#> .  ex:Event-b550739e-2ac2-4c21-9a56-e74791313375 a Event:Event, owl:NamedIndividual;   rdfs:label \"GateOut test\", \"Planned gate out\";   Event:hasTimestamp \"2019-09-22T06:00:00Z\"^^xsd:dateTime;   Event:hasDateTimeType Event:Planned;   Event:involvesDigitalTwin ex:DigitalTwin-f7ed44a4-0ac1-42fc-820b-765bb2a70def, ex:Equipment-a891b64d-d29f-4ef2-88ad-9ec4c88e0833;   Event:involvesBusinessTransaction ex:businessTransaction-a891b64d-d29f-4ef2-88ad-9ec4c88e0833;   Event:hasMilestone Event:START;   Event:hasSubmissionTimestamp \"2019-09-17T23:32:07Z\"^^xsd:dateTime .  ex:DigitalTwin-f7ed44a4-0ac1-42fc-820b-765bb2a70def a DigitalTwin:TransportMeans,     owl:NamedIndividual .  ex:businessTransaction-a891b64d-d29f-4ef2-88ad-9ec4c88e0833 a businessService:Consignment,     owl:NamedIndividual;   businessService:consignmentCreationTime \"2021-05-13T21:23:04Z\"^^xsd:dateTime;   businessService:involvedActor ex:LegalPerson-Maersk .  ex:LegalPerson-Maersk a businessService:LegalPerson, owl:NamedIndividual, businessService:PrivateEnterprise;   businessService:actorName \"Maersk\" .  ex:Equipment-a891b64d-d29f-4ef2-88ad-9ec4c88e0833 a DigitalTwin:Equipment, owl:NamedIndividual;   rdfs:label \"MNBU0494490\" .",
"countriesInvolved": [

]
}
```

```json
{
  "fullEvent": "@base <http:\/\/example.com\/base\/> . @prefix pi: <https:\/\/ontology.tno.nl\/logistics\/federated\/PhysicalInfrastructure#> . @prefix classifications: <https:\/\/ontology.tno.nl\/logistics\/federated\/Classifications#> . @prefix dcterms: <http:\/\/purl.org\/dc\/terms\/> . @prefix LogisticsRoles: <https:\/\/ontology.tno.nl\/logistics\/federated\/LogisticsRoles#> . @prefix rdfs: <http:\/\/www.w3.org\/2000\/01\/rdf-schema#> . @prefix owl: <http:\/\/www.w3.org\/2002\/07\/owl#> . @prefix Event: <https:\/\/ontology.tno.nl\/logistics\/federated\/Event#> . @prefix ReusableTags: <https:\/\/ontology.tno.nl\/logistics\/federated\/ReusableTags#> . @prefix businessService: <https:\/\/ontology.tno.nl\/logistics\/federated\/BusinessService#> . @prefix DigitalTwin: <https:\/\/ontology.tno.nl\/logistics\/federated\/DigitalTwin#> . @prefix skos: <http:\/\/www.w3.org\/2004\/02\/skos\/core#> . @prefix xsd: <http:\/\/www.w3.org\/2001\/XMLSchema#> . @prefix ex: <http:\/\/example.com\/base#> . @prefix time: <http:\/\/www.w3.org\/2006\/time#> . @prefix dc: <http:\/\/purl.org\/dc\/elements\/1.1\/> . @prefix era: <http:\/\/era.europa.eu\/ns#> .  ex:Event-7f0140f7-1c22-4b68-9bea-25418cd51d18 a Event:Event, owl:NamedIndividual;   rdfs:label \"GateOut test\", \"Planned gate out\";   Event:hasTimestamp \"2019-09-22T06:00:00Z\"^^xsd:dateTime;   Event:hasDateTimeType Event:Planned;   Event:involvesDigitalTwin ex:DigitalTwin-f7ed44a4-0ac1-42fc-820b-765bb2a70def, ex:Equipment-a891b64d-d29f-4ef2-88ad-9ec4c88e0833;   Event:involvesBusinessTransaction ex:businessTransaction-a891b64d-d29f-4ef2-88ad-9ec4c88e0833;   Event:hasMilestone Event:End;   Event:hasSubmissionTimestamp \"2019-09-17T23:32:07Z\"^^xsd:dateTime .  ex:DigitalTwin-f7ed44a4-0ac1-42fc-820b-765bb2a70def a DigitalTwin:TransportMeans,     owl:NamedIndividual .  ex:businessTransaction-a891b64d-d29f-4ef2-88ad-9ec4c88e0833 a businessService:Consignment,     owl:NamedIndividual;   businessService:consignmentCreationTime \"2021-05-13T21:23:04Z\"^^xsd:dateTime;   businessService:involvedActor ex:LegalPerson-SomeShipper .  ex:LegalPerson-SomeShipper a businessService:LegalPerson, owl:NamedIndividual, businessService:PrivateEnterprise;   businessService:actorName \"SomeShipper\" .  ex:Equipment-a891b64d-d29f-4ef2-88ad-9ec4c88e0833 a DigitalTwin:Equipment, owl:NamedIndividual;   rdfs:label \"ABCDE\" .",
  "countriesInvolved": [
    
  ]
}
```
You can also send these events to other nodes, by entering their country code in the `countriesInvolved` list. To do this, the nodes need to be able to communicate on the p2p address configured in `node.conf`.

## Note
- The provided Helm scripts are provided in a rough state and may change.
- Currently the steps to designate a node as a Notary are manually done through the command-line as described in the official Corda node documentation. Due to limitations of Kubernetes, we do not provide an automated way to set up a Notary.