# .NET & Windows Commands

## Powershell


## CMD

## Package managers

* See https://github.com/chocolatey/choco/wiki/CommandsList for Chocolatey package manager commands. 

## Deploying a Pravega Kubernetes Cluster

## Step 1: Install Zookeeper using Pravega Zookeeper Operator

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

See the steps here: https://github.com/pravega/pravega-operator#install-the-operator

### Step 4: Deploy a Pravega Cluster

This involves two sub-steps:
* Provisioning an NFS volume provisioned by the NFS Server Provisioner helm chart to provide Tier 2 storage. 
* Creating a Pravega cluster

See the steps here: https://github.com/pravega/pravega-operator#deploy-a-sample-pravega-cluster



