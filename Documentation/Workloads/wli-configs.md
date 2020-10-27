# Workload monitoring configuration

Azure Monitor workload monitoring supports two types of configs - which together allow you to specify your data collection needs and setup.

1. Profile configuration - allows you to specify `what` telemetry is to be collected for the workload. The `what` is usually the common for your fleet of workloads in a profile, and therefore managed at that level.

2. Per-machine configuration - allows you to specify `where` to collect the data from. This can be unique based on the setup of your workloads per machine. For instance, the workload may need info about machine name, ip-address or a connection string that is unique to the machine. 

Azure Monitor allows you to merge the per-machine config with the profile config using [ Go’s text template syntax](https://golang.org/pkg/text/template/). You can use  these together to specify Telegraf inputs configuration in TOML format. 

To see the two configs working together, let's construct a config that allows you to monitoring SQL workloads. 

## Profile config
The following config specifies the high level Telegraf settings that will be used to determine what is collected. 
```toml
{{ $connections := .Parameters.connections }}
[[inputs.sqlserver]]
  servers = [
    {{- range $ind, $cs := $connections }}
    {{- if $ind}}, {{end}}
		"{{ $cs }}"
		{{- end}}
	]
	query_version = 2
	exclude_query = ['Schedulers', 'SqlRequests']
```
In this example, it specifies:
1. Use the SQL input plugin,
2. Use version 2 of the metrics format, and
3. Collect all metrics except Schedulers and SqlRequests

These kinds of settings are usually common for all workload instances in the profile. 

Note that it does not specify the details of the specific workload, e.g. its connection string. Instead, it uses a Go-style templating snippet to setup a merge from the per-machine config at run time. This allows you to have one profile config + *K* per-machine configs - which together specify your monitoring settings.

## Per-machine config
This config essentially specifies the replacement tokens to be used in your profile config. It also allows you to reference secrets from Azure Key Vault so you don't have keep secret values in any config.

Here is the per-machine config corresponding to the profile config above:
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
        "telegrafUsername": "telegraf",
        "connections": [
            "Server=mysqlserver.database.windows.net;Port=1433;Database=mydatabase;User Id=$telegrafUsername;Password=$telegrafPassword;"
        ]
    }
}
```

The key replacement token is the parameter `connections`. It specifies the skeleton of a connection string array which itself references other parameters and secrets. The resolved values of all tokens will generate a true connection string that will be merged with the profile config as specified in the templating code. 

For instance, the `connections` parameter references another parameter `telegrafUsername` which itself has a hard coded value `telegraf` set in the config. It also references `telegrafPassword` which happens to be a secret referece, whose value is retrieved by looking into the Azure Key Vault `https://mykeyvault.vault.azure.net` for secret name `sqlPassword`. 

The per-machine config when combined with the profile config, generates a fully resolved configuration for the current machine, which has information of both `what` to collect and `where` to collect it from.

### Parameters
Parameters are basically tokens that can be referenced in the profile configuration via JSON templating. Parameters have a name and a value; values can be any JSON type including objects and arrays. A parameter is referenced in the profile config using its name in this convention `.Parameters.<name>`. 

Parameters values can reference other parameters. In the example above, the parameter `connections` references the parameter `telegrafUsername` using the convention `$telegrafUsername`. You can use as many levels of references as needed for your scenario, the only requirement is that referenced parameters need to be defined above the ones using it. 

Parameters can also reference secrets in Key Vault using the same convention. For example, `connections` also references the secret `telegrafPassword` using the convention `$telegrafPassword`. 

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

The permissions to access the Key Vault is provided to a Managed Service Identity on the VM (usually at onboarding time). Azure Monitor expects  the Key Vault to provide at least secrets `get` permission to the VM. You can enable it from the [Azure portal](https://docs.microsoft.com/en-us/azure/key-vault/general/assign-access-policy-portal), [PowerShell](https://docs.microsoft.com/en-us/azure/key-vault/general/assign-access-policy-powershell), [CLI](https://docs.microsoft.com/en-us/azure/key-vault/general/assign-access-policy-cli) or [ARM template](https://github.com/acearun/managedsolutions/blob/master/Templates-Dcr/Add-monitoring-vm/kvdeploy.json)

### Config update frequency
Azure Monitor refreshes and resolves both configs every 5 minutes by default. This allows for new configs, tokens and configs to take effect without the need for a redeploy within a relatively short window. 