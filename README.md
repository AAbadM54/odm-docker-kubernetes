# IBM-ODM-Kubernetes
IBM Operational Decision Manager on Kubernetes

[![Build Status](https://travis-ci.org/PierreFeillet/IBM-ODM-Kubernetes.svg?branch=master)](https://travis-ci.org/PierreFeillet/IBM-ODM-Kubernetes)


#  Deploy IBM Operational Decision Manager Standard on a Kubernetes Cluster

This repository centralizes the material to deploy IBM Operational Decision Manager Standard in Kubernetes. clustered topology using WAS Liberty on a Kubernetes Cluster.

[IBM ODM](https://www.ibm.com/support/knowledgecenter/SSQP76_8.9.0/welcome/kc_welcome_odmV.html) is a decisioning platform to automate your business policies. Business rules are used at the heart of the platform to implement decision logic on a business vocabulary and run it as web decision services.

We leverage the ODM Docker material put available on this repository [odm-ondocker](https://github.com/lgrateau/odm-ondocker). It includes Docker files and Docker compose descriptors. Docker files are used to build images of ODM runtimes. And Docker compose desctiptor can be used to group this build and push to your repository for a Kubernetes provisioning. Nevertheless Docker Compose and Kubernetes are 2 distinct technology paths to provision an contain based topology supported by ODM Standard.

![Flow](images/ODMinKubernetes-DeploymentOverview.png)

## Deploying ODM Rules in the following clouds
- [Bluemix Kubernetes Cluster](Bluemix/README.md)
- [MiniKube](MiniKube/README.md)
 
## References

# License
[Apache 2.0](LICENSE)

