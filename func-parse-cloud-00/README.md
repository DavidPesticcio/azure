
# Timer trigger for Python Azure Function (part 1)

![](/images/logos/function.png)

## Introduction

Hosting a software application on the internet typically requires provisioning and managing a virtual or physical server and managing an operating system and web server hosting processes. Microsoft Azure provides several different ways to host and execute code or workflows without using Virtual Machines (VMs) including Azure Functions, Microsoft Power Automate, Azure Logic Apps, and Azure WebJobs. 

![](/images/func-az-ip/az_time_func.png)

This document **gives an example of using Azure Python Function** for obtaining Azure Datacenter and Office 365 IP addresses. Function execution will be started by schedule and obtained data will be stored in an Azure storage account.

## Theoretical Part

### Azure Functions

Serverless is an approach to computing that offloads responsibility for common infrastructure management tasks (e.g., scaling, scheduling, patching, provisioning, etc.) to cloud providers and tools, allowing engineers to focus their time and effort on the business logic specific to their applications or process.

Azure Functions is a serverless application platform. It allows developers to host business logic that can be executed without provisioning infrastructure. Functions provides intrinsic scalability and you are charged only for the resources used. You can write your function code in the language of your choice, including C#, F#, JavaScript, Python, and PowerShell Core. 

In Azure Functions, specific functions share a few core technical concepts and components, regardless of the language or binding you use.

![](/images/func-az-ip/function_separation.png)

A function is the primary concept in Azure Functions. When you are working with a function in Azure, you must choose following configurations:
* Functions hosting plan
* Functions runtime
* Functions trigger
* Functions language

#### Hosting plans
There are three hosting plans available for Azure Functions: Consumption plan, Premium plan, and Dedicated (App Service) plan.

* **Consumption** - You're only charged for the time that your function app runs. This plan includes a free grant on a per subscription basis.
* **Premium** - Provides you with the same features and scaling mechanism as the Consumption plan, but with enhanced performance and VNET access. Cost is based on your chosen pricing tier. To learn more, see Azure Functions Premium plan.
* **Dedicated (App Service)** -	When you need to run in dedicated VMs or in isolation, use custom images, or want to use your excess App Service plan capacity. Uses regular App Service plan billing. Cost is based on your chosen pricing tier.

Both Consumption and Premium plans automatically add compute power when your code is running. Your app is scaled out when needed to handle load, and scaled in when code stops running. For the Consumption plan, you also don't have to pay for idle VMs or reserve capacity in advance.

On any plan, a function app requires a general Azure Storage account, which supports Azure Blob, Queue, Files, and Table storage. This is because Functions relies on Azure Storage for operations such as managing triggers and logging function executions, but some storage accounts do not support queues and tables. These accounts, which include blob-only storage accounts (including premium storage) and general-purpose storage accounts with zone-redundant storage replication, are filtered-out from your existing Storage Account selections when you create a function app.

