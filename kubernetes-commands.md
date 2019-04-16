# Kubernetes (K8s) Commands and Utilities

(For K8s related [links](https://github.com/ravisharda/resource-center/blob/master/containers/kubernetes-links.md) @ [my resource center](https://github.com/ravisharda/resource-center/))

**Table of Contents:**
* [Kubernetes Cluster in zure Kubernetes Service (AKS)](#kubernetes-cluster-in-azure-kubernetes-service-aks)
    * [Creating a new AKS Cluster using Azure CLI](#creating-a-new-aks-cluster-using-azure-cli)
    * [Deleting an AKS Cluster using Azure CLI](#deleting-an-aks-cluster-using-azure-cli)
    * [Scaling AKS Cluster to More Nodes](#scaling-aks-cluster-to-more-nodes)
    * [Opening the Kernetes Dashboard for K8s cluster running in AKS](#opening-the-kernetes-dashboard-for-k8s-cluster-running-in-aks)
    * [More About AKS](#more-about-aks)
* [Deploying a Pravega Kubernetes Cluster](#deploying-a-pravega-kubernetes-cluster)
    * [Step 1: Install Zookeeper using Pravega Zookeeper Operator](#step-1-install-zookeeper-using-pravega-zookeeper-operator)
    * [Step 2: Install Helm](#step-2-install-helm)
    * [Step 3: Install the Pravega Operator](#step-3-install-the-pravega-operator)
    * [Step 4: Deploy a Pravega Cluster](#step-4-deploy-a-pravega-cluster)
    * [Step 5: Use the Pravega cluster](#step-5-use-the-pravega-cluster)
    * [Deleting the deployment](#deleting-the-deployment)
    * [Troubleshooting](#troubleshooting)
* [Deploying Minikube](#deploying-minikube)
* [Kubectl Command Reference](#kubectl-command-reference)
 

## Kubernetes Cluster in Azure Kubernetes Service (AKS)

### Creating a new AKS Cluster using Azure CLI

You can create the cluster using the Azure portal. Here we'll do so using Azure CLI. You migth also want to see: [Quickstart: Deploy an Azure Kubernetes Service (AKS) cluster using the Azure CLI](https://docs.microsoft.com/en-us/azure/aks/kubernetes-walkthrough) 

```
//Launch powershell as admin. 

// Login to Azure. The following launches a URL in the browser where you can log in. 
az login

// Create a variable that holds the resource group name
$resourceGroup = "Testk8sResourceGroup"

// Create the resource group
// You can find all locations using "az account list-locations". Not using "southindia" as using it will
// throw error: "The VM size of AgentPoolProfile:nodepool1 is not allowed in your subscription 
// in location 'southindia'."
az group create -n $resourceGroup -l "southeastasia"

// Create a variable to hold cluster name
$clusterName = "Testk8scluster"

// Create the AKS cluster, with a starting node count (we'll scale the node count to 3 later). 
// We are asking to generate SSH keys that we can use to securely manage this cluster.
az aks create -g $resourceGroup -n $clusterName --node-count 1 --generate-ssh-keys
(or)
// Find the supported VM SKUs or find it https://zimmergren.net/azure-container-services-aks-supported-vm-sizes/
az vm list-skus --location southeastasia
//Now create aks cluster
az aks create -g $resourceGroup -n $clusterName --node-count 1 --generate-ssh-keys --node-vm-size Standard_A2_v2
az aks create --name $clusterName \
              --resource-group $resourceGroup \
              --ssh-key-value ssh-key-<CLUSTER-NAME>.pub \
              --node-count 1 \
              --node-vm-size Standard_D2s_v3 \
              --output table

// Now we can use use kubectl command to explore it. To do so, first we have to install it. 
az aks install-cli

// Add kubectl to path

// Relaunch Powershell and verify kubecctl is working
kubectl version --short

// If you are recreating the cluster, be sure to C:\Users\shardr\.kube\config file. Remove the existing cluster, context, 
// and user for the given name. You will then be able to run `az aks get-credentials` again. Or, may be try this: 
az aks get-credentials --resource-group $resourceGroup --name $clusterName --overwrite-existing

// To configure kubectl to connect to your Kubernetes cluster, use the az aks get-credentials command. 
// This command downloads credentials and configures the Kubernetes CLI to use them.
az aks get-credentials --resource-group $resourceGroup --name $clusterName


// To verify the connection to your cluster, use the kubectl get command to return a list 
// of the cluster nodes.
kubectl get nodes

```
### Deleting an AKS Cluster using Azure CLI

Run: ``az group delete --name $resourceGroup --yes --no-wait``

### Scaling AKS Cluster to More Nodes

Run: ``az aks scale -g $resourceGroup -n $clusterName --node-count 3``

### Opening the Kernetes Dashboard for K8s cluster running in AKS

As earlier:
* ``$resourceGroup = "Testk8sResourceGroup"``
* ``$clusterName = "Testk8scluster"``

Now, run:
* ``kubectl create clusterrolebinding kubernetes-dashboard -n kube-system --clusterrole=cluster-admin --serviceaccount=kube-system:kubernetes-dashboard``
* ``az aks browse -g $resourceGroup -n $clusterName``

### More about AKS

**Further Reading:**
* https://docs.microsoft.com/en-us/azure/aks/kubernetes-walkthrough
* [Nicely drafted steps and additional options - Stackify](https://stackify.com/azure-container-service-kubernetes/)
* [Kubernetes Commands Cheatsheet - Official](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

## Deploying a Pravega Kubernetes Cluster

### Step 1: Install Zookeeper using Pravega Zookeeper Operator

See the steps [here](https://github.com/pravega/zookeeper-operator#usage). 

### Step 2: Install Helm 

* Install Chocolatey
* Install Helm using Chocolatey: ``choco install kubernetes-helm``
* Initialize Helm: ``helm init``
* Now, add service accounts: 
  
  ```
  kubectl create serviceaccount --namespace kube-system tiller
  
  kubectl create clusterrolebinding tiller-cluster-rule --clusterrole=cluster-admin --serviceaccount=kube-system:tiller
  
  # In Powershell, you need to escape the double quote characters
  kubectl patch deploy --namespace kube-system tiller-deploy -p '{\"spec\":{\"template\":{\"spec\":{\"serviceAccount\":\"tiller\"}}}}'
  ```
* Initializer Helm and Tiller: 
  
  ```
  # Initialize helm and tiller. This command only needs to run once per Kubernetes cluster, it will create a tiller 
  # deployment in the kube-system namespace and setup your local helm client.
  helm init --service-account tiller --wait
  ```
* Verify helm is working: ``helm version``

**Searching a specific chart:**

* Check the version `helm search stable/nfs-server-provisioner`
* Install the specific release `helm search stable/nfs-server-provisioner --name `

**See examples here:**
* https://github.com/dotnet-architecture/eShopOnContainers/wiki/10.-Deploying-to-Kubernetes-(AKS-and-local)-using-Helm-Charts
* https://github.com/helm/charts/tree/master/stable/nfs-server-provisioner

### Step 3: Install the Pravega Operator

See the steps here: https://github.com/pravega/pravega-operator#install-the-operator. 

```
# Create a deployment yaml file
pravega-operator> kubectl create -f pravega.yaml

# Verify that the cluster instances and its components are being created.
pravega-operator> kubectl get PravegaCluster
pravega-operator> kubectl get all -l pravega_cluster
```

### Step 4: Deploy a Pravega Cluster

See the steps here: https://github.com/pravega/pravega-operator#deploy-a-sample-pravega-cluster. When creating the Pravega cluster, you'll need the Zookeeper IP Address. Use this command to fetch it (extract the cluster IP address): `kubectl get all -l app=example`

This involves two sub-steps:
* Provisioning an NFS volume provisioned by the NFS Server Provisioner helm chart to provide Tier 2 storage. 
* Creating a Pravega cluster
* Verifying the cluster is running
  ```
  kubectl get PravegaCluster
  
  kubectl get all -l pravega_cluster=pravega
  or
  kubectl get pods -l pravega_cluster=pravega
  ```

### Step 5: Use the Pravega cluster

See the steps here: https://github.com/pravega/pravega-operator#deploy-a-sample-pravega-cluster

A PravegaCluster instance is only accessible WITHIN the cluster (i.e. no outside access is allowed) using the following endpoint in the PravegaClient.
```
tcp://<cluster-name>-pravega-controller.<namespace>:9090
```

The REST management interface is available at:
```http://<cluster-name>-pravega-controller.<namespace>:10080/```

Should you want to open a shell inside a container: 

```
$ kubectl get pods
$ kubectl exec -it pravega-operator-64c767dbd4-jkpbl -- sh
$ kubectl exec -it pravega-pravega-segmentstore-0 -- sh
```

```
# Fetch the controller IP
kubectl get all -l pravega_cluster

> curl -v -k -u admin:1111_aaaa https://10.0.30.253:10080/v1/scopes
```

**To enable direct access to the cluster:**  

For debugging and development you might want to access the Pravega cluster directly. For example, if you created the cluster with name pravega in the default namespace you can forward ports of the Pravega controller pod with name pravega-pravega-controller-68657d67cd-w5x8b as follows:
```
kubectl port-forward -n default pravega-pravega-controller-68657d67cd-w5x8b 9090:9090 10080:10080
```
Note that out cluser name is `pravega` and `namespace` is: 

Get the name of the pods, using the following command:
```
PS C:\Workspace\pravega-operator> kubectl get pods
NAME                                          READY   STATUS    RESTARTS   AGE
example-0                                     1/1     Running   0          17h
example-1                                     1/1     Running   0          17h
example-2                                     1/1     Running   1          17h
pravega-bookie-0                              0/1     Pending   0          52m
pravega-bookie-1                              0/1     Pending   0          52m
pravega-bookie-2                              0/1     Pending   0          52m
pravega-operator-6c6d9fff4f-cfmwb             1/1     Running   0          17h
**pravega-pravega-controller-5d6686cf85-r94x7   1/1     Running   0          52m**
pravega-pravega-segmentstore-0                0/1     Pending   0          52m
punk-sasquatch-nfs-server-provisioner-0       1/1     Running   0          85m
zookeeper-operator-6b6657ffdb-qpw92           1/1     Running   0          17h
```

**Note: What is port forwarding?**
According to a [discussion](https://stackoverflow.com/questions/51468491/how-kubectl-port-forward-works) in Stackoverflow: 

> kubectl port-forward forwards connections to a local port to a port on a pod. Compared to kubectl proxy, kubectl port-forward is more generic as it can forward TCP traffic while kubectl proxy can only forward HTTP traffic.
>
> kubectl port-forward is useful for testing/debugging purposes so you can access your service locally without exposing it.
>
> Below is the name of the pod and it will forward it's port 6379 to localhost:6379.
>
> kubectl port-forward redis-master-765d459796-258hz 6379:6379 
> which is the same as
>
> kubectl port-forward pods/redis-master-765d459796-258hz 6379:6379
>or
> kubectl port-forward deployment/redis-master 6379:6379 
> or
>
> kubectl port-forward rs/redis-master 6379:6379 
> or
> 
> kubectl port-forward svc/redis-master 6379:6379
>
> [Here](https://github.com/lvthillo/explore-minikube/tree/master/deployment/deployment) is also some small port forwarding example to access a database service (clusterip) without exposing it.


### Deleting the deployment

```
# Uninstalling the Pravega Cluster
pravega-operator> kubectl delete -f pravega.yaml
pravega-operator> kubectl delete -f pvc.yaml

# Uninstalling the NFS-provisioner chart
## First find the name of the chart
helm list
## Use the `name` from the output of the last command to delete the chart
helm delete <name>

# Uninstaling the Zookeeper cluster
zookeeper-operator> kubectl delete -f zk.yaml

# Check if the Pravega Controller, Segment Store and Bookkeeper pods have terminated.
> kubectl get pods

# Uninstalling the pravega operator
> kubectl delete -f deploy

# Uninstalling the Zookeeper operator
```
### Troubleshooting

```
# Labels attached to a pod
> kubectl get pod <one of your pods> -o template --template='{{.metadata.labels}}'
> kubectl get pod pravega-pravega-segmentstore-2 -o template --template='{{.metadata.labels}}'
'map[app:pravega-cluster component:pravega-segmentstore controller-revision-hash:pravega-pravega-segmentstore-58695f5dc6 pravega_cluster:pravega statefulset.kubernetes.io/pod-name:pravega-pravega-segmentstore-2]'

# All pods with the given label 
> kubectl get all -l app=pravega-cluster

# All segment store pods (filtered by label selector pravega-segmentstore)
> kubectl get all -l component=pravega-segmentstore
NAME                                 READY   STATUS    RESTARTS   AGE
pod/pravega-pravega-segmentstore-0   1/1     Running   0          4h
pod/pravega-pravega-segmentstore-1   1/1     Running   0          4h
pod/pravega-pravega-segmentstore-2   1/1     Running   0          4h

NAME                                            TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)     AGE
service/pravega-pravega-segmentstore-headless   ClusterIP   None         <none>        12345/TCP   4h

# Print logs of pods with given label
> kubectl logs -l app=pravega-cluster

# Open a shell inside the Pod/container
> kubectl exec -it pravega-pravega-segmentstore-0 -- sh
> kubectl exec -it pravega-zookeeper-0 -- sh


```

## Deploying Minikube

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

## Kubectl Command Reference

**Cheatsheets:**
* https://kubernetes.io/docs/reference/kubectl/cheatsheet/

* Kubectl Version: ``> kubectl version -o json``
* Checking cluster status: ``> kubectl cluster-info``
* To check running nodes: ``> kubectl get nodes``
* `> kubectl get all -l app=example`
* Querying nodes:
  
  ```
  > kubectl get nodes
  > kubectl get nodes -o wide
  ```

* Querying deployments: 
  * See all the Deployments active in your current namespace: `> kubectl get deployments`
  * Get more detailed information on this specific Deployment: `> kubectl describe deployments/demo`

* Querying pods:
  * Get info about a pod: 
    
    ```
    > kubectl describe pod <pod_name>`
  
    # See extra info:
    > kubectl describe pod <pod_name> -o wide
    ```
  * Get info about pods
  
    ```
    > kubectl get pods
    
    # While the default output format is plain text, you can also get info in JSON format.     
    > kubectl get pods -n kube-system -o json
    
    # Output of JSON format is large. You may want to use jq to filter it. Install jq using "apt install jq"
    # https://jqplay.org/ is an online playground for jq.
    > kubectl get pods -n kube-system -o json | jq '.items[].metadata.name'
    > kubectl get pods -o json --all-namespaces | jq '.items |

    # List all pods in ps output format.
    > kubectl get pods

    # List all pods in ps output format with more information (such as node name).
    > kubectl get pods -o wide

    # List a single replication controller with specified NAME in ps output format.
    > kubectl get replicationcontroller web

    # List deployments in JSON output format, in the "v1" version of the "apps" API group:
    > kubectl get deployments.v1.apps -o json

    # List a single pod in JSON output format.
    > kubectl get -o json pod web-pod-13je7

    # List a pod identified by type and name specified in "pod.yaml" in JSON output format.
    > kubectl get -f pod.yaml -o json

    # Return only the phase value of the specified pod.
    > kubectl get -o template pod/web-pod-13je7 --template={{.status.phase}}

    # List all replication controllers and services together in ps output format.
    > kubectl get rc,services

    # List one or more resources by their type and names.
    kubectl get rc/web service/frontend pods/web-pod-13je7
    ```
