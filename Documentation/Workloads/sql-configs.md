# SQL monitoring configuration

Azure Monitor for SQL provides you with a flexible way to specify the instances to monitor it's per-machine config. It really amounts down to specifying a collection of connection strings to the remote SQL instances to monitor and any secret/parameter required to do so.

Here is an example of a SQL monitoring config for a monitoring virtual machine:

```json
{
    "version": 1,
    "secrets": {
        "telegrafPassword": {
            "keyvault": "https://mykeyvault.vault.azure.net/",
            "name": "sqlPassword"
        }
    },
    "parameters": {
        "sqlAzureConnections": [
            "Server=mysqlserver1.database.windows.net;Port=1433;Database=mydatabase2;User Id=telegraf;Password=$telegrafPassword;",
            "Server=mysqlserver2.database.windows.net;Port=1433;Database=mydatabase2;User Id=telegraf;Password=$telegrafPassword;"
        ],
        "sqlVmConnections": [
        ],
        "sqlManagedInstanceConnections": [
        ]
    }
}
```

The config specifies 2 connection strings in the parameter `sqlAzureConnections` - which correspond to the Azure SQL databases to monitor. You can as many connection strings as instances you want to monitor - just ensure that your virtual machine is powerful enough to handle the load. 

In addition, the config allows to specify connection strings for Azure SQL Managed Instances and virtual-machine based SQL Servers using these paramters: 

| Parameter                     | Database type                              |
|-------------------------------|---------------------------------------------|
| sqlAzureConnections           | Azure SQL Database                          |
| sqlManagedInstanceConnections | Azure SQL Managed Instances                 |
| sqlVmConnections              | SQL Servers (virtual machine or on-premise) |

Notice that the strings in the config are not raw connection strings, but contain references to parameters and secrets in the config. This provides some important benefits:

1. Allows the configs to be free of secrets. All secrets can be stored in an Azure Key Vault and the dynamic values are retrieved at run time. This enables scenarios like `config-as-code`. 
2. Reusability of tokens. For instance, the user name is referenced as a parameter, not a static value. This means you can change its value in one place and all references will auto-update their values. 

Once you have setup the connection string and deployed it, Azure Monitor will combine the tokens with your SQL profile config and create a unique monitoring config for the virtual machine. See [Workload monitoring configuration](wli-configs.md) for more information.

The net effect is that the monitoring VM will begin collecting SQL metrics from the two SQL instances corresponding to the connection strings.

### Parameters
Parameters are basically tokens that can be referenced in the profile configuration via templating. Parameters have a name and a value; values can be any JSON type including objects and arrays. In the SQL config, the required parameters include `sqlAzureConnections`, `sqlManagedInstanceConnections` and `sqlVmConnections` and each expects zero or more connection strings as an array.  

Parameters can also reference secrets in Key Vault using the same convention. For example, `sqlAzureConnections` also references the secret `telegrafPassword` using the convention `$telegrafPassword`. 

At run time, all parameters/secrets will be resolved and merged with the profile config to construct the actual config to be used on the machine. 

### Secrets
Secrets are tokens whose values are retrieved at run time from an Azure Key Vault. A secret is defined by a pair of a Key Vault reference and secret name. This allows Azure Monitor to get the dynamic value of the secret and use it is downstream config references. 

You can define as many secrets as needed in the config, including to seperate Key Vaults.

```json
    "secrets": {
        "<secret-token-name-1>": {
            "keyvault": "<key-vault-uri>",
            "name": "<key-vault-secret-name>"
        },
        "<secret-token-name-2>": {
            "keyvault": "<key-vault-uri-2>",
            "name": "<key-vault-secret-name-2>"
        }
    }
```

The permissions to access the Key Vault is provided to a Managed Service Identity on the VM (usually at onboarding time). Azure Monitor expects the Key Vault to provide at least secrets `get` permission to the VM. You can enable it from the [Azure portal](https://docs.microsoft.com/en-us/azure/key-vault/general/assign-access-policy-portal), [PowerShell](https://docs.microsoft.com/en-us/azure/key-vault/general/assign-access-policy-powershell), [CLI](https://docs.microsoft.com/en-us/azure/key-vault/general/assign-access-policy-cli) or [ARM template](https://github.com/acearun/managedsolutions/blob/master/Templates-Dcr/Add-monitoring-vm/kvdeploy.json)

### Config update frequency
Azure Monitor refreshes and resolves both configs every 5 minutes. This allows for new configs, tokens and configs to take effect without the need for a redeploy within a relatively short window. 