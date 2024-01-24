# aks-coding-dojo

## Environment setup

```shell
RESOURCE_GROUP_NAME="aks-coding-dojo-rg"
LOCATION="northeurope"
AKS_NAME="aks-coding-dojo"
AKS_ADMIN_GROUP="aks-coding-dojo-admins"
CODING_DOJO_STUDENTS_GROUP="aks-coding-dojo-students"

az group create --name $RESOURCE_GROUP_NAME --location $LOCATION

az ad group create --display-name $CODING_DOJO_STUDENTS_GROUP --mail-nickname $CODING_DOJO_STUDENTS_GROUP

while read line; do    
    USER_ID=$({ az ad user list --filter "mail eq '$line'" --query "[].id" -o tsv | tr -d '\r' | tr -d '\n'; } < /dev/null)
    echo "add $USER_ID to group $CODING_DOJO_STUDENTS_GROUP"
    { az ad group member add --group $CODING_DOJO_STUDENTS_GROUP --member-id $USER_ID ; } < /dev/null
done < students.txt

az ad group create --display-name $AKS_ADMIN_GROUP --mail-nickname $AKS_ADMIN_GROUP

while read line; do
    USER_ID=$({ az ad user list --filter "mail eq '$line'" --query "[].id" -o tsv | tr -d '\r' | tr -d '\n'; } < /dev/null)
    echo "add $USER_ID to group $AKS_ADMIN_GROUP"
    { az ad group member add --group $AKS_ADMIN_GROUP --member-id $USER_ID ; } < /dev/null
done < admins.txt

AKS_ADMIN_GROUP_ID=$(az ad group show --group aks-admins --query "id" -o tsv | tr -d '\r' | tr -d '\n')

az aks create -g $RESOURCE_GROUP_NAME -n $AKS_NAME \
--enable-aad \
--aad-admin-group-object-ids $AKS_ADMIN_GROUP_ID \
--enable-azure-rbac \
--node-count 1 \
--node-vm-size Standard_B2ms
```

```shell

AKS_ID=$(az aks show -g $RESOURCE_GROUP_NAME -n $AKS_NAME --query id -o tsv | tr -d '\r' | tr -d '\n')
CODING_DOJO_STUDENTS_GROUP_ID=$(az ad group show --group $CODING_DOJO_STUDENTS_GROUP --query "id" -o tsv | tr -d '\r' | tr -d '\n')
az role assignment create --role "Azure Kubernetes Service Cluster User Role" --assignee $CODING_DOJO_STUDENTS_GROUP_ID  --scope $AKS_ID
```

```shell
az aks get-credentials -n $AKS_NAME -g $RESOURCE_GROUP_NAME
kubectl config set-context $AKS_NAME
```


```shell
while read line; do    
    UPN=$({ az ad user list --filter "mail eq '$line'" --query "[].userPrincipalName" -o tsv | tr -d '\r' | tr -d '\n'; } < /dev/null)
    NS=$(echo "$UPN" | awk -F"@" '{print $1}' | tr -d '#EXT#' | tr -d '.' | tr -d '_')
    echo $NS
    { kubectl create ns $NS ; } < /dev/null
    USER_ID=$({ az ad user list --filter "mail eq '$line'" --query "[].id" -o tsv | tr -d '\r' | tr -d '\n'; } < /dev/null)
    { az role assignment create --role "Azure Kubernetes Service RBAC Writer" --assignee $USER_ID --scope $AKS_ID/namespaces/$NS ; } < /dev/null
done < students.txt    
```
