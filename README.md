
    █████╗ ██╗  ██╗███████╗     ██████╗ ██████╗ ██████╗ ██╗███╗   ██╗ ██████╗     
    ██╔══██╗██║ ██╔╝██╔════╝    ██╔════╝██╔═══██╗██╔══██╗██║████╗  ██║██╔════╝     
    ███████║█████╔╝ ███████╗    ██║     ██║   ██║██║  ██║██║██╔██╗ ██║██║  ███╗    
    ██╔══██║██╔═██╗ ╚════██║    ██║     ██║   ██║██║  ██║██║██║╚██╗██║██║   ██║    
    ██║  ██║██║  ██╗███████║    ╚██████╗╚██████╔╝██████╔╝██║██║ ╚████║╚██████╔╝    
    ╚═╝  ╚═╝╚═╝  ╚═╝╚══════╝     ╚═════╝ ╚═════╝ ╚═════╝ ╚═╝╚═╝  ╚═══╝ ╚═════╝     
                                                                                
    ██████╗  ██████╗      ██╗ ██████╗                                              
    ██╔══██╗██╔═══██╗     ██║██╔═══██╗                                             
    ██║  ██║██║   ██║     ██║██║   ██║                                             
    ██║  ██║██║   ██║██   ██║██║   ██║                                             
    ██████╔╝╚██████╔╝╚█████╔╝╚██████╔╝                                             
    ╚═════╝  ╚═════╝  ╚════╝  ╚═════╝                                              
                                                                                

# Objectives

- get familiar with AKS environment
- explore command line tools
- deploy applications through yaml files
- deploy applications through helm package
- explore additional tools
                                                                                                                    
# client tools installation

To follow this coding dojo, some tools are required.
You can use either Powershell console, WSL or Azure cloud shell.

To install tools using Powershell console, follow these steps:

- install chocolatey

in a Powershell admin console, follow the installation procedure: https://chocolatey.org/install

- install azure cli

Still in a Powershell admin console, execute:
```shell
choco install -y azure-cli ; choco install -y kubernetes-cli ; choco install -y kubernetes-helm ; choco install -y k9s
```

# AKS from Azure Portal



# Managing Kubernetes Objects Using Imperative Commands

Reference : https://kubernetes.io/docs/tasks/manage-kubernetes-objects/imperative-command/

- list all namespaces and identify yours

```shell
kubectl get namespaces
```

- list pods in your namespace
```shell
kubectl get pods -n <mynamespace>
```

- set your namespace as default in the current context
```shell
kubectl config set-context --current --namespace=<mynamespace>
```

create pod
```shell
kubectl run hello-pod --image=jbclementdocker/aks-coding-dojo:hello --port=80
```

create service

kubectl expose pod hello-pod2 --name=mon-service --port=80 --target-port=80 --type=ClusterIP

create ingress
kubectl create ingress ingress3 --class=nginx   --rule="/aa*=mon-service:80"   --annotation="nginx.ingress.kubernetes.io/rewrite-target=/"

describe commands

logs commands

delete commands

# Deploy with yaml files

https://learn.microsoft.com/en-us/azure/aks/app-routing?tabs=default%2Cdeploy-app-default
https://learn.microsoft.com/en-us/azure/aks/ingress-basic?tabs=azure-cli


# Deploy a multi-container app with yaml

```shell
kubectl apply -f ./yaml-contoso-store/app.yaml
```

# Deploy an app with Helm

https://learn.microsoft.com/en-us/azure/aks/quickstart-helm?tabs=azure-cli


