
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

# Explore AKS from Azure Portal

- connect to https://portal.azure.com
- search for aks-coding-dojo in the search bar and go to the AKS cluster

# Managing Kubernetes objects

## Managing Kubernetes Objects Using Imperative Commands

Reference : https://kubernetes.io/docs/tasks/manage-kubernetes-objects/imperative-command/

In your Powershell command line (or WSL, or Azure cloud shell...):
- connect to the cluster and generate a kubeconfig file:

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

- create pod
```shell
kubectl run hello-pod --image=jbclementdocker/aks-coding-dojo:hello --port=80
```

- create a ClusterIP service for the pod created
```shell
kubectl expose pod hello-pod --name=hello-service --port=80 --target-port=80 --type=ClusterIP
```

- Expose the service with the ingress controller

** modify the <path> to use a unique value (not already used on the ingress controller) **

```shell
kubectl create ingress hello-ingress --class=nginx --rule="/<path>*=hello-service:80" --annotation="nginx.ingress.kubernetes.io/rewrite-target=/"
```

- View ingresses in the namespace
```shell
kubectl get ingress
```

Navigate to the site http://<IP>/<path>

- Describe pod / service / ingress
```shell
kubectl describe pod hello-pod
kubectl describe service hello-service
kubectl describe ingress hello-ingress
```

- View pod logs

- Delete pod / service / ingress
```shell
kubectl delete pod hello-pod
kubectl delete service hello-service
kubectl delete ingress hello-ingress
```

## Managing Kubernetes Objects Using YAML files

### Create a deployment

- create a deployment.yaml file with the following content
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-deployment
  labels:
    app: hello
spec:
  replicas: 3
  strategy: 
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
  selector:
    matchLabels:
      app: hello-app
  template:
    metadata:
      labels:
        app: hello-app
    spec:
      containers:
      - name: hello-pod
        image: jbclementdocker/aks-coding-dojo:hello
        ports:
        - containerPort: 80
```

- apply the yaml description
```shell
kubectl apply -f deployment.yaml
```

### Create a service

- create a service.yaml file with the following content
```yaml
apiVersion: v1
kind: Service
metadata:
  name: hello-service
  labels: 
    app: hello-svc

spec:
  type: ClusterIP
  selector:
    app: hello-app

  ports:
  - port: 80
    targetPort: 80
    protocol: TCP
```

### Create ingress

- create a ingress.yaml file with the following content

** modify the <path> to use a unique value (not already used on the ingress controller) **

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: hello-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
    - http:
        paths:
          - path: /<path>
            pathType: Prefix
            backend:
              service:
                name: hello-service
                port:
                  number: 80            
```

- View ingresses in the namespace
```shell
kubectl get ingress
```

Navigate to the site http://<IP>/<path>

# 36 15 Kubernetes : explore the cluster with k9s



https://learn.microsoft.com/en-us/azure/aks/app-routing?tabs=default%2Cdeploy-app-default
https://learn.microsoft.com/en-us/azure/aks/ingress-basic?tabs=azure-cli


# Deploy a multi-container app with yaml

```shell
kubectl apply -f ./yaml-contoso-store/app.yaml
```

# Deploy an app with Helm

https://learn.microsoft.com/en-us/azure/aks/quickstart-helm?tabs=azure-cli


