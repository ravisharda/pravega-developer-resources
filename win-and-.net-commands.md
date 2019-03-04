# .NET & Windows Commands

## Powershell


## CMD

## Package managers

* See https://github.com/chocolatey/choco/wiki/CommandsList for Chocolatey package manager commands. 

## Installing and getting Helm to work

* Install Chocolatey
* Install Helm using Chocolatey: ``choco install kubernetes-helm``
* Initialize Helm: ``helm init``
* Now, add service accounts: 
  
  ```
  kubectl create serviceaccount --namespace kube-system tiller
  
  kubectl create clusterrolebinding tiller-cluster-rule --clusterrole=cluster-admin --serviceaccount=kube-system:tiller
  
  kubectl patch deploy --namespace kube-system tiller-deploy -p '{"spec":{"template":{"spec":{"serviceAccount":"tiller"}}}}'
  
  ```
