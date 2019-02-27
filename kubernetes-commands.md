# Kubernetes (K8s) Commands and Utilities

(For K8s related [links](https://github.com/ravisharda/resource-center/blob/master/containers/kubernetes-links.md) @ [my resource center](https://github.com/ravisharda/resource-center/))

## Minikube related

* Version: `minikube version`
* Starting: ``minikube start``
  
  If you see an error ``Error with pre-create check: "This computer doesn't have VT-X/AMD-v enabled. Enabling it in the BIOS is mandatory"``, here's the fix: 
  * Power off the VM. 
  * ``VM-->Settings--->Processor-->Enable "Virtualize Intel VT-x/EPT "...``
  * Now, try again.
  
  Minikube also supports a --vm-driver=none option that runs the Kubernetes components on the host and not in a VM. Using this driver requires Docker and a Linux environment but not a hypervisor: ``minikube --vm-driver=none start``
* Stopping: ``minikube stop``
* Minikube config
  * Note that Minikube configuration file is located under: ``~/.minikube/machines/minikube/config.json``
  * To view config: ``kubectl config view``
* Deleting a minikube VM: ``minikube delete -p minikube``
* Access minikube VM using SSH: ``minikube ssh`` ``sudo su -``
* Config: ``/home/rsharda/.minikube/machines/minikube/config.json``
* Enable Kubernetes dashboard: 
  * Check that the addon is installed: ``minikube addons list``
  * If dashboard is disabled: ``minikube addons enable dashboard``
  * To open directly on your default browser, use: ``minikube dashboard``
  * To get the URL of the dashboard: ``minikube dashboard --url``

## Kubectl
* Version: ``kubectl version -o json``
* Checking cluster status: ``kubectl cluster-info``
* To check running nodes: ``kubectl get nodes``

## Creating a Azure Kubernetes Service

You can create the cluster using the Azure portal. Here we'll do so using Azure CLI. 

```
//Launch powershell as admin. 

// Login to Azure. The following launches a URL in the browser where you can log in. 
az login

// Create a variable that holds the resource group name
$resourceGroup = "myk8scluster"

// Create the resource group
// You can find all locations using "az account list-locations". Not using "southindia" as using it will
// throw error: "The VM size of AgentPoolProfile:nodepool1 is not allowed in your subscription in location 'southindia'."
az group create -n $resourceGroup -l "southeastasia"

// Create a variable to hold cluster name
$clusterName = "ravik8scluster"

// Create the AKS cluster, with a starting node count (we'll scale the node count to 3 later). 
// We are asking to generate SSH keys that we can use to securely manage this cluster.
az aks create -g $resourceGroup -n $clusterName --node-count 1 --generate-ssh-keys

// Now we can use use kubectl command to explore it. 


```

See more about Azure CLI in the [documentation](https://docs.microsoft.com/en-us/cli/azure/?view=azure-cli-latest).