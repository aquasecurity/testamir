The Aqua Security Operator runs within an OpenShift cluster and provides a means to deploy and manage the Aqua Security cluster and components :

* Server (aka “console”)

* Database (for production environments we recommend to use an external database and not the Aqua default database)  

* Gateway 

* Enforcer (aka “agent”)

* Scanner

* CSP (package that we recommend to use in non production environments and contains the Server, Database, and Gateway)

Use the Aqua-Operator to 

* Deploy Aqua Security components on OpenShift

* Scale up Aqua Security components with extra replicas

* Assign metadata tags to Aqua Security components

* Automatically scale the number of Aqua scanners based on the number of images in the scan queue

## Before You Begin Using the Operator CRDs

Obtain access to the Aqua registry - https://www.aquasec.com/about-us/contact-us/

You will need to supply two secrets during the installation  

* A secret for the Docker registry

* A secret for the database

You can  list the secrets in the YAML files or you can define secrets in the OpenShift project (see example below) -

```bash

oc create secret docker-registry aqua-registry --docker-server=registry.aquasec.com --docker-username=<AQUA_USERNAME> --docker-password=<AQUA_PASSWORD> --docker-email=<user email> -n aqua

oc create secret generic aqua-database-password --from-literal=db-password=<password> -n aqua

oc secrets add aqua-sa aqua-registry --for=pull -n aqua

```

## After the Installation
There are multiple options to deploy the Aqua  CSP. You can review the different options in the following [file](https://github.com/aquasecurity/aqua-operator/blob/master/deploy/crds/operator_v1alpha1_aquacsp_cr.yaml).  Note that for production environments we recommend connecting Aqua to an external production grade database. For lab implementations,  you can use the default database  in the installation scripts.
Here is an example of a simple installation  - 
```yaml
--
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
--
```



