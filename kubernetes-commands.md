# Kubernetes (K8s) Commands and Utilities

(For K8s related [links](https://github.com/ravisharda/resource-center/blob/master/containers/kubernetes-links.md) @ [my resource center](https://github.com/ravisharda/resource-center/))

## Minikube related

* Version: `minikube version`
* Starting: ``minikube start``
  
  If you see an error ``Error with pre-create check: "This computer doesn't have VT-X/AMD-v enabled. Enabling it in the BIOS is mandatory"``, here's the fix: 
  * Power off the VM. 
  * ``VM-->Settings--->Processor-->Enable "Virtualize Intel VT-x/EPT "...``
  * Now, try again.
  
* Stopping: ``minikube stop``
* Minikube config
  * Note that Minikube configuration file is located under: ``~/.minikube/machines/minikube/config.json``
  * To view config: ``kubectl config view``

## Kubectl
* Version: ``kubectl version -o json``
* Checking cluster status: ``kubectl cluster-info``

