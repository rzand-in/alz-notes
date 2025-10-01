# alz-notes
Notes about building Azure Landing Zones using the Accelerator

## Purpose
Microsoft maintains the https://azure.github.io/Azure-Landing-Zones/accelerator/ repository to create Azure Landing Zones.
There is a large amount of information about Azure Landing Zones and Cloud Adoption Framework in general at https://aka.ms/alz . What this repository wants to do is to create a simple guide about how to use the code produced and maintaned by Microsoft in the repository mentioned above.
Even when the concepts about landing zones are clear, using the alz repo to deploying them is complex, but to get a working environment we can follow the steps described below.

## Which Micreosoft Landng Zone repository to use
We want to deploy ALZ via infrastructure as code.
There are 2 Github repositories for Landing Zones: 
* The Enterprise Scale - https://github.com/Azure/Enterprise-Scale
* The Accelerator - https://azure.github.io/Azure-Landing-Zones/accelerator/
This is a bit confusing. My understanding is that the former is cronologically the first reference to build Landing Zones, the latter is the new approach that uses Azure Verified Modules to build the infrastructure.
The latter is the one of interest here.
A video at https://www.youtube.com/watch?v=YxOzTwEnDE0&t=4403s shows that even the Microsoft employee using the repo they built and taking some time to deploy.

## Implementation
I followed the 4 phases instruction at https://azure.github.io/Azure-Landing-Zones/accelerator/userguide/
downloaded and filled the spreadsheet

