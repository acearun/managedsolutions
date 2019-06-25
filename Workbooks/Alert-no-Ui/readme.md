# Publishing an Azure managed application definition

## Option 1 - Deploy using Azure Portal
### Step 1 - Get authorization info
Run these command in the Azure Cloud shell and copy the auth info:
```bash
> userid=$(az ad user show --upn-or-object-id example@contoso.org --query objectId --output tsv);roleid=$(az role definition list --name Owner --query [].name --output tsv);echo [{\"principalId\":\"$userid\", \"roleDefinitionId\":\"$roleid\" }]
```
💡 Make sure you change example@contoso.org to your email.

Copy the value of returned by the last command to use in the `authorizations` parameter of the next step.

### Step 2 - Deploy
Clicking on the button below, will create the Managed Application definition to a Resource Group in your Azure subscription.

[![Deploy to Azure](http://azuredeploy.net/deploybutton.png)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Facearun%2Fmanagedsolutions%2Fmaster%2FWorkbooks%Alert-no-Ui%2Fazuredeploy.json)

## Option 2 - Deploy Using the Azure Cloud Shell
### Step 1 - Open Cloud Shell
Select the Cloud Shell button on the menu on the upper-right corner of the [Azure portal](https://portal.azure.com).

![Image showing the cloud shell ](https://docs.microsoft.com/en-us/azure/includes/media/cloud-shell-try-it/cloud-shell-menu.png)

### Step 2 - Create a resource group for definition

```bash
az group create --name appDefinitionGroup --location westcentralus
```

### Step 3 - Get User principal
```bash
userid=$(az ad user show --upn-or-object-id example@contoso.org --query objectId --output tsv)
```
### Step 4 - Get Role Id
```bash
roleid=$(az role definition list --name Owner --query [].name --output tsv)
```

### Step 4 - Create the managed app definition
```bash
az managedapp definition create \
  --name "ManagedWorkbooks" \
  --location "westcentralus" \
  --resource-group appDefinitionGroup \
  --lock-level ReadOnly \
  --display-name "Managed Workbooks" \
  --description "Managed Azure Workbooks" \
  --authorizations "$userid:$roleid" \
  --package-file-uri "http://raw.githubusercontent.com/acearun/managedsolutions/master/Workbooks/Alert-no-Ui/Deploy.zip"
  ```


## Deploying the managed app
Either way you choose to deploy the managed app definition, the next step usually is to create a deployment of the app:
1. Use the deploy option in the Managed App definition.
2. Choose the sub and resource group in the deployment workflow
3. Deploy

