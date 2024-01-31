# aks-coding-dojo

## Environment setup

- Set environment variables

```shell
RESOURCE_GROUP_NAME="aks-coding-dojo-rg"
LOCATION="northeurope"
AKS_NAME="aks-coding-dojo"
ACR_NAME="acrakscodingdojo"
AKS_ADMIN_GROUP="aks-coding-dojo-admins"
CODING_DOJO_STUDENTS_GROUP="aks-coding-dojo-students"
```

- Create resource group
```shell
az group create --name $RESOURCE_GROUP_NAME --location $LOCATION
```

- create Entra ID group for students
```shell
az ad group create --display-name $CODING_DOJO_STUDENTS_GROUP --mail-nickname $CODING_DOJO_STUDENTS_GROUP
```

- Add list of members in the sudents group based on the content of students.txt
```shell
while read line; do    
    USER_ID=$({ az ad user list --filter "mail eq '$line'" --query "[].id" -o tsv | tr -d '\r' | tr -d '\n'; } < /dev/null)
    echo "add $USER_ID to group $CODING_DOJO_STUDENTS_GROUP"
    { az ad group member add --group $CODING_DOJO_STUDENTS_GROUP --member-id $USER_ID ; } < /dev/null
done < students.txt
```

- create Entra ID group for AKS admins
```shell
az ad group create --display-name $AKS_ADMIN_GROUP --mail-nickname $AKS_ADMIN_GROUP
```

- Add list of members in the admins group based on the content of admins.txt
```shell
while read line; do
    USER_ID=$({ az ad user list --filter "mail eq '$line'" --query "[].id" -o tsv | tr -d '\r' | tr -d '\n'; } < /dev/null)
    echo "add $USER_ID to group $AKS_ADMIN_GROUP"
    { az ad group member add --group $AKS_ADMIN_GROUP --member-id $USER_ID ; } < /dev/null
done < admins.txt
```

-- Get the id of the admins group
```shell
AKS_ADMIN_GROUP_ID=$(az ad group show --group $AKS_ADMIN_GROUP --query "id" -o tsv | tr -d '\r' | tr -d '\n')
```

-- create Azure container registry
```shell
az acr create --resource-group $RESOURCE_GROUP_NAME --name $ACR_NAME --sku Basic
```

-- create the AKS cluster
```shell
az aks create -g $RESOURCE_GROUP_NAME -n $AKS_NAME \
--enable-aad \
--aad-admin-group-object-ids $AKS_ADMIN_GROUP_ID \
--enable-azure-rbac \
--node-count 1 \
--node-vm-size Standard_B2ms
--attach-acr $ACR_NAME
```

-- Assign role "Azure Kubernetes Service Cluster User Role" and "Azure Kubernetes Service RBAC Reader" to the students group on the AKS cluster created above
```shell
AKS_ID=$(az aks show -g $RESOURCE_GROUP_NAME -n $AKS_NAME --query id -o tsv | tr -d '\r' | tr -d '\n')
CODING_DOJO_STUDENTS_GROUP_ID=$(az ad group show --group $CODING_DOJO_STUDENTS_GROUP --query "id" -o tsv | tr -d '\r' | tr -d '\n')
az role assignment create --role "Azure Kubernetes Service Cluster User Role" --assignee $CODING_DOJO_STUDENTS_GROUP_ID  --scope $AKS_ID
az role assignment create --role "Azure Kubernetes Service RBAC Reader" --assignee $CODING_DOJO_STUDENTS_GROUP_ID  --scope $AKS_ID
```

-- Get kubeconfig file
```shell
az aks get-credentials -n $AKS_NAME -g $RESOURCE_GROUP_NAME --overwrite-existing
kubectl config set-context $AKS_NAME
```

-- Create namespace in the AKS cluster for each student and assign role "Azure Kubernetes Service RBAC Writer" to the student
```shell
while read line; do    
    UPN=$({ az ad user list --filter "mail eq '$line'" --query "[].userPrincipalName" -o tsv | tr -d '\r' | tr -d '\n'; } < /dev/null)
    NS=$(echo "$UPN" | awk -F"@" '{print $1}' | tr -d '#EXT#' | tr -d '.' | tr -d '_' | sed 's/avanadecom//')
    echo $NS
    { kubectl create ns $NS ; } < /dev/null
    USER_ID=$({ az ad user list --filter "mail eq '$line'" --query "[].id" -o tsv | tr -d '\r' | tr -d '\n'; } < /dev/null)
    { az role assignment create --role "Azure Kubernetes Service RBAC Writer" --assignee $USER_ID --scope $AKS_ID/namespaces/$NS ; } < /dev/null
done < students.txt    
```

-- Create ingress controller
```shell
NAMESPACE=ingress-basic

helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update

helm install ingress-nginx ingress-nginx/ingress-nginx \
  --create-namespace \
  --namespace $NAMESPACE \
  --set controller.service.annotations."service\.beta\.kubernetes\.io/azure-load-balancer-health-probe-request-path"=/healthz \
  --set controller.service.externalTrafficPolicy=Local
```