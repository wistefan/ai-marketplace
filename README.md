# AI-Marketplace data space demonstrator

App-of-apps for the marketplace

This fork extends the standard installation of the KI-Marktplatz data marketplace 
by components for the data sharing Proof-of-Concepts (PoCs).

> :bulb: This repository just provides a setup for temporary demonstration purposes. It is not recommended to be used in a production enviroment. Credentials are visible in clear text and are not encrypted. Installations should be deleted when demonstrations/presentations/etc. have finished. 

The GitHub actions of this repo are configured to deploy a full instance with all components 
required for this PoC, as soon as a branch is created. It is meant for a temporary deployment only. 
Note that the deployment should be deleted after 
each presentation/demo/etc., since there are only test accounts registered and credentials are visible in clear text in this 
repo.

Before moving this installation to a production environment, make sure to encrypt all credentials, keys, etc., e.g., 
using [sealed-secrets](https://github.com/bitnami-labs/sealed-secrets).

All scripts are developed for using an OpenShift Kubernetes cluster, but can be easily adapted for any 
kind of infrastructure.


## Deployment

It is required to setup two GitHub secrets in the 
repository ([also check this manual](https://github.com/FIWARE-Ops/marinera/blob/main/documentation/GITHUB_CI.md#openshift-service-account-permissions)):
* `OPENSHIFT_SERVER`: Server URL of the OpenShift cluster
* `OPENSHIFT_TOKEN`: Token from an OpenShift service account with sufficient permissions for creation/deletion of projects and applications, role assignments and deployments via Helm charts (e.g., with `cluster-admin` role) 

In order to deploy all components, simply create a branch which is named differently than `main`. 
The GitHub action will deploy all components to the namespace `kim-{BRANCH_NAME}`. 

Routes for externally exposed services are automatically created and hostnames are set dynamically. In order to 
retrieve the created hostnames, one can run, e.g., 
```shell
kubectl -n kim-{BRANCH_NAME} get routes
```
For the marketplace, when the branch was called `poc`, this might give you something like
```shell
NAME                                      HOST/PORT                                                                PATH   SERVICES                                PORT    TERMINATION     WILDCARD
marketplace-biz-ecosystem-logic-proxy-0   marketplace-biz-ecosystem-logic-proxy-0-kim-poc.apps.fiware.fiware.dev          marketplace-biz-ecosystem-logic-proxy   <all>   edge/Redirect   None
```
The marketplace logic proxy would be available under the URL: `https://marketplace-biz-ecosystem-logic-proxy-0-kim-poc.apps.fiware.fiware.dev`.


### Uninstall

For removing all components and deleting the applications and namespace, simply remove the branch.



## Credentials

Different accounts are created automatically with default passwords.

| Component     | Username               | Password          | Comment |
|---------------|------------------------|-------------------|---------|
| Keyrock Marketplace | admin@test.com | admin | Admin user of the marketplace |
| Keyrock Provider | admin@test.com | admin | Admin user of the Provider Keyrock IDP |
| Keyrock Provider | operator@provider.com | operator | Operator employee user of the Provider |
| Keyrock Automotive Supplier | admin@test.com | admin | Admin user of the Automotive Supplier Keyrock IDP |
| Keyrock Automotive Supplier | operator@autosupplier.com | operator | Operator employee user of Automotive Supplier |
| Keyrock Automotive Supplier | user@autosupplier.com | user | Standard user of Automotive Supplier |
| Keyrock Car Dealer | admin@test.com | admin | Admin user of the Car Dealer Keyrock IDP |
| Keyrock Car Dealer | operator@cardealer.com | operator | Operator employee user of Car Dealer |
| Keyrock Car Dealer | user@cardealer.com | user | Standard user of Car Dealer |

Root CA, keys and certificates have been created and self-signed using openssl. Keys and certificates used for this PoC 
can be found in the [certs folder](./certs). These should never be used in any kind of production enviroment or on a 
contineously running environment.  
Below table displays the assigned EORIs assigned to the different organisations and their keys/certificates:
| Organisation           | EORI                       |
|------------------------|----------------------------|
| Satellite              | EU.EORI.FIWARESATELLITE    |
| Marketplace            | EU.EORI.DEMARKETPLACE      |
| Hella                  | EU.EORI.DEHELLA            |
| Automotive Supplier    | EU.EORI.DEAUTOSUPPLIER     |
| Car Dealer             | EU.EORI.DECARDEALER        |


## Service endpoints

The Hella service provider offers endpoints to access data via the different data space types. 


### i4Trust

When using the i4Trust data space, the NGSI-LD endpoint is reacheable via `/diagnosis-i4trust/ngsi-ld/v1/entities`. 

Example call:
```shell
curl -X GET --header "Authorization <JWT>" https://hella-kong-kong-kim-poc.apps.fiware.fiware.dev/diagnosis-i4trust/ngsi-ld/v1/entities/<ENTITY-ID>
```
where `<JWT>` is a signed [iSHARE JWT](https://dev.ishareworks.org/introduction/jwt.html) access token issued by an IDP or by the service provider.

There is an example script [get_data_m2m_i4trust.py](./scripts/get_data_m2m_i4trust.py) which automatizes the process of obtaining an access token 
and retrieving diagnosis data at the Kong instance of the service provider. This should be run on behalf of the service consumer 
organisation after acquisition of the access rights. It will only work with the FIWARE Kubernetes cluster. Usage:
```shell
python scripts/get_data_m2m_i4trust.py <NAMESPACE> <PARTY>
```
where `<NAMESPACE>` denotes the mandatory parameter of the deployed namespace (e.g., `kim-{BRANCH_NAME}`) and 
`<PARTY>` is the optional parameter of the consuming party (default: `autosupplier`, other options: `cardealer`).
