# Publishing an Azure managed application defination

## Open Azure Cloud Shell
Select the Cloud Shell button on the menu on the upper-right corner of the [Azure portal](https://portal.azure.com).

![Image showing the cloud shell ](https://docs.microsoft.com/en-us/azure/includes/media/cloud-shell-try-it/cloud-shell-menu.png)

## Create a resource group for definition

```bash
az group create --name appDefinitionGroup --location westcentralus
```

## Assign permissions
### User principal
```bash
userid=$(az ad user show --upn-or-object-id example@contoso.org --query objectId --output tsv)
```
### Role definition
```bash
roleid=$(az role definition list --name Owner --query [].name --output tsv)
```

## Create the managed app
az managedapp definition create \
  --name "ManagedWorkbooks" \
  --location "westcentralus" \
  --resource-group appDefinitionGroup \
  --lock-level ReadOnly \
  --display-name "Managed Workbooks" \
  --description "Managed Azure Workbooks" \
  --authorizations "$userid:$roleid" \
  --package-file-uri "http://raw.githubusercontent.com/acearun/managedsolutions/master/Workbooks/Test1/test1.zip"