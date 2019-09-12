# Sample VM monitoring solution

A sample VM solution that shows how to deploy a workbook and create a log alert.

[![Deploy to Azure](http://azuredeploy.net/deploybutton.png)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Facearun%2Fmanagedsolutions%2Fmaster%2FSolutions%2FVmMonitoring-test%2Fazuredeploy.json)

[More info on deploying](../deploy.md)

## Details
This solution sets up the environment with two things - a scheduled query alert and a workbook for analysis.

### Workbook
The solution creates a workbook called _Virtual machine monitor_ in Azure Monitor > Workbook gallery. This allows a user to analyze the VMs interactively using workbooks. A solution packages metadata about the templates it exports in the outputs section of its `mainTemplate.json`. Here is a template metadata that this solution exports:

```json
{

    "outputs": {

        "templateMetadata": {
            "type": "object",
            "value": {
                "customRpId": "[variables('customRpId')]",
                "solutionName": "[variables('solutionDisplayName')]",
                "templates": [
                    {
                        "id": "https://arunamanagedappstorage.blob.core.windows.net/managedsolutions/ResourceMonitor2.workbook",
                        "source": "url",
                        "galleries": [
                            {
                                "name": "Virtual machine monitor",
                                "category": "Virtual machine monitoring solution",
                                "order": 100,
                                "type": "workbook",
                                "resourceType": "Azure Monitor"
                            }
                        ],
                        "localized": {}
                    }    
                ]
            }
        }
    }
}
```

### Scheduled query
The solution also creates a scheduled query alert against a log analytics workspace specified by the user. The information of the workspace to target and the action group to gather from the user is defined in the _createUiDefinition.json_ file

```json
{
    "$schema": "https://schema.management.azure.com/schemas/0.1.2-preview/CreateUIDefinition.MultiVm.json#",
    "handler": "Microsoft.Azure.CreateUIDef",
    "version": "0.1.2-preview",
    "parameters": {
        "basics": [
        ],
        "steps": [
            {
                "name": "alertConfig",
                "label": "Alert settings",
                "elements": [
                    {
                        "name": "workspaceId",
						"type": "Microsoft.Common.TextBox",
						"label": "Workspace resource id",
						"defaultValue": "",
						"toolTip": "Fully qualified resource id of the backing workspace",
						"visible": true
											
					},
                    {
                        "name": "actionGroupId",
						"type": "Microsoft.Common.TextBox",
						"label": "Action group resource id",
						"defaultValue": "",
						"toolTip": "Fully qualified resource id of the action group to use for the created alerts",
						"visible": true											
                    },
                    {
                        "name": "alertPrefix",
						"type": "Microsoft.Common.TextBox",
						"label": "Alert name prefix",
						"defaultValue": "AME_KMC_",
						"toolTip": "The prefix to for the created alerts",
						"visible": true											
					}                    
                ]
            }
        ],
        "outputs": {
            "workspaceId": "[steps('alertConfig').workspaceId]",
            "actionGroupId": "[steps('alertConfig').actionGroupId]",
            "alertPrefix": "[steps('alertConfig').alertPrefix]",
        }
    }
}
```
