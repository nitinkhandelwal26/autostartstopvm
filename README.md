# Start / stop VMs on a schedule

Create an Azure function application and deploy functions that starts or stops virtual machines in the specified resource group, subscription, or by tag on a schedule.

### Login to Azure Account 
```powershell
Install-Module -Name Az -AllowClobber -Scope AllUsers

Connect-AzAccount
$subcription = Get-AzSubscription
$subcription
Select-AzSubscription -Tenant <{TenantId}>
```

### Create a new resource group and function application on Azure

## Publish ARM template

Go to Templates in Azure portal
copy azuredeploy.json to newly created template.


## Deploy ARM template

Modify the values for each of the below variables in run.ps1 as needed.

```powershell
# Specify the VMs that you want to stop. Modify or comment out below based on which VMs to check.
#$VMResourceGroupName = "vmrg"
#$VMName = "testVM1"
$TagName = "AutomaticallyStart"
```

Modify the start and stop time in the function.json file. They are currently set to 6:30am and 6:30pm CEST. You can change the timezone by modifying the application setting WEBSITE_TIME_ZONE. You can also pass in a parameter 'timezone' to the above [ARM template] that was used to create the function application if you want a different timezone so that daylight savings time will be honored.

```json
{
  "disabled": false,
  "bindings": [
    {
      "name": "Timer",
      "type": "timerTrigger",
      "direction": "in",
      "schedule": "0 0 20 * * *"
    }
  ]
}
```
Go to template newly created, click on Deploy
Provide the name of new resource group where this template will be deployed
Click on Publish

This should create a new resource group with a function application and a managed service identity enabled. The id of the service principal for the MSI should be returned as an output from the deployment.

Example: principalId    String   cac1fa06-2ad8-437d-99f6-b75edaae2921

### Grant the managed service identity contributor access to the subscription or resource group so it can perform actions

The below command sets the access at the subscription level.

```powershell
$Context = Get-AzContext
New-AzRoleAssignment -ObjectId <principalId> -RoleDefinitionName Contributor -Scope "/subscriptions/$($Context.Subscription)"
```

