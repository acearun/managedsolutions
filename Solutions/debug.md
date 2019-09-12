# Debugging managed applications

## Issues with deploying an app definition
These are usually related to issues in the `azuredeploy.json`. The portal notification popup usually has good information about why the deployment failed:

1. Invalid app def name.
2. Insufficient permissions.
3. Bad payloads.

## Issues with deploying an main template
The best place to find information is the deployment logs in the target resource groups. I usually use the link in the deployment failure notification to go to the main resource group. Go the _Deployments_ menu item to see the logs related to this resource group. This info is good if there are issues deploying the manged app itself. 

If the managed app exists there in the main resource group, click it and go to its overview page. In the essentials section on top of the overview, click on the managed resource group name - this takes you to the hidden resource group. It also has a _Deployments_ menu item with more information about deployments of secondary components like alerts. This is where I find the most useful information for debugging - including information about nested deployments.

## Common issues
### Principal not found
A known issue related to permissions not propagating fast enough. Workarounds include:
1. Keeping the app def and app resource group the same to reduce the chance of this issue happening.
2. Retrying if it happens - usually doesn't happen the second time.

