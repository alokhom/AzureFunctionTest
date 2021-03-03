**Question 1**

Take some sample code of your own choosing for a stateless service intended to be containerized and run as an Azure function (there are a couple of sources below that you could use if you'd like to, but preferably take any custom code that allows you to quickly move over to thinking about devops).
 It isn't strictly necessary that your sample code is runnable code, but maybe it's nice to have something to refer to and as a starting point for a repo.
 
*Answer 1:*

I have taken a sample code from the python example on Azure functions site. 


**Question 2**

After you've chosen some sample code for a stateless service, 
write down a brief description of the part of the dev team's workflow that relates to CI/CD - only specifics affected by your choices.


*Answer 2:*

* I made a project in github https://github.com/alokhom/AzureFunctionTest. 
* I installed Azure CLI and Azure Functions Core Tools on vscode. 
* I did a AZ login to the azure account from the terminal
* I created a python environment locally/ activate venv/ func init a python project / add a new function (e.g. HttpExample ) with Trigger "Http trigger" with --authlevel "anonymous"
* the idea is to use a event t 
* I checked scriptFile and tested locally with "func start" to see if any URL args can trigger the function. Once fine plan for CI/CD. This would be some initial things i look forward from the developer. 
* For monitoring, I added Application Insights with samplingSettings, enableLiveMetrics, httpAutoCollectionOptions with enableHttpTriggerExtendedInfoCollection true, figuration, enableDependencyTracking, enablePerformanceCountersCollection, snapshotConfiguration. Also added managedDependency, retry with fixed delay, functionTimeout with 5 mins, enabled healthMonitor with check intervals and window, threshold. 
* I git push code to Github 
* I created a project in azuredevOps and picked up the github repo from https://github.com/alokhom/AzureFunctionTest. (as you have intended to have github as a project source) 
*  Create supporting Azure resources for your function and publish functionapp where you will publish your function. ie. make the Azure Function app service to be used by the pipeline

```az login
   az group create --name test --location northeurope
   az storage account create --name storageaccounttest9076 --location westeurope --resource-group test --sku Standard_LRS
   az functionapp create --resource-group test --consumption-plan-location westeurope --runtime python --runtime-version 3.8 --functions-version 3 --name HttpExampleFunctionApp123456 --storage-account storageaccounttest9076 --os-type linux 
   It publish app in functionapp from the folder. 
   func azure functionapp publish HttpExampleFunctionApp123456
   ```

* I made a Azure Pipeline for Python Function App to Linux on Azure:

* Invoke the function on Azure to test the app .. pass the name arg with value.
      use the URL test the app.
``` 
https://httpexamplefunctionapp123456.azurewebsites.net/api/httpexample?name=Alok 
```

**Question 3**
making CI-build/CD pipeline in Azure devops pipelines. 

*Answer 3:*

* As i mentioned above, I made a Azure devOps project and create a Pipeline providing the GitHub project https://github.com/alokhom/AzureFunctionTest and review the pipeline YAML
* It would have only Build and Deploy. We need to add the Test stage and its jobs, strategy and steps.
* I made depend the Test stage with Build stage success
* I made depend Deploy stage with Test Stage success or Build stage. Depends how you want to make the pipeline work based on test cases and importance. I tried both. 
* Now check-in updated code in Github and a webhook trigger it made to AzurePipelines pipeline and it would follow the dev/test/prod pipelines.
* For test cases  test for lint/pytest/tox. 
* I can write more test functions but have to check.



**Question 4**

Formulate a few sketches of scripts (richly commented upon) that would form a pipeline from development to deployment via testing. 

Imagine using Microsoft-hosted agents from the pre-defined agent pool in Azure Pipelines.

*Answer 4*

I am using Microsoft hosted agents. A linux OS supporting Python 3.x is created when making the function App instance on Azure portal. 


**Question 5**

What scripting language(s) did you choose and why?

*Answer 5*

*  Python - for Azure functions opens up a wide range of scenarios, including data processing, machine learning workloads, and automation scripting, that were previously hard to implement as serverless or FaaS solutions. 
*  Python runtime shares the unique Functions programming model, so you can easily import your Python scripts and manage all your dependencies using the standard requirements format. As always, the wide variety of triggers and bindings will enable you to seamlessly connect to cloud scale data sources and messaging services such as Azure Storage, Cosmos DB, Service Bus, Event Hubs, and Event Grid, by simply using method attributes to do so.
*  Lesser lines of code. 
*  strong community support 
*  portability. 


**Question 6**

Make a graphical account of your idea for a pipeline, incorporating components and resources.

Formulate (in yaml sketches and words) the Azure pipeline jobs and steps that you imagine should form a minimal, but realistic pipeline.




**Question 7**

What choices did you make regarding container hosting on Azure, and why?

*Answer 7*

consumption plan - 
It can help you pay when functions are running. Almost no memory/CPU consumed in this plan. Depends on length of function and frequency of computing. It has features of geo redundancy. 




**Question 8**

Can one use Application Insights to monitor and study telemetry of containerized (stateless) services in the form of Azure functions? 
If so, how? If not, how would you do it?

*Answer 8*

yes, used Application Insights config  in the host.json besides what was generated by func command https://github.com/alokhom/AzureFunctionTest/blob/master/host.json

To view near real-time streaming logs in Application Insights in the Azure portal, one can open the monitor for the function app. Dashboard > monitor > HttpExampleFunctionApp123456

Also use from CLI. 
func azure functionapp logstream HttpExampleFunctionApp123456 --browser

You can configure Insights alert publish  on Microsoft Teams. 


**Question 9**

Can you sketch a plan for scaling containerized Azure functions hosted in Azure with choices of tools and libraries?

*Answer 9*

* you can use this az cli command to instantly alter desired_prewarmed_count 


```
az resource update -g <resource_group> -n <function_app_name>/config/web --set properties.preWarmedInstanceCount=<desired_prewarmed_count> --resource-type Microsoft.Web/sites
```

* app scale limit 
* functionAppScaleLimit can be set to 0 or null for unrestricted

```
az resource update --resource-type Microsoft.Web/sites -g <RESOURCE_GROUP> -n <FUNCTION_APP-NAME>/config/web --set properties.functionAppScaleLimit=<SCALE_LIMIT>
```
There are many aspects of a function app that impacts how it scales, including host configuration, runtime footprint, and resource efficiency. 
You should also be aware of how connections behave as your function app scale
-----------
* Share and manage connections

* Avoid sharing storage accounts

* Don't mix test and production code in the same function app. If you're using a function app in production, don't add test-related functions and resources to it. It can cause unexpected overhead during production code execution.
Be careful what you load in your production function apps. Memory is averaged across each function in the app.

* Use async code but avoid blocking calls. Asynchronous programming is a recommended best practice, especially when blocking I/O operations are involved.

* Use multiple worker processes. To improve performance, especially with single-threaded runtimes like Python, use the FUNCTIONS_WORKER_PROCESS_COUNT to increase the number of worker processes per host (up to 10). Azure Functions then tries to evenly distribute simultaneous function invocations across these workers.

* Receive messages in batch whenever possible. Some triggers like Event Hub enable receiving a batch of messages on a single invocation. Batching messages has much better performance. You can configure the max batch size in the host.json file.  you'll need to explicitly set the cardinality property in your function.json to many in order to enable batching

* Configure host behaviors to better handle concurrency. The host.json file in the function app allows for configuration of host runtime and trigger behaviors. In addition to batching behaviors, you can manage concurrency for a number of triggers. Often adjusting the values in these options can help each instance scale appropriately for the demands of the invoked functions.Settings in the host.json file apply across all functions within the app, within a single instance of the function. For example, if you had a function app with two HTTP functions and maxConcurrentRequests requests set to 25, a request to either HTTP trigger would count towards the shared 25 concurrent requests. When that function app is scaled to 10 instances, the ten functions effectively allow 250 concurrent requests (10 instances * 25 concurrent requests per instance).
  

**Question 10**

Do make additional suggestions if the questions are lacking in any respect, such as viable options to pre-warm services in cases when they have been scaled down to zero.

*Answer 10*

if it is seen that some functions are more utilised then you can opt for premium plan where default pre-warmed instances is 1. It works with Always-ready instances can be set to say 1 or 2. So this elimites cold start with triggers going to  always-ready instances first. And then in the backgroud more pre-warmed instances are being prepared till max scale out instance limit. Max limit can be applied based on Application Insights and current/future trends. 

if you have Azure App Service plan those will be for instances where more predictive scaling and costs are required

scenario:

As soon as the first trigger comes in, the five always-ready instances become active, and a pre-warmed instance is allocated. The app is now running with six provisioned instances: the five now-active always ready instances, and the sixth pre-warmed and inactive buffer. If the rate of executions continues to increase, the five active instances are eventually used. When the platform decides to scale beyond five instances, it scales into the pre-warmed instance. When that happens, there are now six active instances, and a seventh instance is instantly provisioned and fill the pre-warmed buffer. This sequence of scaling and pre-warming continues until the maximum instance count for the app is reached. No instances are pre-warmed or activated beyond the maximum.



**some other best practices:**
* if you have a function that processes many thousands of queue messages, and another that is only called occasionally but has high memory requirements, you might want to deploy them in separate function apps so they get their own sets of resources and they scale independently of each other.
*  if your function stores a lot of data in memory, consider having fewer functions in a single app
* most are covered here : https://docs.microsoft.com/en-gb/azure/azure-functions/functions-best-practices
* In function apps in Azure, you should instead follow the steps in How to disable functions in Azure Functions to disable specific functions 
* use  App Service Advisor
* follow PEP 8 -- Style Guide for Python Code
