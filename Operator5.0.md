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

## Prerquisits 
Make sure you have a license and access to the Aqua registry. If not please contact Aqua's sale Aqua at cloudsales@aquasec.com.


## Installing Aqua Operator
Please refer to the instuctions [here](https://github.com/aquasecurity/aqua-operator/blob/master/docs/InstallOpenShift.md)

