# AzureFunctionTest

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
*  low memory footprint in some python libraries like NumPy.


**Question 6**

Make a graphical account of your idea for a pipeline, incorporating components and resources.

Formulate (in yaml sketches and words) the Azure pipeline jobs and steps that you imagine should form a minimal, but realistic pipeline.




**Question 7**

What choices did you make regarding container hosting on Azure, and why?

*Answer 7*

consumption plan - 
It can help you pay when functions are running. Almost no memory/CPU consumed in this plan. Depends on length of function and frequency of computing. 

One can check Premium plans or Dedicated plans and opt for features of geo distribution. There also there should be a purpose to meet coming from Architectural view points. 




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

* Scaling containerized workloads is a key feature of container orchestrators. AKS supports automatic scaling across two dimensions: Container instances and compute nodes. Together they give AKS the ability to quickly and efficiently respond to spikes in demand and add additional resources. With the declarative approach, you create a configuration file, called a manifest, to describe what you want instead of what to do. Kubernetes reads the manifest and transforms your desired end state into actual end state. It helps in  reliable and repeatable deployments. The manifest file becomes a project artifact and is used in your CI/CD pipeline for automating Kubernetes deployments When using declarative configuration, you can preview the changes that will be made before committing them by using kubectl diff -f FOLDERNAME against the folder where your configuration files are located. Once you're sure you want to apply the changes, run kubectl apply -f FOLDERNAME. Add -R to recursively process a folder hierarchy. They instruct the Kubernetes deployment controller on how to deploy new changes, scale out load, or roll back to a previous revision. If a cluster is unstable, a declarative deployment will automatically return the cluster back to a desired state. For example, if a node should crash, the deployment mechanism will redeploy a replacement to achieve your desired state. It provides improved change control and better support for continuous deployment using a build and deploy pipeline*

* When should you avoid using containers and orchestrators?
If you're unable to build your application following the Twelve-Factor App principles, you should consider avoiding containers and orchestrators. In these cases, consider a VM-based hosting platform, or possibly some hybrid system. With it, you can always spin off certain pieces of functionality into separate containers or even serverless functions *

* Docker Desktop /minikube / VS Docker tooking / VS code docket tooling - are good examples or local tooling. Along with Visual Studio Docker Tooling, you can choose Docker support*

*  you can wrap Azure Functions inside Docker containers and deploy them using the same processes and tools as the rest of your Kubernetes-based app. You'll need a custom image that supports dependencies or a configuration not supported by the default image. In these cases, it makes sense to deploy your function in a custom Docker container . if you deploy your function to a Kubernetes cluster, you'll no longer benefit from the built-in scaling provided by Azure Functions. You'll need to use Kubernetes' scaling features *
------
* With Docker * 
```
func init ProjectName --worker-runtime dotnet --docker
```
* --docker option generates a Dockerfile for the project  which defines a suitable custom container for use with Azure Functions and the selected runtime. * 
```
docker build --tag <DOCKER_ID>/azurefunctionsimage:v1.0.0 .
docker run -p 8080:80 -it <docker_id>/azurefunctionsimage:v1.0.0
docker login
docker push <docker_id>/azurefunctionsimage:v1.0.0
az login
az group create --name AzureFunctionsContainers-rg --location westeurope
az storage account create --name <storage_name> --location westeurope --resource-group AzureFunctionsContainers-rg --sku Standard_LRS
az functionapp plan create --resource-group AzureFunctionsContainers-rg --name myPremiumPlan --location westeurope --number-of-workers 1 --sku EP1 --is-linux
az functionapp create --name <app_name> --storage-account <storage_name> --resource-group AzureFunctionsContainers-rg --plan myPremiumPlan --runtime <functions runtime stack> --deployment-container-image-name <docker_id>/azurefunctionsimage:v1.0.0

display connection string an add the setting to the function app.
az storage account show-connection-string --resource-group AzureFunctionsContainers-rg --name <storage_name> --query connectionString --output tsv
az functionapp config appsettings set --name <app_name> --resource-group AzureFunctionsContainers-rg --settings AzureWebJobsStorage=<connection_string>

Retrieve the function URL with the access (function) key by using the Azure portal, or by using the Azure CLI with the az rest command.)
on Azure portal in Azure functions, select default (function key) and then copy the URL to the clipboard.
Paste the function URL into your browser's address bar, adding the parameter &name=Azure to the end of this URL. 
Enable CD to Azure
az functionapp deployment container config --enable-cd --query CI_CD_URL --output tsv --name <app_name> --resource-group AzureFunctionsContainers-rg
Copy the deployment webhook URL to the clipboard.
Open Docker Hub, sign in, and select Repositories on the nav bar. Locate and select image, select the Webhooks tab, specify a Webhook name, paste your URL in Webhook URL, and then select Create:
With the webhook set, Azure Functions redeploys your image whenever you update it in Docker Hub
```
------
* with AKS *

