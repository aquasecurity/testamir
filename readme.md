The Aqua Security Operator runs within an OpenShift cluster and is responsible for deploying and managing the Aqua Security platform and components :
* Server (aka “console”)
* Database (for production environments we recommend to use an external database and not the Aqua default database)  
* Gateway 
* Enforcer (aka “agent”)
* Scanner
* CSP (a simple package that contains the Server, Database, and Gateway)

Use the Aqua-Operator to : 
* Deploy Aqua Security components on OpenShift
* Scale up Aqua Security components with extra replicas
* Assign metadata tags to Aqua Security components
* Automatically scale the number of Aqua scanners based on the number of images in the scan queue
	
The Aqua operator provides multiple Custom Resrouces that enables the flexability to insatll Aqua in different confiugratoins. Please make sure to read the Aqua installation manual (https://docs.aquasec.com/docs) before using the Operator. For advance configuraions please consult with Aqua's support team.
    
For a simple Aqua configuration please follow the following guidelines -
1. Install the Aqua Opertor 
2. Manage all the pre-requists as covered in the insutrcstion below (see "Before You Begin Using the Operator CRDs")
3. Use the AquaCSP Custom Resource to install Aqua in your Cluster. AquaCSP will automaticlly deploy the Console, Database, Scanner, and Gateway CR
4. To install the Enforcer CR, you will need to access Aqua and create a new Enforcer Group. Copy the group's 'token' as you will need it later for the Enfrocer Custom Resource
5. Use the AquaEnforcer Custom Resource to install the Aqua Enforcer. Replace the value of the 'token' property with the token you kept from step 3
	

## Before You Begin Using the Operator CRDs
Install the Aqua Operator and obtain access to the Aqua registry - https://www.aquasec.com/about-us/contact-us/
You will need to supply two secrets during the installation - 
* A secret for the Docker registry
* A secret for the database

You can  list the secrets in the YAML files or you can define secrets in the OpenShift project (see example below) -
```bash
oc create secret docker-registry aqua-registry --docker-server=registry.aquasec.com --docker-username=<AQUA_USERNAME> --docker-password=<AQUA_PASSWORD> --docker-email=<user email> -n aqua
oc create secret generic aqua-database-password --from-literal=db-password=<password> -n aqua
oc secrets add aqua-sa aqua-registry --for=pull -n aqua
```

## Installating AquaCSP
There are multiple options to deploy the Aqua  CSP. You can review the different options in the following [file](https://github.com/aquasecurity/aqua-operator/blob/master/deploy/crds/operator_v1alpha1_aquacsp_cr.yaml).  Note that for production environments we recommend connecting Aqua to an external production grade database. For lab implementations,  you can use the default database in the installation scripts.
Here is an example of a simple installation  - 
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

## Installating AquaEnforcer
You can review the different options to to implement AquaEnforcer in the following [file] (https://github.com/aquasecurity/aqua-operator/blob/master/deploy/crds/operator_v1alpha1_aquaenforcer_cr.yaml)
Here is an example of a simple installation  - 
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
