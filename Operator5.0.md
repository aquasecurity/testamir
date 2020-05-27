The Aqua Security Operator is used to deploy and manage the Aqua Security platform and its components:
* Server (aka “console”)
* Database (optional, you can map an external database as well) 
* Gateway 
* Enforcer (aka “agent”)
* Scanner
* CSP (a simple package that contains the Server, Database, and Gateway)

Use the Aqua-Operator to: 
* Deploy the Aqua Security platform on OpenShift
* Scale up Aqua Security components with extra replicas
* Assign metadata tags to Aqua Security components
* Automatically scale the number of Aqua scanners based on the number of images in the scan queue
	
The Aqua Operator provides a few [Custom Resources](https://github.com/aquasecurity/aqua-operator/tree/master/deploy/crds) to manage the Aqua platform. 
Please make sure to read the Aqua installation manual (https://docs.aquasec.com/docs) before using the Operator. 
   
## Prerequisites 
Make sure you have a license and access to the Aqua registry. If you want to obtain a license please contact us at https://www.aquasec.com/about-us/contact-us/.

## Installing Aqua Operator
Please refer to the instuctions [here](https://github.com/aquasecurity/aqua-operator/blob/master/docs/InstallOpenShift.md)

## Support
For support please contact support@aquasec.com
