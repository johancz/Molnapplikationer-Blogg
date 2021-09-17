---
layout: post
title:  "Exercise 4 - Serverless, FaaS and Azure Functions"
date:   2021-09-17 14:21
tags: serverless, FaaS, Azure, Azure Functions, GitHub Actions 
---

<h1 style="color:red">Alt. text on images!</h1>
<h1 style="color:red">Alt. text on images!</h1>
<h1 style="color:red">Alt. text on images!</h1>


## What is Serverless and Function As A Service (FaaS)?

### Serverless

Let's preface this by stating that serverless doesn't actually mean that there are no servers.

Serverless enables developers to focus on the code and content of their app, without having to worry about everything related to running, maintaining and supporting the servers which run their app. This is up to the provider of the serverless service, who make sure that the servers run smoothly and securely. They take care of scaling your app to multiple servers for you based on demand, and they make sure that your app is resilient. Did a server go down? No problem, the app is spooled up on a different server.

Serverless is an event-driven model, meaning the developer select what should trigger their app, such as an HTTP request or a predefined schedule.

There are multiple benefits to serverless computing, some of the most talked about are cost efficiency and scaling. Serverless employs a pay-as-you-go business model, where the customers only pays when the app is used and for the resources the app uses (memory, compute time and number of requests). There are no costs associated with installing, maintaining or supporting hardware and the customer does not have to employ a person to do all of these things. Nor does the customer have to worry about the servers' operating system licenses.

Serverless apps have great scalability, with no setup required. Apps can simply deploy to more servers as demand increases, or scale down as demand subsides. It just works. Developers can of course configure some things, such as setting a limit on how many servers the app can scale to.

Serverless apps are also easy to deploy, and deployment is fast. There is nothing to manage, just deploy the app.

But there are also some potential downsides to serverless. Because serverless apps only really run on demand, there is the possibility of an initial performance hit when functions that are used less frequently are called, since the app may need a cold start. Serverless apps can also be difficult to debug and test, with the limit or non-existent visibility into the backend processes and how they run, this can complicate these matters. And if an apps is divided into multiple functions, this can further complicate matters. Another disadvantage with serverless computing, and this can be true for most cloud computing, is security and privacy. How does the service provider handle security? What policies do they employ? Can you trust that your sensitive data is safe and secure? This can be difficult to gauge with the limited insight into how the servers are configured.

Also, apps that have processes with long life-times might be not be suitable for serverless since the costs can quickly go up.


### FaaS (Function as a Service)

FaaS is an event driven execution model where functions/logic are deployed to stateless containers which can be executed on demand. FaaS is suitable for short lived processes.


<!-- -->

## My Azure Functions mini calculator

### An overview of the code:

### How to get your project to run in Azure Functions

Begin by creating a new Azure Functions project with Visual Studio.

![Testing of Azure Functions app on Azure portal - steps and input](/Molnapplikationer-Blogg/data/images/exercise-4-serverless-faas-azure-functions/Visual-Studio-new-Azure-Functions-project.png)

After selecting a name and location for the project on your PC, configured your function to run on ".NET Core 3 (LTS)" and to have a "Http trigger". Leave "Storage Account" and "Authorization level" on their default values, "Storage Emulator" and "Function" respectively.

![](/Molnapplikationer-Blogg/data/images/exercise-4-serverless-faas-azure-functions/Visual-Studio-new-Azure-Functions-project-setup-configuration.png)

Click "Create" and wait for Visual Studio to set things up for you. 

Once everything is setup, the code for your new function ("Function1") should open, and you may want to go ahead at this stage and rename it:

![](/Molnapplikationer-Blogg/data/images/exercise-4-serverless-faas-azure-functions/Visual-Studio-new-Azure-Functions-project-rename-function.png)

Start your app with debugging to make sure it runs locally. A console window will open and you should see something like this:

![Azure Functions project - Run - output in console](/Molnapplikationer-Blogg/data/images/exercise-4-serverless-faas-azure-functions/Visual-Studio-new-Azure-Functions-project-debugrun-console-output.png)

Open the url you'll find in the console output, something along the lines of: `http://localhost:port/api/YourFunction`, in a browser.
And maybe pass it a query to test everything.

![Azure Functions project - Run - output in browser](/Molnapplikationer-Blogg/data/images/exercise-4-serverless-faas-azure-functions/Visual-Studio-new-Azure-Functions-project-debugtest-browser-output.png)


Now it's up to you to code your app.


#### Publish to Azure Functions from Visual Studio.

When you have developed something you want to publish to Azure Functions. Right click your project in the Project Explorer and click "Publish".

![Click "Publish" in the Solution Explorer](/Molnapplikationer-Blogg/data/images/exercise-4-serverless-faas-azure-functions/Visual-Studio-new-Azure-Functions-project-solution-explorer-publish.png)

