---
layout: post
title:  "Exercise 4 - Serverless, FaaS and Azure Functions"
date:   2021-09-17 23:12
tags: serverless, FaaS, Azure, Azure Functions, GitHub Actions 
---


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

```csharp
using System;
using System.IO;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Extensions.Http;
using Microsoft.AspNetCore.Http;
using Microsoft.Extensions.Logging;
using Newtonsoft.Json;
using System.Text.RegularExpressions;
using System.Linq;
using System.Data;

namespace Azure_Functions_Mini_Calculator
{
    public static class CalculateFunction
    {
        [FunctionName("CalculateFunction")]
        public static async Task<IActionResult> Run(
            [HttpTrigger(AuthorizationLevel.Function, "get", "post", Route = null)] HttpRequest req,
            ILogger log)
        {
            log.LogInformation("C# HTTP trigger function processed a request.");

            string inputString1 = req.Query["input1"];
            string inputString2 = req.Query["input2"];
            string queryToCalculate = req.Query["calculate"];

            string requestBody = await new StreamReader(req.Body).ReadToEndAsync();
            dynamic data = JsonConvert.DeserializeObject(requestBody);

            inputString1 = inputString1 ?? data?.input1;
            inputString2 = inputString2 ?? data?.input2;
            queryToCalculate = queryToCalculate ?? data?.calculate;

            string okResponseMessage = "";

            if (double.TryParse(inputString1, out double input1) && double.TryParse(inputString2, out double input2))
            {
                okResponseMessage = $"{inputString1} + {inputString2} = {input1 + input2}";
            }
            else
            {
                try
                {
                    double result = Convert.ToDouble(new DataTable().Compute(queryToCalculate, null));
                    okResponseMessage = $"{queryToCalculate} = {result}";
                }
                catch (Exception)
                {
                    return new BadRequestObjectResult("Invalid input data!");
                }
            }

            return new OkObjectResult(okResponseMessage);
        }
    }
}
```

The app is triggered by an Http request, both GET and POST requests will trigger it. The app once triggered logs the request and then fetches the values from the query for the three query parameters; `input1`, `input2`, and `calculate`. A `StreamReader` reads the body of the request which is then deserialized (JSON to a `dynamic` object).

Next it will check if our three query parameters actually have values, if they don't (i.e. they are `null`). If any of them are null, it will attempt to find the parameter in the body of the request. If the parameters is not found in either the query string or the body, the app will eventually return a "Bad Request" with the message "Invalid input data!".

Next we try to parse the strings `inputString1` and `inputString2` to `doubles`. If both of them can be parsed, we set `okResponseMessage` to the result of the two numbers added together. If at least one of the parameters cannot be parsed, we try to compute any equation found in the `calculate` parameter using the `DataTable` class and its `Compute` function and the result is then converted to a double and we set `okResponseMessage` to the result of the equation. If `DataTable().Compute()` cannot compute the equation (i.e. invalid input) it will throw and we catch the exception and return "Bad Request" with the "Invalid input data!".

If we successfully calculated a result, we return the result (stored in `okResponseMessage`) within an "Ok" result.


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
6. Push your new workflow file to your master ("main") branch and the pipeline to deploy to Azure Functions is now live.

When a commit is pushed to "main", it will attempt to build and publish it to Azure Functions. If it succeeds you'll see this.

![Azure Functions action template](/Molnapplikationer-Blogg/data/images/exercise-4-serverless-faas-azure-functions/github-actions-pipeline-success.png)


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

![Testing of Azure Functions app with REST Client for Visual Studio Code](/Molnapplikationer-Blogg/data/images/exercise-4-serverless-faas-azure-functions/Visual-Studio-Code-Rest-Client-testing-2.png)


### What potential security threats are there for my app or apps like it? And what did I do to secure my apps against them?

Serverless apps generally have more security threats than normal apps due the increased number of inputs and triggers.

There really aren't that many security threats for this simple mini calculator, I'm not using a database, nor am I in any way dealing with sensitive data. I'm not dealing with authentication of users, beyond requiring a function key to make API calls. We can however disregard this for the sake of the exercise.

If we consult [OWASP top 10 - Interpretation for Serverless][link-owasp-serverless-top-10], one potential security risk is injection. An attacker could attempt to manipulate input in such as way that they gain access to sensitive data.

To prevent injection based attack I validate the input with TryParse and only if the data is valid do I proceed to add the two numbers together. Any invalid input will result in a "Bad Request"


If my app was to have input fields on the page, XSS (Cross Side Scripting) could present further security threats.

I do rely on the use of a "publish profile" to authenticate against Azure in my GitHub Actions pipeline.
If the data in file were to be exposed it would present serious security issues. Attacks would for example have means to publish malicious content to my Functions app. To solve this I store my publish profile as GitHub repository secret.


## Sources & Links
- [Azure Functions Action - Github Marketplace][github-marketplace-azure-functions-action]
- [Azure Functions Action template - Github Marketplace][github-marketplace-azure-functions-action-dotnet-template]
- [REST Client for Visual Studio Code][vs-code-rest-client-extension]
- [Azure Portal][link-azure-portal]
- [OWASP Serverless Top 10][link-owasp-serverless-top-10]
- [Intro to Azure Functions - What they are and how to create and deploy them - IAmTimCorey (Youtube)][link-youtube-intro-to-azure-functions]
- [Understanding Serverless Computing for the Beginner - geekflare.com][geekflare-know-about-serverless]
- [Serverless computing - azure.microsoft.com][azure-microsoft-serverless-computing]


[github-marketplace-azure-functions-action]: https://github.com/marketplace/actions/azure-functions-action
[github-marketplace-azure-functions-action-dotnet-template]: https://github.com/Azure/actions-workflow-samples/blob/master/FunctionApp/windows-dotnet-functionapp-on-azure.yml
[vs-code-rest-client-extension]: https://marketplace.visualstudio.com/items?itemName=humao.rest-client
[link-azure-portal]: https://portal.azure.com/#home
[link-owasp-serverless-top-10]: https://owasp.org/www-project-serverless-top-10/
[link-youtube-intro-to-azure-functions]: https://www.youtube.com/watch?v=zIfxkub7CLY
[geekflare-know-about-serverless]: https://geekflare.com/know-about-serverless/
[azure-microsoft-serverless-computing]: https://azure.microsoft.com/en-us/overview/serverless-computing/
