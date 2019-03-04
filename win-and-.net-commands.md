# .NET & Windows Commands

## Powershell


## CMD

## Package managers

* See https://github.com/chocolatey/choco/wiki/CommandsList for Chocolatey package manager commands. 

### Installing and getting Helm to work

* Install Chocolatey
* Install Helm using Chocolatey: ``choco install kubernetes-helm``
* Initialize Helm: ``helm init``
* Now, add service accounts: 
  
  ```
  kubectl create serviceaccount --namespace kube-system tiller
  
  kubectl create clusterrolebinding tiller-cluster-rule --clusterrole=cluster-admin --serviceaccount=kube-system:tiller
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
