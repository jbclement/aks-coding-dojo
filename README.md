
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

- Expose the service with the ingress controller (**modify the <path> to use a unique value (not already used on the ingress controller)**)

```shell
kubectl create ingress hello-ingress --class=nginx --rule="/<path>*=hello-service:80" --annotation="nginx.ingress.kubernetes.io/rewrite-target=/"
```

- View ingresses in the namespace
```shell
kubectl get ingress

NAME            CLASS   HOSTS   ADDRESS        PORTS   AGE
hello-ingress   nginx   *       4.208.77.173   80      6m8s
```

Navigate to the site http://4.208.77.173/yourpath

- Describe pod / service / ingress
```shell
kubectl describe pod hello-pod
kubectl describe service hello-service
kubectl describe ingress hello-ingress
```

- View pod logs
```shell
kubectl logs hello-pod
```

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

NAME            CLASS   HOSTS   ADDRESS        PORTS   AGE
hello-ingress   nginx   *       4.208.77.173   80      6m8s
```

Navigate to the site http://4.208.77.173/yourpath

# Deploy a multi-container app with yaml

- create a file aks-contoso-store.yaml with the following content:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: rabbitmq
spec:
  replicas: 1
  selector:
    matchLabels:
      app: rabbitmq
  template:
    metadata:
      labels:
        app: rabbitmq
    spec:
      nodeSelector:
        "kubernetes.io/os": linux
      containers:
      - name: rabbitmq
        image: mcr.microsoft.com/mirror/docker/library/rabbitmq:3.10-management-alpine
        ports:
        - containerPort: 5672
          name: rabbitmq-amqp
        - containerPort: 15672
          name: rabbitmq-http
        env:
        - name: RABBITMQ_DEFAULT_USER
          value: "username"
        - name: RABBITMQ_DEFAULT_PASS
          value: "password"
        resources:
          requests:
            cpu: 10m
            memory: 128Mi
          limits:
            cpu: 250m
            memory: 256Mi
        volumeMounts:
        - name: rabbitmq-enabled-plugins
          mountPath: /etc/rabbitmq/enabled_plugins
          subPath: enabled_plugins
      volumes:
      - name: rabbitmq-enabled-plugins
        configMap:
          name: rabbitmq-enabled-plugins
          items:
          - key: rabbitmq_enabled_plugins
            path: enabled_plugins
---
apiVersion: v1
data:
  rabbitmq_enabled_plugins: |
    [rabbitmq_management,rabbitmq_prometheus,rabbitmq_amqp1_0].
kind: ConfigMap
metadata:
  name: rabbitmq-enabled-plugins
---
apiVersion: v1
kind: Service
metadata:
  name: rabbitmq
spec:
  selector:
    app: rabbitmq
  ports:
    - name: rabbitmq-amqp
      port: 5672
      targetPort: 5672
    - name: rabbitmq-http
      port: 15672
      targetPort: 15672
  type: ClusterIP
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: order-service
spec:
  replicas: 1
  selector:
    matchLabels:
      app: order-service
  template:
    metadata:
      labels:
        app: order-service
    spec:
      nodeSelector:
        "kubernetes.io/os": linux
      containers:
      - name: order-service
        image: ghcr.io/azure-samples/aks-store-demo/order-service:latest
        ports:
        - containerPort: 3000
        env:
        - name: ORDER_QUEUE_HOSTNAME
          value: "rabbitmq"
        - name: ORDER_QUEUE_PORT
          value: "5672"
        - name: ORDER_QUEUE_USERNAME
          value: "username"
        - name: ORDER_QUEUE_PASSWORD
          value: "password"
        - name: ORDER_QUEUE_NAME
          value: "orders"
        - name: FASTIFY_ADDRESS
          value: "0.0.0.0"
        resources:
          requests:
            cpu: 1m
            memory: 50Mi
          limits:
            cpu: 75m
            memory: 128Mi
      initContainers:
      - name: wait-for-rabbitmq
        image: busybox
        command: ['sh', '-c', 'until nc -zv rabbitmq 5672; do echo waiting for rabbitmq; sleep 2; done;']
        resources:
          requests:
            cpu: 1m
            memory: 50Mi
          limits:
            cpu: 75m
            memory: 128Mi
---
apiVersion: v1
kind: Service
metadata:
  name: order-service
spec:
  type: ClusterIP
  ports:
  - name: http
    port: 3000
    targetPort: 3000
  selector:
    app: order-service
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: product-service
spec:
  replicas: 1
  selector:
    matchLabels:
      app: product-service
  template:
    metadata:
      labels:
        app: product-service
    spec:
      nodeSelector:
        "kubernetes.io/os": linux
      containers:
      - name: product-service
        image: ghcr.io/azure-samples/aks-store-demo/product-service:latest
        ports:
        - containerPort: 3002
        resources:
          requests:
            cpu: 1m
            memory: 1Mi
          limits:
            cpu: 1m
            memory: 7Mi
---
apiVersion: v1
kind: Service
metadata:
  name: product-service
spec:
  type: ClusterIP
  ports:
  - name: http
    port: 3002
    targetPort: 3002
  selector:
    app: product-service
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: store-front
spec:
  replicas: 1
  selector:
    matchLabels:
      app: store-front
  template:
    metadata:
      labels:
        app: store-front
    spec:
      nodeSelector:
        "kubernetes.io/os": linux
      containers:
      - name: store-front
        image: ghcr.io/azure-samples/aks-store-demo/store-front:latest
        ports:
        - containerPort: 8080
          name: store-front
        env:
        - name: VUE_APP_ORDER_SERVICE_URL
          value: "http://order-service:3000/"
        - name: VUE_APP_PRODUCT_SERVICE_URL
          value: "http://product-service:3002/"
        resources:
          requests:
            cpu: 1m
            memory: 200Mi
          limits:
            cpu: 1000m
            memory: 512Mi
---
apiVersion: v1
kind: Service
metadata:
  name: store-front
spec:
  ports:
  - port: 80
    targetPort: 8080
  selector:
    app: store-front
  type: LoadBalancer
```

- apply yaml file:
```shell
kubectl apply -f aks-contoso-store.yaml
```

- delete all deployments and services
```shell
kubectl delete --all deployments
kubectl delete --all services
```


# Deploy an app with Helm

https://learn.microsoft.com/en-us/azure/aks/quickstart-helm?tabs=azure-cli


# References

- Ingress controllers
https://learn.microsoft.com/en-us/azure/aks/app-routing?tabs=default%2Cdeploy-app-default
https://learn.microsoft.com/en-us/azure/aks/ingress-basic?tabs=azure-cli
https://learn.microsoft.com/en-us/azure/application-gateway/ingress-controller-overview



