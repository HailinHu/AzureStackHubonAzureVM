# Azure Stack on Azure
Creates new VM and install prerequisites for Azure Stack Hub Development kit (ASDK) to run Proof of Concept.

## Welcome to the Azure Stack Hub on Azure Step by Step Guide!
In this guide, we'll walk you experiencing a number of the amazing capabilities within Azure Stack Hub 2012.

## Description
Deploy-AzureStackonAzureVM.ps1 script prepares Storage Account and copy VHD file before you calling ARM Template

## Prepaire for the Deployment
* Gloabl Azure AD tenant with Gloabladmin permission


## Step by Step Guidance
### Step 1 - Download the Deploy-AzureStackonAzureVM.ps1 script

Run the following PowerShell command to install the new installation script. This will also downloads required modules from powershell gallery.

```powershell
Find-Script Deploy-AzureStackonAzureVM | Install-Module -Force
```

Can be run from Azure Cloudshell as well. :)

### Step 2 - Run the Deploy-AzureStackonAzureVM.ps1 script

Once script downloaded from PowerShell gallery, run the following command to locate some examples.

```powershell
Get-Help Deploy-AzureStackonAzureVM.ps1 -Examples
```

If your scenario does not require customization, simply run Deploy-AzureStackonAzureVM.ps1 without any parameters. if you provide any parameters, It will use default parameters and will prompt for local Administrator password for the new VM.

#### Example 1

```powershell
Deploy-AzureStackonAzureVM.ps1 -Verbose
```

#### Example 2

```powershell
Deploy-AzureStackonAzureVM -ResourceGroupName myResourceGroup -Region 'West Europe' -VirtualMachineSize 'Standard_E32s_v3'
```

Note: The execution of the script takes appoximately 20 mins or so. Please do NOT close the browser or cloudshell window during execution.

### Step 3 - Starting Azure Stack Hub deployment
Once the VM deployed logon to VM using 'Administrator' username and the password provided during the deployment. 
a. Open the PowerShell and excute below script,

   #### Deploy Azure Stack Using Azure AD

```powershell
cd C:\CloudDeployment\Setup     
$adminpass = Get-Credential Administrator     
.\InstallAzureStackPOC.ps1 -AdminPassword $adminpass.Password
```
   #### Deploy Azure Stack Using AD FS

```powershell
cd C:\CloudDeployment\Setup     
$adminpass = Get-Credential Administrator 
.\InstallAzureStackPOC.ps1 -AdminPassword $adminpass.Password -UseADFS
```

![GitHub Logo](images/image02.png)

**Note1**: After that, it takes about 10 hours to complete the setup and 310 steps :)) During this time, your VM may restart several times. But the installation will continue. 

![GitHub Logo](images/image03.png)

**Note2**: If you want to monitor the deployment progress, sign in as azurestack\AzureStackAdmin after the ASDK host restarts.

![GitHub Logo](images/image04.png)



## Install PowerShell commands for Azure Stack hub

You can install PowerShell commands for Azure Stack through the PowerShell Gallery.

```powershell
Set-PSRepository -Name "PSGallery" -InstallationPolicy Trusted
```

After that, you can install the latest Azure Stack PowerShell module to the ASDK host by running the commands below.

```powershell
[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12

  Install-Module -Name Az.BootStrapper -Force
  Install-AzProfile -Profile 2020-09-01-hybrid -Force
  Install-Module -Name AzureStack -RequiredVersion 2.1.0

  Get-Module -Name "Az*" -ListAvailable
  Get-Module -Name "Azs*" -ListAvailable
```

We can now configure a GitHub repository hosting PowerShell modules in the ASDK Host VM to manage resources and deploying them to Azure Stack. 

```powershell
# Change directory to the root directory.
cd \

# Download the tools archive.
[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12 
invoke-webrequest `
  https://github.com/Azure/AzureStack-Tools/archive/az.zip `
  -OutFile az.zip

# Expand the downloaded files.
expand-archive az.zip `
  -DestinationPath . `
  -Force

# Change to the tools directory.
cd AzureStack-Tools-az
```

You can run the following commands to verify that your ASDK deployment is successful:

a. Sign in as AzureStack\AzureStackAdmin on the ASDK host computer.

b. Open PowerShell as an admin (not PowerShell ISE).

```powershell
Enter-PSSession -ComputerName AzS-ERCS01 -ConfigurationName PrivilegedEndpoint
Test-AzureStack
```

![GitHub Logo](images/image01.png)

## Register AAD account for Azure Stack hub

As the last step in our ASDK configuration, you need to register your ASDK environment with Azure. This will allow you to use both Azure marketplace items and full features of Azure Stack. You can run the command below to register. Make sure to define your Azure Subscription ID and a uniqe-registration-name before running the command.
 
```powershell
# Add the Azure cloud subscription environment name. 
# Supported environment names are AzureCloud, AzureChinaCloud, or AzureUSGovernment depending which Azure subscription you're using.
Connect-AzAccount -EnvironmentName "<environment name>"

# Register the Azure Stack Hub resource provider in your Azure subscription
Register-AzResourceProvider -ProviderNamespace Microsoft.AzureStack

# Import the registration module that was downloaded with the GitHub tools
Import-Module C:\AzureStack-Tools-az\Registration\RegisterWithAzure.psm1

# If you have multiple subscriptions, run the following command to select the one you want to use:
# Get-AzSubscription -SubscriptionID "<subscription ID>" | Select-AzSubscription

# Register Azure Stack Hub
$AzureContext = Get-AzContext
$CloudAdminCred = Get-Credential -UserName AZURESTACK\CloudAdmin -Message "Enter the credentials to access the privileged endpoint."
$RegistrationName = "<unique-registration-name>"
Set-AzsRegistration `
-PrivilegedEndpointCredential $CloudAdminCred `
-PrivilegedEndpoint AzS-ERCS01 `
-BillingModel Development `
-RegistrationName $RegistrationName `
-UsageReportingEnabled:$true
 ```

After the command runs successfully, you will see the message that your environment has been successfully registered to Azure. :smile:

![GitHub Logo](images/Image05.png)

After everything is completed, you can connect to https://adminportal.local.azurestack.external to the admin portal, and you can control the user portal by connecting to https://portal.local.azurestack.external.

![GitHub Logo](images/admimportal.png)

![GitHub Logo](images/userportal.png)

We will go into the details of the configuration in our next articles on Azure Stack Hub.

References:
> Yağmur Şahin GitHub: https://github.com/yagmurs/AzureStack-VM-PoC
>
> Azure Docs: https://docs.microsoft.com/en-us/azure-stack/asdk/?view=azs-2008
>
> DevCloudOps: https://www.devcloudops.com/2020/12/16/azure-stack-hub-development-kit-asdk-installation-and-configuration-on-azure-vm/
