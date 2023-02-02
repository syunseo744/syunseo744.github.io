---
date: 2023-01-17T00:00:00.000+13:00
title: Azure Deployment history cleanup with Azure DevOps
author: Luke
categories:
- Azure
toc: true
header:
  teaser: ''

---
Microsoft Azure has a limit of [800 deployments per resource group](https://learn.microsoft.com/azure/azure-resource-manager/management/azure-subscription-service-limits?WT.mc_id=AZ-MVP-5004796#resource-group-limits "Resource group limits"). This means that a single resource group can only contain 800 historical deployments at most.

> A deployment in Azure refers to the process of creating or updating resources in a resource group.

When deploying resources in Azure, it is essential to keep track of the number of [historic deployments](https://learn.microsoft.com/en-us/azure/azure-resource-manager/templates/deployment-history?tabs=azure-portal&WT.mc_id=AZ-MVP-5004796 "View deployment history with Azure Resource Manager") in a resource group to ensure that the limit is not exceeded. This is because new deployments will fail if the limit is exceeded, and creating or updating resources in that resource group will not be possible.

If you have CI/CD _(Continuous Integration and Continuous Deployment)_ set up to deploy or change your infrastructure or services with code, it can be easy to reach this limit.

To avoid exceeding the deployment limit, it may be necessary to clean up old deployments.

This can be done by using a script to remove deployments that are no longer needed.

So let's build an Azure DevOps pipeline that runs weekly to connect to our Microsoft Azure environment and clean up historical deployments.

For this article, I will assume you have an Azure DevOps repository setup and the permissions _(Owner)_ to make the necessary privileged actions to the Microsoft Azure environment to do the setup.

#### Deploy and Configure

##### Create Service Prinicipal

 1. Navigate to the [**Microsoft Azure Portal**](https://portal.azure.com/#home "Microsoft Azure")
 2. Click on [**Azure Active Directory**](https://portal.azure.com/#view/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/\~/Overview "Azure Active Directory")
 3. Click on [**App Registrations**](https://portal.azure.com/#view/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/\~/RegisteredApps "Azure App Registrations")
 4. Click on: **+ New Registration**
 5. Enter the following information:
    * **Name** (i.e. SPN.AzDeploymentCleanup)
 6. Click **Register**
 7. Copy the following for later when we add the SPN to Azure DevOps.
    * Application (client) ID
    * Directory (tenant ID)
 8. Click on **Certificates & Secrets**
 9. Press **+ New Client Secret**
10. Enter a relevant description and expiry date and click Add
11. **Copy** the **value** of the new secret (this is essentially your password), and you won't be able to see the value again.

##### Create Custom Role & Assign permissions

Now that your service principal has been created, it is time to assign permissions because this script targets all subscriptions under a management group; we are going to set the permissions to that management group so that it flows to all subscriptions underneath it - and in the view of least privileged we will create a Custom Role to apply to our Service Principal.

#### Create Custom Role

In order for the deployment history to be completed, we will need the following permissions:

* "Microsoft.Management/managementGroups/read" for Get-AzManagementGroup
* "Microsoft.Management/managementGroups/subscriptions/read" for Get-AzManagementGroupSubscription
* "Microsoft.Resources/subscriptions/resourceGroups/read" for Get-AzResourceGroup
* "Microsoft.Resources/subscriptions/resourceGroups/deployments/read" for Get-AzResourceGroupDeployment
* "Microsoft.Resources/subscriptions/resourceGroups/deployments/delete" for Remove-AzResourceGroupDeployment

 1. Navigate to the [**Microsoft Azure Portal**](https://portal.azure.com/#home "Microsoft Azure")
 2. In the search bar above, type in and navigate to [**Management Groups**](https://portal.azure.com/#view/Microsoft_Azure_ManagementGroups/ManagementGroupBrowseBlade/\~/MGBrowse_overview "Management Groups")
 3. Click a management group, click on **Access Control (IAM)**
 4. Click **+ Add**
 5. Click **Add Custom Role**
 6. Type in a role name _(an example is: AzDeploymentHistoryCleanup)_
 7. Check Start from **Scratch** and next click
 8. Click **+ Add permissions** and add the permissions below (you can search for them):
 9. Click **Next**
10. Add **Assignable Scopes** _(this is the scope you can use to assign a role to - this won't assign it to the Service Principal; it will only open it up so we can assign it)_. Make sure you assign it at the management group level you are targetting.
11. Click **Review + Create**
12. Click **Create**

#### Assign Permissions

Now that the custom role has been created, it is time to assign the role to the service principal we created earlier.

 1. Navigate to the [**Microsoft Azure Portal**](https://portal.azure.com/#home "Microsoft Azure")
 2. In the search bar above, type in and navigate to [**Management Groups**](https://portal.azure.com/#view/Microsoft_Azure_ManagementGroups/ManagementGroupBrowseBlade/\~/MGBrowse_overview "Management Groups")
 3. Click on the management group you want to manage and click on **Access Control (IAM)**
 4. Click **Add**
 5. Click **Add Role Assignment**
 6. Select your custom role _(you can toggle the type column, so CustomRoles are first in the list)_
 7. Click **Members**
 8. Make sure '_User, group or service principal_' is selected and click **+ Select Members**
 9. Select your Service Principal created earlier _(i.e. SPN.AzDeploymentCleanup)_
10. Click **Select**
11. Click **Review + assign** to assign the role.
12. 