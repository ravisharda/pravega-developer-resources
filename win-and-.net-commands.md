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

See the steps here: https://github.com/pravega/pravega-operator#deploy-a-sample-pravega-cluster. When creating the Pravega cluster, you'll need the Zookeeper IP Address. Use this command to fetch it (extract the cluster IP address): `kubectl get all -l app=example`

This involves two sub-steps:
* Provisioning an NFS volume provisioned by the NFS Server Provisioner helm chart to provide Tier 2 storage. 
* Creating a Pravega cluster
* Verifying the cluster is running
  ```
  kubectl get PravegaCluster
  kubectl get all -l pravega_cluster=pravega
  ```


### Step 5: Use the Pravega cluster

See the steps here: https://github.com/pravega/pravega-operator#deploy-a-sample-pravega-cluster

A PravegaCluster instance is only accessible WITHIN the cluster (i.e. no outside access is allowed) using the following endpoint in the PravegaClient.
```
tcp://<cluster-name>-pravega-controller.<namespace>:9090
```

The REST management interface is available at:
```http://<cluster-name>-pravega-controller.<namespace>:10080/```

**To enable direct access to the cluster:**  

For debugging and development you might want to access the Pravega cluster directly. For example, if you created the cluster with name pravega in the default namespace you can forward ports of the Pravega controller pod with name pravega-pravega-controller-68657d67cd-w5x8b as follows:
```
kubectl port-forward -n default pravega-pravega-controller-68657d67cd-w5x8b 9090:9090 10080:10080
```