More detailed information about hosting plans available [here](https://docs.microsoft.com/en-us/azure/azure-functions/functions-scale).

#### Trigger events

Triggers are what cause a function to run. A trigger defines how a function is invoked and a function must have exactly one trigger. Triggers have associated data, which is often provided as the payload of the function.

Binding to a function is a way of declaratively connecting another resource to the function; bindings may be connected as input bindings, output bindings, or both. Data from bindings is provided to the function as parameters.

You can mix and match different bindings to suit your needs. Bindings are optional and a function might have one or multiple input and/or output bindings.

Azure Functions support a wide range of trigger types: 
* **Timer trigger** - execute a function at a set interval.
* **HTTP trigger** - execute a function when an HTTP request is received.
* **Blob storage trigger** - execute a function when a file is uploaded or updated in Azure Blob storage.
* **Queue storage trigger** - execute a function when a message is added to an Azure Storage queue.
* **Azure Cosmos DB trigger** - execute a function when a document changes in a collection.
* **Event Grid trigger** - respond to Azure Event Grid events via subscriptions and filters
* **Event Hub trigger** - execute a function when an event hub receives a new event.
* **Service Bus Queue/Topic trigger** - execute a function that react to and send queue/topic messages

#### Runtime
A function app runs on a specific version of the Azure Functions runtime. There are three major versions: 1.x, 2.x, and 3.x. By default, function apps are created in version 2.x of the runtime. 

The major versions of the Azure Functions runtime are related to the version of .NET on which the runtime is based. Runtime versions 3.x and 2.x are running on .NET Core, whereas 1.x is using .NET Framework .

More detailed information about functions runtime you can find [here](https://docs.microsoft.com/en-us/azure/azure-functions/functions-versions).

#### Language

Depending on which runtime version is used, different languages are supported. All runtimes support C#, JavaScript, F#. Java, Powershell, Python and TypeScript works on 2.x and 3.x versions. 

More detailed information about supported languages is described [here](https://docs.microsoft.com/en-us/azure/azure-functions/supported-languages#languages-by-runtime-version)

### Microsoft Azure and Office 365 IP Ranges 
The Office 365 and Azure Cloud IP Addresses helps you better identify and differentiate Office 365/Azure network traffic, making it easier for you to evaluate, configure, and stay up to date with changes. 

For the data on the Office 365 IP address use you have https://endpoints.office.com/endpoints/worldwide endpoint.

For obtaingin Azure Datacenter IP ranges you have to use https://azuredcip.azurewebsites.net/getazuredcipranges via an API. 

## Prerequisites

Before you begin the next section, you’ll need:
* [Azure Cloud account](https://azure.microsoft.com/free/)
* Local environment with installed: [Python](https://www.python.org/downloads/) (version 3.6 or higher), [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest), [Azure Functions Core Tools](https://github.com/Azure/azure-functions-core-tools#versionss) (newest available latest version) and [Git client](https://git-scm.com/downloads)

## Practical Part
To run this demo you'll need to:
1. Create Azure environment using ARM [template](https://raw.githubusercontent.com/groovy-sky/azure/master/func-parse-cloud-00/azuredeploy.json), which creates Linux consumption plan, Python application service, storage account and Application Insights.  
2. Publish [the demo function](https://github.com/groovy-sky/azure-office-ip) to Azure App Service. Its structure looks following:

![](/images/func-az-ip/func_structure_folder.png)


### ARM template deployment
There are a few ways to deploy [the demo ARM template]((https://raw.githubusercontent.com/groovy-sky/azure/master/func-parse-cloud-00/azuredeploy.json)). Easiest way how you can do so, by clicking to the button below:

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fgroovy-sky%2Fazure%2Fmaster%2Ffunc-parse-cloud-00%2Fazuredeploy.json" target="_blank"> <img src="https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/1-CONTRIBUTION-GUIDE/images/deploytoazure.png"/> </a> 

Provide required data and start the deployment:

![](/images/func-az-ip/az_func_deploy.png)

Template create 4 resources with unique - App Service, Application Insights, App Service plan and Storage account. **In the next section you'll need to use the App Service name for function publishing**.

### Function publishing

The Azure Functions Core Tools let you to run functions on your local computer or publish to Azure. When you publish a function using Core Tools - it don't ask you to sign in to Azure. Instead, they access your subscriptions and resources by loading your session information from the Azure CLI. If you don't have an active session in one of those tools, publishing will fail.

Which is why **before publishing you need to login to Azure CLI** and only then execute code below (**also don't forget to replace ```<function_name>``` with your App Service name**):

```
[ ! -d "iaac-demo/.git" ] && git clone https://github.com/groovy-sky/azure-office-ip
cd azure-office-ip && git pull
func azure functionapp publish <function_name> --python
```

The whole thing takes less than 5 minutes: 

![](/images/func-az-ip/func_deploy.gif)


## Results

This demos timer trigger has a schedule, which is set to run each 3 hours. If you don't want to wait 3 hours you can manually start the code execution:

![](/images/func-az-ip/az_func_manual_run.png)


By default, after function was published and triggered, generated data will be accessible through the following link - `https://<storage-name>.blob.core.windows.net/$web/main.html` (where `<storage-name>` is your Storage account name). Optionally, you can get a better URL (like `https://<storage-name>.z6.web.core.windows.net`) by enabling "Static website" feature:
![](/images/func-az-ip/az_func_static_website.png)

If the publishing and running was successful you can validate a result by accessing web page/storage account:

![](/images/func-az-ip/az_trigger_func_res.png)

## Summary

At this point you've deployed Azure environment, published and executed Function. 

The main drawback of this approach is the absence of a proper monitoring. In the next chapter we will try to find the way how to make things better using an get a better visibility.


## Related Information

* https://www.ibm.com/cloud/learn/microservices

* https://www.ibm.com/cloud/learn/serverless

* https://docs.microsoft.com/en-us/Office365/Enterprise/office-365-ip-web-service

* https://endpoints.office.com/endpoints/worldwide?clientrequestid=b10c5ed1-bad1-445f-b386-b919946339a7

* https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/storage

* https://docs.microsoft.com/en-us/azure/storage/blobs/storage-blob-static-website

* https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-blob/azure/storage/blob/_blob_client.py

* https://github.com/Azure/Azure-Functions/wiki/Bring-your-own-storage-(Linux-consumption)

* https://www.serverlesslibrary.net/?technology=Functions%202.x&language=Python

* https://github.com/sendgrid/sendgrid-python#without-mail-helper-class

* https://docs.microsoft.com/en-us/azure/azure-functions/functions-reference-python#folder-structure