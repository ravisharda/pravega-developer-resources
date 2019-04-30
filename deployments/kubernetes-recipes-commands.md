# Kubernetes (K8s) Recipes and Commands

NOTE:
* The steps shown here show use paths, endpoint addresses, etc. that work on one particular environment. They may not work you: be sure to replace them. 
* The source of truth for many of the steps - especially the ones related to Pravega operator - is really the documentation under the corresponding repo. I've reproduced them here for quick reference. Be sure to confirm the steps are current and applicable to the version you use. 
* The container images used may not be the one that you seeking to work with. Replace the versions as needed. 

**Table of Contents:**

  * [Kubernetes Cluster in Azure Kubernetes Service (AKS)](#kubernetes-cluster-in-azure-kubernetes-service--aks-)
    + [Creating a new AKS Cluster using Azure CLI](#creating-a-new-aks-cluster-using-azure-cli)
    + [Scaling AKS Cluster to More Nodes](#scaling-aks-cluster-to-more-nodes)
    + [Deleting an AKS Cluster using Azure CLI](#deleting-an-aks-cluster-using-azure-cli)
    + [Opening the Kernetes Dashboard for K8s cluster running in AKS](#opening-the-kernetes-dashboard-for-k8s-cluster-running-in-aks)
    + [More about AKS](#more-about-aks)
  * [Deploying a Pravega Kubernetes Cluster](#deploying-a-pravega-kubernetes-cluster)
    + [Step 1: Install Zookeeper using Pravega Zookeeper Operator](#step-1--install-zookeeper-using-pravega-zookeeper-operator)
    + [Step 2: Deploy a Zookeeper Cluster using the Zookeeper Operator](#step-2--deploy-a-zookeeper-cluster-using-the-zookeeper-operator)
    + [Step 3: Install & Configure Helm](#step-3--install---configure-helm)
    + [Step 4: Install the Pravega Operator](#step-4--install-the-pravega-operator)
    + [Step 5: Deploy a Pravega Cluster](#step-5--deploy-a-pravega-cluster)
      - [Without External Access](#without-external-access)
      - [With External Access Enabled](#with-external-access-enabled)
      - [With External Access and TLS Enabled](#with-external-access-and-tls-enabled)
    + [Step 6: Using the Pravega cluster](#step-6--using-the-pravega-cluster)
      - [Port forwarding to access the cluster](#port-forwarding-to-access-the-cluster)
    + [Deleting the deployment](#deleting-the-deployment)
    + [Troubleshooting](#troubleshooting)
  * [Deploying Minikube](#deploying-minikube)
  * [Kubectl Command Reference](#kubectl-command-reference)

<small><i><a href='http://ecotrust-canada.github.io/markdown-toc/'>Table of contents generated with markdown-toc</a></i></small>

## Kubernetes Cluster in Azure Kubernetes Service (AKS)

### Creating a new AKS Cluster using Azure CLI

You can create the cluster using the Azure portal. Here we'll do so using Azure CLI. You migth also want to see: [Quickstart: Deploy an Azure Kubernetes Service (AKS) cluster using the Azure CLI](https://docs.microsoft.com/en-us/azure/aks/kubernetes-walkthrough) 

```powershell
# Launch powershell as admin. 

# Login to Azure. The following launches a URL in the browser where you can log in. 
PS> az login

# Create a variable that holds the resource group name
PS> $resourceGroup = "Testk8sResourceGroup"

# Create the resource group:
# You can find all locations using "az account list-locations". Not using "southindia" as using it will
# throw error: "The VM size of AgentPoolProfile:nodepool1 is not allowed in your subscription 
# in location 'southindia'."
PS> az group create -n $resourceGroup -l "southeastasia"

# Create a variable to hold cluster name
PS> $clusterName = "Testk8scluster"

# Create the AKS cluster, with a starting node count (we'll scale the node count to 3 later). 
# We are asking to generate SSH keys that we can use to securely manage this cluster.
PS> az aks create -g $resourceGroup -n $clusterName --node-count 1 --generate-ssh-keys
# Alternatively, Find the supported VM SKUs and create the cluster 
PS> az vm list-skus --location southeastasia
PS> az aks create -g $resourceGroup -n $clusterName --node-count 1 --generate-ssh-keys --node-vm-size Standard_A2_v2
# Or:
PS> az aks create --name $clusterName `
              --resource-group $resourceGroup `
              --ssh-key-value ssh-key-<CLUSTER-NAME>.pub `
              --node-count 1 `
              --node-vm-size Standard_D2s_v3 `
              --output table`

# Now we can use use kubectl command to explore it. To do so, first we have to install it. 
# If you already have this installed, you can skip this step. 
PS> az aks install-cli
# Add kubectl to path
# Relaunch Powershell and verify kubecctl is working
PS> kubectl version --short

# If you are recreating the cluster, be sure to C:\Users\shardr\.kube\config file. Remove the existing cluster, context, 
# and user for the given name. You will then be able to run `az aks get-credentials` again. Or, may be try this: 
PS> az aks get-credentials --resource-group $resourceGroup --name $clusterName --overwrite-existing
# Doing it for the first time? To configure kubectl to connect to your Kubernetes cluster, use the az aks get-credentials command. 
# This command downloads credentials and configures the Kubernetes CLI to use them.
PS> az aks get-credentials --resource-group $resourceGroup --name $clusterName

# To verify the connection to your cluster, use the kubectl get command to return a list 
# of the cluster nodes.
kubectl get nodes

```
### Scaling AKS Cluster to More Nodes

``PS> az aks scale -g $resourceGroup -n $clusterName --node-count 3``

### Deleting an AKS Cluster using Azure CLI

``PS> az group delete --name $resourceGroup --yes --no-wait``

### Opening the Kernetes Dashboard for K8s cluster running in AKS

As earlier:
* ``$resourceGroup = "Testk8sResourceGroup"``
* ``$clusterName = "Testk8scluster"``

Now, run:
* ``PS> kubectl create clusterrolebinding kubernetes-dashboard -n kube-system --clusterrole=cluster-admin --serviceaccount=kube-system:kubernetes-dashboard``
* ``PS> az aks browse -g $resourceGroup -n $clusterName``

### More about AKS

**Further Reading:**
* https://docs.microsoft.com/en-us/azure/aks/kubernetes-walkthrough
* [Nicely drafted steps and additional options - Stackify](https://stackify.com/azure-container-service-kubernetes/)
* [Kubernetes Commands Cheatsheet - Official](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

## Deploying a Pravega Kubernetes Cluster

### Step 1: Install Zookeeper using Pravega Zookeeper Operator

See the steps [here](https://github.com/pravega/zookeeper-operator#usage). 

Here are the current steps I use:
```powershell
PS> cd C:\Workspace\zookeeper-operator-master

# Register the ZookeeperCluster custom resource definition (CRD).
PS> kubectl create -f deploy/crds/zookeeper_v1beta1_zookeepercluster_crd.yaml

# Create the operator role and role binding for all namespaces
PS> kubectl create -f deploy/all_ns/rbac.yaml

# Deploy the Zookeeper operator.
kubectl create -f deploy/all_ns/operator.yaml

# Verify that the Kubernetes Operator is running
PS> kubectl get deploy
```

### Step 2: Deploy a Zookeeper Cluster using the Zookeeper Operator

See the steps [here](https://github.com/pravega/zookeeper-operator#deploy-a-sample-zookeeper-cluster)

Here are the current steps I use:

```powershell
# Deploy a Zookeeper cluster
PS> kubectl create -f zk.yaml

# Wait for ZK cluster instances to be running. Also, note the IP address of the Zookeeper instance.
PS> kubectl get zk
```

Here's how my zk.yaml file looks:

```yaml
apiVersion: "zookeeper.pravega.io/v1beta1"
kind: "ZookeeperCluster"
metadata:
  name: "pravega-zookeeper"
spec:
  size: 3
```

### Step 3: Install & Configure Helm 

These tasks are to be done only **once** per client host. 

* Install Chocolatey
* Install Helm using Chocolatey: ``choco install kubernetes-helm``

The following tasks are done on a **per-Kubernetes cluster** basis:

* Initialize Helm: ``PS> helm init``
* Now, add service accounts: 
  
  ```
  PS> kubectl create serviceaccount --namespace kube-system tiller
  
  PS> kubectl create clusterrolebinding tiller-cluster-rule --clusterrole=cluster-admin --serviceaccount=kube-system:tiller
  
  # In Powershell, you need to escape the double quote characters
  PS> kubectl patch deploy --namespace kube-system tiller-deploy -p '{\"spec\":{\"template\":{\"spec\":{\"serviceAccount\":\"tiller\"}}}}'
  ```
* Initialize Helm and Tiller: 
  
  ```
  # Initialize helm and tiller. This command only needs to run once per Kubernetes cluster, it will create a tiller 
  # deployment in the kube-system namespace and setup your local helm client.
  PS> helm init --service-account tiller --wait
  ```
* Verify helm is working: ``PS> helm version``

**Searching a specific chart:**

* Check the version `helm search stable/nfs-server-provisioner`
* Install the specific release `helm search stable/nfs-server-provisioner --name `

**See examples here:**
* https://github.com/dotnet-architecture/eShopOnContainers/wiki/10.-Deploying-to-Kubernetes-(AKS-and-local)-using-Helm-Charts
* https://github.com/helm/charts/tree/master/stable/nfs-server-provisioner

### Step 4: Install the Pravega Operator

See the steps here: https://github.com/pravega/pravega-operator#install-the-operator. 

The current steps I use are:
```
PS> cd C:\workspace\pravega-operator

# Run the following command to install the PravegaCluster custom resource definition (CRD), 
# create the pravega-operator service account, roles, bindings, and the deploy the Operator.
PS> kubectl create -f deploy

# Verify that the Pravega Operator is running.
PS> kubectl get deploy
```

### Step 5: Deploy a Pravega Cluster

See the steps [here](https://github.com/pravega/pravega-operator#deploy-a-sample-pravega-cluster). 

1. Provision tier2 storage. 

```
# First, we need to provision Tier 2 storage. We'll use an NFS volume provisioned by the 
# NFS Server Provisioner helm chart to provide Tier 2 storage.
PS> helm install stable/nfs-server-provisioner

# Verify that the nfs storage class is now available.
PS> kubectl get storageclass

# Now, create a PersistentVolumeClaim that will be used as Tier 2 for Pravega.
# See an example file below.
PS> kubectl create -f pvc.yaml

```

Yaml file I use:

`pvc.yaml`: 

```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: pravega-tier2
spec:
  storageClassName: "nfs"
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 50Gi
```

#### Without External Access

Assuming I have a [yaml file](https://github.com/ravisharda/pravega-developer-resources/blob/master/deployments/yaml-examples/k8s-internal-access-only.md) specifying the deployment, I deploy a cluster using these steps:

```powershell
# Deploy the cluster
PS> kubectl create -f pravega.yaml

# Verify that the cluster instances and its components are being created.
PS> kubectl get PravegaCluster
PS> kubectl get all -l pravega_cluster

```

#### With External Access Enabled

See [this](https://github.com/pravega/pravega-operator/blob/master/doc/external-access.md) Pravega Operator documentation for the latest information.

```powershell
# The three yaml files have content as described in the external access documentation referenced above. 
PS> kubectl create -f serviceacct.yaml
PS> kubectl create -f roles.yaml
PS> kubectl create -f rolebinding.yaml

PS> kubectl create -f pravega.yaml
```

#### With External Access and TLS Enabled

1. Create the service accounts, roles and role bindings. 

```powershell
# The three yaml files have content as described in the external access documentation referenced above. 
PS> kubectl create -f serviceacct.yaml
PS> kubectl create -f roles.yaml
PS> kubectl create -f rolebinding.yaml
```

2. Now, create the secrets:

```powershell
> cd c:\workspace\pki
> kubectl create secret generic controller-tls `
        --from-file=controllerTlsCertFile=./controller01.pem `
	--from-file=controllerCacert=./ca-cert `
	--from-file=controllerTlsKeyFile=./controller01.key.pem `
	--from-file=controllerTlsKeyStoreFile=./controller01.jks `
	--from-file=passwordfile=./password 
 secret/controller-tls created
	
# Verify the secrets
> kubectl get secret controller-tls
> kubectl describe secret controller-tls

kubectl create secret generic segmentstore-tls `
      --from-file=segmentstoreTlsCertFile=./segmentstore01.pem `
      --from-file=segmentstoreCacert=./ca-cert `
      --from-file=segmentstoreTlsKeyFile=./segmentstore01.key.pem

> kubectl get secret segmentstore-tls
> kubectl describe secret segmentstore-tls
```

Deleting a secrets:

```
kubectl delete secret pravega-pki
kubectl delete secret controller-tls
```

3. Deploy Pravega:

(I use a a yaml file that looks like the one [here](https://github.com/ravisharda/pravega-developer-resources/blob/master/deployments/yaml-examples/k8s-external-access-tls-auth.md)

```
pravega-operator> kubectl create -f pravega.yaml

# Verify that the cluster instances and its components are being created.
pravega-operator> kubectl get PravegaCluster
pravega-operator> kubectl get all -l pravega_cluster
```

### Step 6: Using the Pravega cluster

A PravegaCluster instance which doesn't have the external access enabled, is only accessible within the cluster (i.e. no outside access is allowed) using the following endpoint in the PravegaClient.
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

#### Port forwarding to access the cluster

For debugging and development you might want to access the Pravega cluster directly. For example, if you created the cluster with name pravega in the default namespace you can forward ports of the Pravega controller pod with name pravega-pravega-controller-68657d67cd-w5x8b as shown below. Do keep in mind though that you might really be able to access only the controller (an dnot the segment stores). So, it is useful only for using the Controller REST endpoint. 

```
kubectl port-forward -n default pravega-pravega-controller-68657d67cd-w5x8b 9090:9090 10080:10080
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
    
    # Watching
    > kubectl get pods -w
    
    # Sort by status
    > kubectl get pods --sort-by=.status.phase
    
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
