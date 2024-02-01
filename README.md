
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

- create a ingress.yaml file with the following content (**modify the <path> to use a unique value (not already used on the ingress controller)**)

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

- Generate helm chart
```shell
helm create azure-vote-front
```

- Edit azure-vote-front/chart.yaml to add dependency for redis, and update appVersion to v1
```yaml
apiVersion: v2
name: azure-vote-front
description: A Helm chart for Kubernetes

dependencies:
  - name: redis
    version: 17.3.17
    repository: https://charts.bitnami.com/bitnami

# A chart can be either an 'application' or a 'library' chart.
#
# Application charts are a collection of templates that can be packaged into versioned archives
# to be deployed.
#
# Library charts provide useful utilities or functions for the chart developer. They're included as
# a dependency of application charts to inject those utilities and functions into the rendering
# pipeline. Library charts do not define any templates and therefore cannot be deployed.
type: application

# This is the chart version. This version number should be incremented each time you make changes
# to the chart and its templates, including the app version.
# Versions are expected to follow Semantic Versioning (https://semver.org/)
version: 0.1.0

# This is the version number of the application being deployed. This version number should be
# incremented each time you make changes to the application. Versions are not expected to
# follow Semantic Versioning. They should reflect the version the application is using.
# It is recommended to use it with quotes.
appVersion: "v1"
```

- update dependency
```shell
helm dependency update azure-vote-front
```

- Edit azure-vote-front/values.yaml with following changes:
  - Add a redis section to set the image details, container port, and deployment name.
  - Add a backendName for connecting the frontend portion to the redis deployment.
  - Change image.repository to <loginServer>/azure-vote-front.
  - Change image.tag to v1.
  - Change service.type to LoadBalancer

```yaml
# Default values for azure-vote-front.
# This is a YAML-formatted file.
# Declare variables to be passed into your templates.

replicaCount: 1

backendName: azure-vote-backend-master
redis:
  image:
    registry: mcr.microsoft.com
    repository: oss/bitnami/redis
    tag: 6.0.8
  fullnameOverride: azure-vote-backend
  auth:
    enabled: false

image:
  repository: acrakscodingdojo.azurecr.io/azure-vote-front
  pullPolicy: IfNotPresent
  tag: "v1"

imagePullSecrets: []
nameOverride: ""
fullnameOverride: ""

serviceAccount:
  # Specifies whether a service account should be created
  create: true
  # Automatically mount a ServiceAccount's API credentials?
  automount: true
  # Annotations to add to the service account
  annotations: {}
  # The name of the service account to use.
  # If not set and create is true, a name is generated using the fullname template
  name: ""

podAnnotations: {}
podLabels: {}

podSecurityContext: {}
  # fsGroup: 2000

securityContext: {}
  # capabilities:
  #   drop:
  #   - ALL
  # readOnlyRootFilesystem: true
  # runAsNonRoot: true
  # runAsUser: 1000

service:
  type: LoadBalancer
  port: 80

ingress:
  enabled: false
  className: ""
  annotations: {}
    # kubernetes.io/ingress.class: nginx
    # kubernetes.io/tls-acme: "true"
  hosts:
    - host: chart-example.local
      paths:
        - path: /
          pathType: ImplementationSpecific
  tls: []
  #  - secretName: chart-example-tls
  #    hosts:
  #      - chart-example.local

resources: {}
  # We usually recommend not to specify default resources and to leave this as a conscious
  # choice for the user. This also increases chances charts run on environments with little
  # resources, such as Minikube. If you do want to specify resources, uncomment the following
  # lines, adjust them as necessary, and remove the curly braces after 'resources:'.
  # limits:
  #   cpu: 100m
  #   memory: 128Mi
  # requests:
  #   cpu: 100m
  #   memory: 128Mi

autoscaling:
  enabled: false
  minReplicas: 1
  maxReplicas: 100
  targetCPUUtilizationPercentage: 80
  # targetMemoryUtilizationPercentage: 80

# Additional volumes on the output Deployment definition.
volumes: []
# - name: foo
#   secret:
#     secretName: mysecret
#     optional: false

# Additional volumeMounts on the output Deployment definition.
volumeMounts: []
# - name: foo
#   mountPath: "/etc/foo"
#   readOnly: true

nodeSelector: {}

tolerations: []

affinity: {}
```

- Edit azure-vote-front/templates/deployment.yaml to add an env section to pass the name of the redis deployment
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "azure-vote-front.fullname" . }}
  labels:
    {{- include "azure-vote-front.labels" . | nindent 4 }}
spec:
  {{- if not .Values.autoscaling.enabled }}
  replicas: {{ .Values.replicaCount }}
  {{- end }}
  selector:
    matchLabels:
      {{- include "azure-vote-front.selectorLabels" . | nindent 6 }}
  template:
    metadata:
      {{- with .Values.podAnnotations }}
      annotations:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      labels:
        {{- include "azure-vote-front.labels" . | nindent 8 }}
	{{- with .Values.podLabels }}
        {{- toYaml . | nindent 8 }}
        {{- end }}
    spec:
      {{- with .Values.imagePullSecrets }}
      imagePullSecrets:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      serviceAccountName: {{ include "azure-vote-front.serviceAccountName" . }}
      securityContext:
        {{- toYaml .Values.podSecurityContext | nindent 8 }}
      containers:
        - name: {{ .Chart.Name }}
          securityContext:
            {{- toYaml .Values.securityContext | nindent 12 }}
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag | default .Chart.AppVersion }}"
          imagePullPolicy: {{ .Values.image.pullPolicy }}
          env:
          - name: REDIS
            value: {{ .Values.backendName }}
          ports:
            - name: http
              containerPort: {{ .Values.service.port }}
              protocol: TCP
          livenessProbe:
            httpGet:
              path: /
              port: http
          readinessProbe:
            httpGet:
              path: /
              port: http
          resources:
            {{- toYaml .Values.resources | nindent 12 }}
          {{- with .Values.volumeMounts }}
          volumeMounts:
            {{- toYaml . | nindent 12 }}
          {{- end }}
      {{- with .Values.volumes }}
      volumes:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      {{- with .Values.nodeSelector }}
      nodeSelector:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      {{- with .Values.affinity }}
      affinity:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      {{- with .Values.tolerations }}
      tolerations:
        {{- toYaml . | nindent 8 }}
      {{- end }}
```

- Install the helm chart
```shell
helm install azure-vote-front azure-vote-front/
```

- Get the public IP address of the load balancer
```shell
kubectl get service azure-vote-front
```

- Navigate to the application using the EXTERNAL-IP

- view installed helm charts
```shell
helm ls
```

- uninstall helm chart
```shell
helm uninstall azure-vote-front
```


# References

- Ingress controllers

https://learn.microsoft.com/en-us/azure/aks/app-routing?tabs=default%2Cdeploy-app-default
https://learn.microsoft.com/en-us/azure/aks/ingress-basic?tabs=azure-cli
https://learn.microsoft.com/en-us/azure/application-gateway/ingress-controller-overview

- Helm tutorial

https://learn.microsoft.com/en-us/azure/aks/quickstart-helm?tabs=azure-cli

- Multi-container app

https://github.com/Azure-Samples/aks-store-demo

- Deploying ASP.NET Core applications to Kubernetes

https://andrewlock.net/series/deploying-asp-net-core-applications-to-kubernetes/


