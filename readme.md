The Aqua Security Operator runs within an OpenShift cluster and is responsible for deploying and managing the Aqua Security platform and components:
* Server (aka “console”)
* Database (for production environments we recommend to use an external database and not the Aqua default database)  
* Gateway 
* Enforcer (aka “agent”)
* Scanner
* CSP (a simple package that contains the Server, Database, and Gateway)

Use the Aqua-Operator to: 
* Deploy Aqua Security on OpenShift
* Scale up Aqua Security components with extra replicas
* Assign metadata tags to Aqua Security components
* Automatically scale the number of Aqua scanners based on the number of images in the scan queue
	
The Aqua Operator provides a few Custom Resources that enable the flexibility to insatll Aqua in different configurations. Please make sure to read the Aqua installation manual (https://docs.aquasec.com/docs) before using the Operator. For advance configurations please consult with Aqua's support team.

## Prerequisits
Make sure you have a license and access to the Aqua registry. Please contact our sales team if you want to aquire a license for the product -  https://www.aquasec.com/about-us/contact-us/

## Installing the Operator 
1. Create a new project/namespace called 'aqua' for the Aqua service 
2. Install the Aqua Opertor from the OperatorHub and add it to the 'aqua' namespace

## Before you deploy the CRDs
You need to supply two secrets during for the deployment  - 
* A secret for the Docker registry
* A secret for the database

You can  list the secrets in the YAML files or you can define secrets in the OpenShift project (see example below) -
```bash
oc create secret docker-registry aqua-registry --docker-server=registry.aquasec.com --docker-username=<AQUA_USERNAME> --docker-password=<AQUA_PASSWORD> --docker-email=<user email> -n aqua
oc create secret generic aqua-database-password --from-literal=db-password=<password> -n aqua
oc secrets add aqua-sa aqua-registry --for=pull -n aqua
```

3. Use the AquaCSP Custom Resource to install Aqua in your Cluster. Aqua CSP will automatically deploy the Console, Database, Scanner, and Gateway Custom Resource
4. To install the Enforcer Custom Resource, you will need to access Aqua and create a new Enforcer Group. Copy the group's 'token' as you will need it later for the Enfrocer Custom Resource
5. Use the AquaEnforcer Custom Resource to install the Aqua Enforcer. Replace the value of the 'token' property with the token you kept from step 3
	
## Installing AquaCSP
Use the AquaCSP Custom Resource to install Aqua in your Cluster. AquaCSP will automatically deploy the console, dtabase, scanner, and gateway custom-resources. You can deploy Aqua's enforcers automaticly by defining the enforcer field in the AquaCSP CR file. If you don't want to deploy the enforcers automatically, you shoul access Aqua's concole and create a new Enforcer Group. Copy the group's 'token' and use it in the AquaEnforcer Custom Resource to install the Aqua Enforcer. Replace the value of the 'token' property with the token you copied from Aqua.  

There are multiple options to deploy the Aqua CSP. You can review the different options in the following [file](https://github.com/aquasecurity/aqua-operator/blob/master/deploy/crds/operator_v1alpha1_aquacsp_cr.yaml).  

* You can choose the Aqua version to deploy in the 'version' property
* You can create a Route automatially by setting the 'route' property to 'true'
* Note that by default Aqua's console and gateway will use a ClusterIP service.

Here is an example of a simple deployment  - 
```yaml
---
apiVersion: operator.aquasec.com/v1alpha1
kind: AquaCsp
metadata:
  name: aqua
  namespace: aqua
spec:
  infra:                                    
    serviceAccount: "aqua-sa"               
    namespace: "aqua"                       
    version: "4.6"                          
    requirements: true                      
  common:
    imagePullSecret: "aqua-registry"        # Optional: if already created image pull secret then mention in here
    dbDiskSize: 10       
    serverDiskSize: 4   
  database:                                 
    replicas: 1                            
    service: "ClusterIP"                    
  gateway:                                  
    replicas: 1                             
    service: "ClusterIP"                    
  server:                                   
    replicas: 1                             
    service: "NodePort" 
```

You can define a Route to enable external access to Aqua's console.

## Installing AquaEnforcer
You can review the different options to implement AquaEnforcer in the following [file](https://github.com/aquasecurity/aqua-operator/blob/master/deploy/crds/operator_v1alpha1_aquaenforcer_cr.yaml).

Here is an example of a simple deployment  - 
```yaml
---
apiVersion: operator.aquasec.com/v1alpha1
kind: AquaEnforcer
metadata:
  name: aqua
spec:
  infra:                                    
    serviceAccount: "aqua-sa"                
    version: "4.6"                          # Optional: auto generate to latest version
  common:
    imagePullSecret: "aqua-registry"            # Optional: if already created image pull secret then mention in here
  deploy:                                   # Optional: information about aqua enforcer deployment
    image:                                  # Optional: if not given take the default value and version from infra.version
      repository: "enforcer"                # Optional: if not given take the default value - enforcer
      registry: "registry.aquasec.com"      # Optional: if not given take the default value - registry.aquasec.com
      tag: "4.6"                            # Optional: if not given take the default value - 4.5 (latest tested version for this operator version)
      pullPolicy: "IfNotPresent"            # Optional: if not given take the default value - IfNotPresent
  gateway:                                  # Required: data about the gateway address
    host: aqua-gateway
    port: 8443
  token: "<<your-token>>"                            # Required: enforcer group token also can use an existing secret instead
```
