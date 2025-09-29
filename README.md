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
As instructed edit the inputs.yaml file to copy the inputs-azure-devops.yaml values.
while configuring the file I left a null string for the security subscription_id and added a ~ (tilde symbol) at the beginning of the output_folder_path tuple. 


