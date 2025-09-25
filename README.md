# alz-notes
Notes about building Azure Landing Zones using the Accelerator

## Purpose
Microsoft maintains the https://azure.github.io/Azure-Landing-Zones/accelerator/ repository to create Azure Landing Zones.
There is a large amount of information about Azure Landing Zones and Cloud Adoption Framework in general at https://aka.ms/alz, what this repository wants to do is to create a simple guide about how to use the code produced and maintaned by Microsoft in the repository mentioned above.
Even when the concepts about landing zones are clear, using the alz repo to deploying them is complex, but to get a working environment we can follow the steps described below.

## Which Micreosoft Landng Zone repository to use
We want to deploy ALZ via infrastructure as code.
There are 2 Github repositories for Landing Zones: 
* The Enterprise Scale - https://github.com/Azure/Enterprise-Scale
* The Accelerator - https://azure.github.io/Azure-Landing-Zones/accelerator/
This is a bit confusing. My understanding is that the former is cronologically the first reference to build Landing Zones, the slatter is the new approach that uses Azure Verified Modules to build the infrastructure.
The latter is the one of interest here.
A video at https://www.youtube.com/watch?v=YxOzTwEnDE0&t=4403s shows some Microsoft employee using the repo they uiot and taking some time to deploy.

## Implementation
I followed the instruction at
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