run commands:
```
pwsh --version
az version
```
to check the versions of the installed product (PowerShell 7.5.1 and azure-cli": "2.77.0" in my case)
Launched powershell (This is an Ubuntu OS)
```
pwsh
```
Checked and Installed the alz Powershell module.
```
Get-InstalledModule -Name ALZ
Install-Module -Name ALZ
```
The phase number 2 require to make choices. The first is to choose which kind of configuration to run. The choice for this notes is the "Platform Landing Zone Library (lib) Folder" one that should allow us to easily do customization. Ther choices are to use Terraform and Azure DevOps.
Before starting the last phase we need to create the bootstrap envirenment.

To generate the bootstrap configuration for this project I followed the instruction at https://azure.github.io/Azure-Landing-Zones/accelerator/userguide/2_start/terraform-azuredevops/ and run the following commands:
```
pwsh
New-Item -ItemType "file" "~/accelerator/config/inputs.yaml" -Force
New-Item -ItemType "directory" "~/accelerator/output"
New-Item -ItemType "file" "~/accelerator/config/platform-landing-zone.tfvars" -Force  # Exclude this line if using FSI or SLZ starter modules
New-Item -ItemType "directory" "~/accelerator/config/lib"
$tempFolderName = "~/accelerator/temp"
New-Item -ItemType "directory" $tempFolderName
$tempFolder = Resolve-Path -Path $tempFolderName
git clone -n --depth=1 --filter=tree:0 "https://github.com/Azure/alz-terraform-accelerator" "$tempFolder"
cd $tempFolder

$libFolderPath = "templates/platform_landing_zone/lib"
git sparse-checkout set --no-cone $libFolderPath
git checkout

cd ~
Copy-Item -Path "$tempFolder/$libFolderPath" -Destination "~/accelerator/config" -Recurse -Force
Remove-Item -Path $tempFolder -Recurse -Force
```
As instructed I edited the inputs.yaml file to copy the inputs-azure-devops.yaml values and assign the values that I previously wrote in the spreadsheet file.
While configuring the inputs file I left a null string for the security subscription_id and added a ~ (tilde symbol) at the beginning of the output_folder_path tuple. 
Now is time to populate the platform-landing-zone.tfvars with the values of one of the most popular scenarios: Multi-Region Hub and Spoke Virtual Network with Azure Firewall.
I copied the [hub-and-spoke-vnet.tfvars](https://raw.githubusercontent.com/Azure/alz-terraform-accelerator/refs/heads/main/templates/platform_landing_zone/examples/full-multi-region/hub-and-spoke-vnet.tfvars) and modified it to disable the creation of the bastion hosts and virtual network gateway to spare some money and time. I also disable the telemetry since this is just a test.
I did not create a service principal during the prerequisites phase, so I did it now. I granted the new SPN the owner rbac as per instruction but to be able to grant the rbac to the root management group I had to [Elevate access for a Global Administrator](https://learn.microsoft.com/en-gb/azure/role-based-access-control/elevate-access-global-admin?tabs=azure-portal%2Centra-audit-logs#perform-steps-at-root-scope) and log off and logon again the Azure portal.
I assigned the owner role with "Allow user to assign all roles (highly privileged)" since it was not specified which selection to choose.
I am logged on to az cli so I do not need to set the following variables:
```
$env:ARM_TENANT_ID="<tenant id>"
$env:ARM_CLIENT_ID="<client id>"
$env:ARM_CLIENT_SECRET="<client id>"
$env:ARM_SUBSCRIPTION_ID="<subscription id>"
```
Now I can run the accelerator:
```
Deploy-Accelerator `
-inputs "~/accelerator/config/inputs.yaml", "~/accelerator/config/platform-landing-zone.tfvars" `
-starterAdditionalFiles "~/accelerator/config/lib" `
-output "~/accelerator/output"
```
The first time I launched the command I received 2 errors:
```
│ Error: Invalid value for variable
│ 
│   on terraform.tfvars.json line 295:
│  295:   "subscription_ids": {
│  296:     "management": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
│  297:     "identity": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
│  298:     "connectivity": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
│  299:     "security": ""
│  300:   },
│     ├────────────────
│     │ var.subscription_ids is map of string with 4 elements
│ 
│ All subscription IDs must be valid GUIDs
```
To correct this I added a sub ID (the management one) for the security subscription. The second error was:
```
│ Error: Listing users: "Request returned status: 401 Unauthorized"
│ 
│   with module.azure_devops.data.azuredevops_users.alz["rzand@hotmail.com"],
│   on ../../modules/azure_devops/groups.tf line 7, in data "azuredevops_users" "alz":
│    7: data "azuredevops_users" "alz" {
│ 
```
This error needed a bit of troubleshooting. I realized that my Azure DevOps organization was not connected with my EntraId directory. I connected the two and also corrected the time zone of the Azure DevOps organization since it was incongrous with the location I am now but I was still receiving the error.
I tried to remove the approver or change it with other pricipal names but no luck.I tried to hardcode a principal_name in the groups.tf file, tried to add the sp-alz-bootstrap application to the Azure DevOps group "[rzand]\Project Collection Administrators" and even to comment all the content after line 6 of the groups.tf file but still getting 401 type errors. I notice that the userId reported for the unauthorized user was not the one of any user in the active directory. I then tried to login to Azure DevOps via command line
```
az devops login
```
using my PAT and then discovered that the user was not authorized to access Azure DevOps objects.
I checked my PAT in the Azure DevOps security and realized that there were no PATs, this I guess because I connected the organization to EntraId.
I recreated the PAT for my user and the Azure DevOps agents, run the deploymenyt one more time and this time the deployment succeeded.
```
Bootstrap has completed successfully! Thanks for using our tool. Head over to Phase 3 in the documentation to continue...
```
I decided to destroy everything and start over with the original groups.tf file that manages the Azure DevOps apprivers.

When I redeployed the accelerator I received an error stating that an Azure DevOps project with a certain Id does not exist or I do not have permissions to access it. That Id is mentioned in the file ./.cache/clipboard-indicator@tudmotu.com/registry.txt
Deleteing the file did not solve the problem so I deleted the entire content of the output directory.
This time the failure was because some roles and agent pool already existed
I removed the agent pool and removed the custom roles. for completing the latter I had to remove the role assignments first.
Finally success!
```
Time taken to complete Terraform apply:

Days Hours Minutes Seconds Milliseconds
---- ----- ------- ------- ------------
0    0     7       30      730


Bootstrap has completed successfully! Thanks for using our tool. Head over to Phase 3 in the documentation to continue...
```





