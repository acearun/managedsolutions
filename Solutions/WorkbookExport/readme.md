# Solution that exports workbook templates

This sample shows how to package a workbook template in a solution and export it to a resource.

[![Deploy to Azure](http://azuredeploy.net/deploybutton.png)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Facearun%2Fmanagedsolutions%2Fmaster%2FSolutions%2FWorkbookExport%2Fazuredeploy.json)

## Details
A solution packages metadata about the templates it exports in the outputs section of its `mainTemplate.json`. Here is a template metadata that this solution exports:

```json
{
    ...
    "outputs": {
        ...
        "templateMetadata": {
            "type": "object",
            "value": {
                "customRpId": "[variables('customRpId')]",
                "solutionName": "[variables('solutionDisplayName')]",
                "templates": [
                    {
                        "id": "https://arunamanagedappstorage.blob.core.windows.net/managedsolutions/FailureAnalysis.workbook",
                        "source": "url",
                        "galleries": [
                            {
                                "name": "App Failures - Acme Solution",
                                "category": "Acme analytics",
                                "order": 100,
                                "type": "tsg",
                                "resourceType": "microsoft.insights/components"
                            }
                        ],
                        "localized": {
                            "es-es": {
                                "id": "https://arunamanagedappstorage.blob.core.windows.net/managedsolutions/FailureAnalysis.workbook",
                                "galleries": [
                                    {
                                        "name": "App Fallos - Acme Solution",
                                        "order": 100,
                                        "category": "Acme análisis",
                                        "type": "tsg",
                                        "resourceType": "microsoft.insights/components"
                                    }
                                ]
                            }
                        }
                    }
                ]
            }
        }
    }
}
```