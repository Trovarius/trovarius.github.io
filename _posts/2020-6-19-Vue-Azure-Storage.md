---
layout: post
title: Static Vue app in Azure Storage
---

# Deploy VUE app in Azure Storage

Ever now and then we have to build and deploy a simple SPA web site, this often have to be available for others. 

Even thought we have heaps of solutions out there today I like to share how to do it using Azure Storage and later on automate its deployment using integrated Circle CI.

LETS START!!

## Prerequisite

* [Azure account](azure.microsoft.com)
* [Azure CLI](https://docs.microsoft.com/pt-pt/cli/azure/install-azure-cli?view=azure-cli-latest)
* [Vue installed](https://cli.vuejs.org/guide/installation.html)
* [CircleCI account](https://circleci.com/integrations/github/)


## Vue App

Once you have everything installed you should create your Vue app just by typing

```sh
$ vue create <your-app-name>
$ cd ./<your-app-name>
```

Inside this folder you will find your app, and from here we have to execute the command below. This command will create a folder called `./dist` where you will find all files needed to be deployed in Azure Store.

```sh
$ npm run build
```

## Azure Storage

We going to publish or app in the Azure Storage so we need to first configure it to accept a static web page, instead of showing you how I will suggest you follow (this link)[https://docs.microsoft.com/pt-pt/azure/storage/blobs/storage-blob-static-website-host] made by Microsoft so if it changes in the future you will be able to have any additional information. 

## Publishing

Now we have our static app and our host configured we just need to plug them both but how to do it?
Basically we only need to login into your Azure account and your `./dist` folder into your already reacted Azure Store for static site.

```sh
$ az login // this command will redirect you to Azure Login page to enter your credentials
$ az storage blob upload-batch -s dist -d \\$web
```

This command will publish your `dist` folder inside the `$web` folder into Azure Storage, to confirm its everything working access the `Static WebSite` URL provided in the creation of Azure Storage and check if current website is working.

## Azure Service Principal

Now you have your app working and can continue developing it and publishing it, however a good practice is to not use your login due to security reasons. Azure Service Principal came to help in this matter.

Azure Service Principal is going to work as a identity to publish our app without have to login with our account and just like the Azure Storage you can find how to create a ASP (Azure Service Principal) for your application (here)[https://docs.microsoft.com/pt-pt/cli/azure/create-an-azure-service-principal-azure-cli?view=azure-cli-latest#sign-in-using-a-service-principal].

In  the link above you will find 3 ways to create the ASP for your account, in order to simplicity I choose to go for the third option. 

```sh
$ az ad sp create-for-rbac --name ServicePrincipalName --create-cert

# OUTPUT
#Creating a role assignment under the scope of "/subscriptions/myId"
#Please copy C:\myPath\myNewFile.pem to a safe place.
#When you run 'az login', provide the file path in the --password argument
# {
#  "appId": "myAppId",
#  "displayName": "myDisplayName",
#  "fileWithCertAndPrivateKey": "C:\\myPath\\myNewFile.pem",
#  "name": "http://myName",
#  "password": null,
#  "tenant": "myTenantId"
# }
```

Nice and good we now have a `.pem` file ( references [here](https://en.wikipedia.org/wiki/Privacy-Enhanced_Mail) and [here](https://serverfault.com/questions/9708/what-is-a-pem-file-and-how-does-it-differ-from-other-openssl-generated-key-file) ) and we going to use it to login in our Azure Account using the information we got with de above command.

```sh
$  az login --service-principal -u http://myName -p C:\\myPath\\myNewFile.pem --tenant myTenantId 
```

Once we are logged we can execute the publish command again, but this time using de account name.

```sh
$ az storage blob upload-batch -s dist -d \$web --account-name <Account Name>
```

That's it... see you next time.