![image](https://user-images.githubusercontent.com/12021776/109802892-a4c36a80-7c20-11eb-9481-0e8f0291d218.png)

``` 

# First create a resource group
az group create --name myResourceGroup --location eastus

# Now create the AKS cluster and enable the cluster autoscaler
az aks create \
  --resource-group myResourceGroup \
  --name myAKSCluster \
  --node-count 1 \
  --vm-set-type VirtualMachineScaleSets \
  --load-balancer-sku standard \
  --enable-cluster-autoscaler \
  --min-count 1 \
  --max-count 3
  

func init --docker-only

To build an image and deploy your functions to Kubernetes, run the following command:
docker login
func kubernetes deploy --name <name-of-function-deployment> --registry <container-registry-username>

The deploy command executes a series of actions:
The Dockerfile created earlier is used to build a local image for the function app.
The local image is tagged and pushed to the container registry where the user is logged in.
A manifest is created and applied to the cluster that defines a Kubernetes Deployment resource, a ScaledObject resource, and Secrets, which includes environment variables imported from your local.settings.json file.

```



* scaling up using AKS - Scaling up a cloud-native application involves choosing more capable resources from the cloud vendor. For example, you can a new node pool with larger VMs in your Kubernetes cluster. Then, migrate your containerized services to the new pool.
* scaling out in AKS - Cloud-native applications often experience large fluctuations in demand and require scale on a moment's notice. They favor scaling out. Scaling out is done horizontally by adding additional machines (called nodes) or application instances to an existing cluster. In Kubernetes, you can scale manually by adjusting configuration settings for the app (for example, scaling a node pool), or through autoscaling. AKS clusters can autoscale in one of two ways: *

* First, the Horizontal Pod Autoscaler monitors resource demand and automatically scales your POD replicas to meet it. When traffic increases, additional replicas are automatically provisioned to scale out your services. Likewise, when demand decreases, they're removed to scale-in your services. You define the metric on which to scale, for example, CPU usage. You can also specify the minimum and maximum number of replicas to run. AKS monitors that metric and scales accordingly.

Next, the AKS Cluster Autoscaler feature enables you to automatically scale compute nodes across a Kubernetes cluster to meet demand. With it, you can automatically add new VMs to the underlying Azure Virtual Machine Scale Set whenever more compute capacity of is required. It also removes nodes when no longer required. 

Working together, both ensure an optimal number of container instances and compute nodes to support fluctuating demand. The horizontal pod autoscaler optimizes the number of pods required. The cluster autoscaler optimizes the number of nodes required. 

Both the horizontal pod autoscaler and cluster autoscaler can also decrease the number of pods and nodes as needed. The cluster autoscaler decreases the number of nodes when there has been unused capacity for a period of time. Pods on a node to be removed by the cluster autoscaler are safely scheduled elsewhere in the cluster. The cluster autoscaler may be unable to scale down if pods can't move, such as in the following situations:

A pod is directly created and isn't backed by a controller object, such as a deployment or replica set.
A pod disruption budget (PDB) is too restrictive and doesn't allow the number of pods to be fall below a certain threshold.
A pod uses node selectors or anti-affinity that can't be honored if scheduled on a different node.

Aside from Azure Kubernetes Service (AKS), you can also deploy containers to Azure App Service for Containers and Azure Container Instances. *




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
