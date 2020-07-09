# Deploying IBM Operational Decision Manager Standard on Red Hat OpenShift on IBM Cloud with App ID Authentication

This will explain you how to deployed a topology 

## Included Components
- [IBM Operational Decision Manager](https://www.ibm.com/support/knowledgecenter/en/SSQP76_8.10.x/com.ibm.odm.kube/kc_welcome_odm_kube.html)
- [IBM Cloud AppID (App ID)](https://cloud.ibm.com/docs/appid)


## Tutorial environment
The commands and tools was tested on MacOS.

## Prerequisites
Install this pre-requisite in your machine.
* [IBM Cloud  Cli](https://cloud.ibm.com/docs/cli)
* [Helm](https://github.com/helm/helm/releases)
* [Kubectl](https://kubernetes.io/fr/docs/tasks/tools/install-kubectl/)

Create an  [IBM Cloud Account](https://cloud.ibm.com/)

## Steps to deploy ODM on Kubernetes from AWS EKS

- [Deploying IBM Operational Decision Manager Standard on Red Hat OpenShift on IBM Cloud with App ID Authentication](#deploying-ibm-operational-decision-manager-standard-on-red-hat-openshift-on-ibm-cloud-with-app-id-authentication)
  - [Included Components](#included-components)
  - [Tutorial environment](#tutorial-environment)
  - [Prerequisites](#prerequisites)
  - [Steps to deploy ODM on Kubernetes from AWS EKS](#steps-to-deploy-odm-on-kubernetes-from-aws-eks)
  - [1. Preparing your environment (40 min)](#1-preparing-yourenvironment-40-min)
    - [Create a  Red Hat OpenShift on IBM Cloud :  (20 min)](#create-a-red-hat-openshift-on-ibm-cloud--20-min)
    - [Setup your local environment (10 min)](#setup-your-local-environment-10-min)
  - [2. Preparing App ID services (20 min)](#2-preparing-app-id-services-20-min)
    - [Create an App ID Sercice instance](#create-an-app-id-sercice-instance)
    - [Add an appliction](#add-an-appliction)
    - [Add a user](#add-a-user)
  - [3. Preparing Operational Decision Manager for AppID (20 min)](#3-preparing-operational-decision-manager-for-appid-20-min)
    - [Create the user mapping file.](#create-the-user-mapping-file)
    - [Create the OpenID ODM configuration file](#create-the-openid-odm-configuration-file)
    - [Create the ODM OpenID secrets](#create-the-odm-openid-secrets)
    - [Get the certificate and create ODM secret certificate.](#get-the-certificate-and-create-odm-secret-certificate)
  - [4. Preparing ODM for deployment (40 min)](#4-preparing-odm-for-deployment-40-min)
      - [Create a pull secret to pull images from the IBM Entitled Registry](#create-a-pull-secret-to-pull-images-from-the-ibm-entitled-registry)
  - [5 Deploy the ODM topology (10 min)](#5-deploy-the-odm-topology-10-min)
  - [6 Add Redirect Web URL (5 min)](#6-add-redirect-web-url-5-min)
  - [7 Play with ODM](#7-play-with-odm)

## 1. Preparing your environment (40 min)
### Create a  Red Hat OpenShift on IBM Cloud :  (20 min)
          see the IBM Cloud documentation https://cloud.ibm.com/docs/openshift?topic=openshift-getting-started to setup a cluster. 

### Setup your local environment (10 min)
   - see the IBM Cloud documentation  [Installing the IBM Cloud and OpenShift CLI](https://cloud.ibm.com/docs/openshift?topic=openshift-openshift-cli) to setup your environment to use IBM Cloud Kubernetes instance.
   
     After reading you should have installed:
      - The kubernetes-service plugin. 
         
         ```ibmcloud plugin install kubernetes-service ```
      - The helm (3.2 or up) cmd line
      - The oc cmd line
      - The kubectl cmd line
 
   - Then you shoud follwo this [Connect to the cluster from the CLI](https://cloud.ibm.com/docs/openshift?topic=openshift-access_cluster#access_oc_cli) documentation to be able to connect to the custer.
   - Check the connection with the cluster. Try to list the cluster projects by this command.
        
        ```oc get projects```

## 2. Preparing App ID services (20 min)
### Create an App ID Sercice instance

  -  Follow this steps to [Create an App ID service instance](https://cloud.ibm.com/docs/appid?topic=appid-getting-started#create) .

### Add an appliction
  - Go to the Application Tab.
  - Click Add an application button.
  - Enter a name to your application. 
  - Expand the application item.
  - Click save button
  -  Take a note of the following informations:
     -  clientId: Will be reference as `<ClientID>` in the next steps.
     -  secret: Will be reference as `<ClientSecret>` in the next steps.
     -  oAuthServerUrl: Will be reference as `<oAuthServerUrl>` in the next steps.
  
### Add a user 
  - Follow this documentation to [Add a user](https://cloud.ibm.com/docs/appid?topic=appid-cd-users#add-users) in the AppID service Cloud Provider registry.

  - Take a note of the Predefined attributes "id". This can be found when you click in the user you have previous created.
  - Click "Copy Id" button or this can be found in the summary. 
    - This will be referenced as `<UserAppID>` in the next steps.

>Something like that : 
>``` "id": "e5cbaa8e-3e07-4038-b785-35f1aa5a4841",```
  

## 3. Preparing Operational Decision Manager for AppID (20 min)
### Create the user mapping file.
1. Download the [webSecurityTpl.xml](webSecurityTpl.xml) file in your local machine.
2. Copy it to webSecurity.xml
3. Edit the file.
4. Replace the placeholder <USER_ID> by the concatenation of  `<oAuthServerUrl>/<UserAppID>`

> `<oAuthServerUrl>` was found in the step Add an application
> `<UserAppID>` was found in the step Add a user 
> Ex : https://us-south.appid.cloud.ibm.com/oauth/v4/db028209-00b5-47e9-9b36-151a107d515d/e5cbaa8e-3e07-4038-b785-35f1aa5a4841

### Create the OpenID ODM configuration file
1. Download the [openIdWebSecurityTpl.xml](openIdWebSecurityTpl.xml) file in your local machine.
2. Copy it to openIdWebSecurity.xml
3. Edit the file.
4. Replace the placeholder:
   -  `<oAuthServerUrl>` by the value `<oAuthServerUrl>`
   -  `<clientId>` by the value `<ClientID>`
   -  `<oAuthServerUrl>` by the value `<ClientSecret>`

### Create the ODM OpenID secrets
Create the secret by this following command line.

 ```kubectl create secret generic mywebsecuritysecret --from-file=webSecurity.xml --from-file=openIdWebSecurity.xml=openIdWebSecurity.xml```
    
 Then 
 
 ```kubectl create secret generic my-openid-admin-secret  --from-literal=adminUser=<ClientID>  --from-literal=adminPassword=<ClientSecret>```

### Get the certificate and create ODM secret certificate.
  - Go to the AppID web interface in the instance you have previously created.
  - Download the Java samples app
    - In your browser go to the AppId service created previously.
    - Click [Download sample] Button
    - Select Java 
    - Download and unzip it in your local machine.
    - Go to the directory app-id-sample-java/Liberty where there is the mytrustore.jks file.
  - Create a secret that contains the trustory.jks require to connect to AppID by using this following cmd.
  
    ```oc create secret generic mysecuritysecret  --from-literal=keystore_password=Liberty --from-file=keystore.jks=mytruststore.jks  --from-literal=truststore_password=Liberty --from-file=truststore.jks=mytruststore.jks```

> You can manage your own certificate. Is that case you should add the mytruststore.jks in your certificate and create this secret.

## 4. Preparing ODM for deployment (40 min)
#### Create a pull secret to pull images from the IBM Entitled Registry
1. Log in to [MyIBM Container Software Library](https://myibm.ibm.com/products-services/containerlibrary) with the IBMid and password that are associated with the entitled software.

2. In the **Container software library** tile, verify your entitlement on the **View library** page, and then go to **Get entitlement key** to retrieve the key.

3. Create a pull secret by running a `kubectl create secret` command.
   ```oc create secret docker-registry admin.registrykey --docker-server=cp.icr.io --docker-username=cp --docker-password="<API_KEY_GENERATED>" --docker-email=<USER_EMAIL>```

   > **Note**: The `cp.icr.io` value for the **docker-server** parameter is the only registry domain name that contains the images.
   
   > **Note**: Use “cp” for the docker-username. The docker-email has to be a valid email address (associated to your IBM ID). Make sure you are copying the Entitlement Key in the docker-password field within double-quotes.

4. Take a note of the secret and the server values so that you can set them to the **pullSecrets** and **repository** parameters when you run the operator for your containers.


## 5 Deploy the ODM topology (10 min)

Assuming you have the helm chart in your local machine retrived from the Entitled registry or from a PPA.

- Create a Projects: 

  ```oc new-project odm-ibmcloud```

- Give permission

  ```oc adm policy add-scc-to-user privileged -z default```

- Deploy the topology:

  ```helm install odm-8104 --set image.tag=8.10.4.0 --set image.repository=cp.icr.io/cp/cp4a/odm --set image.pullSecrets=admin.registrykey --set internalDatabase.populateSampleData=true --set internalDatabase.persistence.enabled=false --set service.enableRoute=true --set image.arch=amd64 --set customization.authSecretRef=mywebsecuritysecret --set customization.securitySecretRef=mysecuritysecret --set oidc.enabled=true --set oidc.adminRef=my-openid-admin-secret --set oidc.allowedDomains=us-south.appid.cloud.ibm.com ibm-odm-prod-2.3.0.tgz```

Where:
- ibm-odm-prod : Is the location of the ODM charts

## 6 Add Redirect Web URL (5 min)

You should declare the redirect url of your application to the AppID service. Todo that : 

- Retrieve the routes to acces to your instance:
```oc get routes```
- Go to the AppID instance you have previously created
- Click Manage Authentication tab
- Click Authentication Setting tab
- Click + for all the routes declare https://`<HOSTS>`/oidcclient/redirect/odm
  
```Exemple:
$ oc get routes
 NAME                            HOST/PORT                                                                                                                     PATH   SERVICES                             PORT                          TERMINATION   WILDCARD
odm-8104-odm-dc-route           odm-8104-odm-dc-route-odm-rob.dbacluster04-4245ee08d404afbcaa8f0c6b522e175c-0001.us-east.containers.appdomain.cloud                  odm-8104-odm-decisioncenter          decisioncenter-https          passthrough   None
odm-8104-odm-dr-route           odm-8104-odm-dr-route-odm-rob.dbacluster04-4245ee08d404afbcaa8f0c6b522e175c-0001.us-east.containers.appdomain.cloud                  odm-8104-odm-decisionrunner          decisionrunner-https          passthrough   None
odm-8104-odm-ds-console-route   odm-8104-odm-ds-console-route-odm-rob.dbacluster04-4245ee08d404afbcaa8f0c6b522e175c-0001.us-east.containers.appdomain.cloud          odm-8104-odm-decisionserverconsole   decisionserverconsole-https   passthrough   None
odm-8104-odm-ds-runtime-route   odm-8104-odm-ds-runtime-route-odm-rob.dbacluster04-4245ee08d404afbcaa8f0c6b522e175c-0001.us-east.containers.appdomain.cloud          odm-8104-odm-decisionserverruntime   decisionserverruntime-https   passthrough   None
```

In this example, the redirect url to add to the web interface will be :
- https://odm-8104-odm-dc-route-odm-rob.dbacluster04-4245ee08d404afbcaa8f0c6b522e175c-0001.us-east.containers.appdomain.cloud/oidcclient/redirect/odm
- https://odm-8104-odm-dr-route-odm-rob.dbacluster04-4245ee08d404afbcaa8f0c6b522e175c-0001.us-east.containers.appdomain.cloud/oidcclient/redirect/odm
- https://odm-8104-odm-dr-route-odm-rob.dbacluster04-4245ee08d404afbcaa8f0c6b522e175c-0001.us-east.containers.appdomain.cloud/oidcclient/redirect/odm
- https://odm-8104-odm-ds-runtime-route-odm-rob.dbacluster04-4245ee08d404afbcaa8f0c6b522e175c-0001.us-east.containers.appdomain.cloud/oidcclient/redirect/odm


## 7 Play with ODM

- Retrieve the routes to acces to your instance:
```oc get routes```

- Then open a browser : https://`<HOST OF THE COMPONENT>`
- You should be redirected to the AppID login page
- Enter your login/password
- You should be redirected to the ODM component page.


TODO:
- Add a real BD : Postgresql ibmcloud
- Manage the internal communication. (Run simulation for example)
- Add an architecture diagram
- Use PVC instead of local persistence.