In the wizard that runs, select "Azure" your "Target" and "Azure Function App (Windows)" as the "Specific target".
Next under "Functions instance":
1. First make sure you're using the correct Microsoft Account (see the image).
2. Select the subscription you want to use and choose "Resource Group" under "View".
3. Select a resource group and function app under "Function Apps" or click the "+" button to create a new app.
4. Verify that you're using the correct Microsoft Account (see the image).
5. Enter a unique name for your app.
6. Select an existing resource group or click "New..." to create one.
7. Set "Plan Type" to "Consumption" and select a server near you under "Location".
8. Use the default "Azure Storage" or create a new "Storage" (I won't go into how you can create new storage).
9. Click "Create". You may want to go drink some water at this point, it will a minute or so for it create your app and resource group.
10. Back in the wizard, leave "Run from package file (recommended)" checked and click "Finish".

![Configure your Azure Functions app before publishing](/Molnapplikationer-Blogg/data/images/exercise-4-serverless-faas-azure-functions/Visual-Studio-new-Azure-Functions-project-solution-explorer-publish-wizard-step-3.png)

You should now see this:

![Configure your Azure Functions app before publishing](/Molnapplikationer-Blogg/data/images/exercise-4-serverless-faas-azure-functions/Visual-Studio-new-Azure-Functions-project-solution-explorer-publish-ready-to-publish.png)

Click "Publish" and if everything goes as planned you get a message saying the app was published successfully.

![Configure your Azure Functions app before publishing](/Molnapplikationer-Blogg/data/images/exercise-4-serverless-faas-azure-functions/Visual-Studio-new-Azure-Functions-project-solution-explorer-publish-published.png)

Congratulation, you have published your app to Azure Functions. 🍰


#### Deploy to Azure Functions with a GitHub Actions pipeline

> First create a github repository and push your code if you don't already have a repository.

What you want to do now is set up some authentication to be able to deploy from a Github Action.

Begin by navigating to your Azure Functions app on the [Azure Portal][link-azure-portal].

Click "Get publish profile" (as in in the image below) and save it to your computer.

![Azure Function publish profile](/Molnapplikationer-Blogg/data/images/exercise-4-serverless-faas-azure-functions/Azure-Porta-Function-get-publish-profile.png)

<div class="alert alert-info" role="alert"><i class="fa fa-info-circle"></i> <b>Warning:</b> This file contains sensitive data, so make sure you don't publish it or push it to your git repository.</div>


What you want to do next is to copy the contents of the "publish profile" and save it as a GitHub secret.
To do this:
1. Navigate to your GitHub repository.
2. Click "Setting".
3. Click "Secrets".
4. In the "Name" field enter: `AZURE_FUNCTIONAPP_PUBLISH_PROFILE`
5. Copy the contents of the "publish profile" file you downloaded and paste it into the "Value" box.
6. Click "Add secret".

You can now use this secret in your pipeline to authenticate against Azure when deploying your app.

![Azure Function publish profile](/Molnapplikationer-Blogg/data/images/exercise-4-serverless-faas-azure-functions/github-repository-secret-new.png)


To publish to Azure Functions with Github Actions you can use [this action on the Github Marketplace][github-marketplace-azure-functions-action]

Start by copying the template to a new workflow file, e.g. `./.github/workflows/`.

Let's make changes to the template:

![Azure Functions action template](/Molnapplikationer-Blogg/data/images/exercise-4-serverless-faas-azure-functions/Azure-Functions-action-template-editthese.png)

1. (Optional) Change the name of the action.
2. To make sure you only publish your app when you push to your master branch, change the "On" variable from this:
```yaml
on:
  [push]
```
to this:
```yaml
on:
    push:
        branches: [ main ]
```
3. Change the value of `AZURE_FUNCTIONAPP_NAME` to the name of your Azure Function.
4. Change the value of `AZURE_FUNCTIONAPP_PACKAGE_PATH` to the path to the directory of your source code.
5. Change the value of `DOTNET_VERSION` to the dotnet version you are using in your project.
6. Push your new workflow file to your master branch.


### How did I test the application?

I have only tested my function with the Rest Client extension for Visual Studio Code and with the "Test/Run" functionality on the Azure Portal.


#### Azure Portal

To test my function on the Azure Portal I followed these steps.
0. Navigate to your function.
1. Click "Code + Test" on the right.
2. Click the "Test/Run" button in the toolbar above the code editor/viewer.
3. Click the "Input" tab to the right of the code editor/viewer.
4. Add query parameters by clicker "+Add parameters" or add data into the body.
5. Hit "Run" at the bottom.

![Testing of Azure Functions app on Azure portal - steps and input](/Molnapplikationer-Blogg/data/images/exercise-4-serverless-faas-azure-functions/Azure-dotcom-function-management-testrun-input.png)

You should now see the output:

![Testing of Azure Functions app on Azure portal - steps and input](/Molnapplikationer-Blogg/data/images/exercise-4-serverless-faas-azure-functions/Azure-dotcom-function-management-testrun-output.png)


#### Testing with REST Client for Visual Studio Code

By intalling the [REST Client][vs-code-rest-client-extension] extension for Visual Studio Code I can make API calls to test my Azure Functions app like so:

![Testing of Azure Functions app with REST Client for Visual Studio Code](/Molnapplikationer-Blogg/data/images/exercise-4-serverless-faas-azure-functions/Visual-Studio-Code-Rest-Client-testing.png)


### Vilka säkerhets hot finns där till en applikation om din (beskriv minst en)?

There really aren't that many security threats for this simple mini calculator, I'm not using a database, nor am I in any way dealing with sensitive data. I'm not dealing with authentication of users, beyond requiring a function key to 

#### Och har du gjort något för att säkra dig emot dissa? (hint: OWASP top 10 - Interpretation for Serverless)


<!-- -->


## Sources & Links
- [Azure Functions Action - Github Marketplace][github-marketplace-azure-functions-action]
- [Azure Functions Action template - Github Marketplace][github-marketplace-azure-functions-action-dotnet-template]
- [REST Client for Visual Studio Code][vs-code-rest-client-extension]
- [Azure Portal][link-azure-portal]


[github-marketplace-azure-functions-action]: https://github.com/marketplace/actions/azure-functions-action
[github-marketplace-azure-functions-action-dotnet-template]: https://github.com/Azure/actions-workflow-samples/blob/master/FunctionApp/windows-dotnet-functionapp-on-azure.yml
[vs-code-rest-client-extension]: https://marketplace.visualstudio.com/items?itemName=humao.rest-client
[link-azure-portal]: https://portal.azure.com/#home