# Sample VM monitoring solution

A sample VM solution that shows how to deploy a workbook and create a log alert.

[![Deploy to Azure](http://azuredeploy.net/deploybutton.png)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Facearun%2Fmanagedsolutions%2Fmaster%2FSolutions%FVmMonitoring%2Fazuredeploy.json)

[More info on deploying](../deploy.md)

## Details
A solution packages metadata about the templates it exports in the outputs section of its `mainTemplate.json`. Here is a template metadata that this solution exports:

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
                        "id": "https://arunamanagedappstorage.blob.core.windows.net/managedsolutions/ResourceMonitor.workbook",
                        "source": "url",
                        "galleries": [
                            {
                                "name": "Resource monitor",
                                "category": "Virtual machine monitoring solution",
                                "order": 100,
                                "type": "workbook",
                                "resourceType": "Azure Monitor"
                            }
                        ],
                        "localized": {}
                    },
                    {
                        "id": "https://arunamanagedappstorage.blob.core.windows.net/managedsolutions/ResourceMonitor.workbook",
                        "source": "url",
                        "galleries": [
                            {
                                "name": "Virtual machine monitor",
                                "category": "Virtual machine monitoring solution",
                                "order": 100,
                                "type": "m-insights",
                                "resourceType": "microsoft.compute/virtualmachines"
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