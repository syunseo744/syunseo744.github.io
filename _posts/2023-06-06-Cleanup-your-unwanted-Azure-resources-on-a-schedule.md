---
title: Cleanup your unwanted Azure resources on a schedule
author: Luke
categories:
  - Azure
  - PowerShell
permalink: /azure/Cleanup-your-unwanted-Azure-resources-on-a-schedule/
toc: false
header:
  teaser: /images/posts/CleanupyourUnwantedAzureResourcesonaSchedule.png
date: '2023-06-06 00:00:00 +1300'
---

Cleanup your unwanted Azure resources on a schedule

Every few months, I get that dreaded email "Your Microsoft Azure subscription has been suspended" - this is due to creating resources, and leaving them provisioned, so I needed a method of deleting the resources I didn't need, or wanted to spin up for a few days. I also needed away to creating resources that can stay, either for learning or a demo, independent of how the resources were deployed into the environment *(via the Azure Portal, Terraform, Bicep)*.

Naturally I went straight to [Azure Automation](https://learn.microsoft.com/azure/automation/?WT.mc_id=AZ-MVP-5004796 "Azure Automation documentation"){:target="blank"} and using PowerShell.

What I ended up with was a Runbook capable of **EXTREME AZURE DESTRUCTION** which was exactly what I wanted.

> This script is provided as-is with no warranties or guarantees. Use at your own risk. This is not intended to be a script to use in Production, mainly test environments, as this WILL CAUSE massive destruction and irretrievable data loss... You have been warned.

I am not going to go into setting up Azure Automation, if interested you can refer to a few of my blog posts I have done previously that goes through the process:

* [Deallocate ‘Stopped’ Virtual Machines using Azure Automation](https://luke.geek.nz/azure/deallocate-stopped-virtual-machines-using-azure-automation/ "Deallocate ‘Stopped’ Virtual Machines using Azure Automation"){:target="blank"}
* [Turn on a Azure Virtual Machine using Azure Automation](https://luke.geek.nz/azure/turn-on-a-azure-virtual-machine-using-azure-automation/ "Turn on a Azure Virtual Machine using Azure Automation"){:target="blank"}
* [Disable SFTP support on an Azure Storage account on a Schedule](https://luke.geek.nz/azure/disable-sftp-support-on-an-azure-storage-account-on-a-schedule/ "Disable SFTP support on an Azure Storage account on a Schedule"){:target="blank"}

The script named: Invoke-DakaraSuperWeapon, aptly named as a reference to the Dakara weapon from the TV series Stargate SG1 - a weapon if great power.

> The [Dakara superweapon](https://stargate.fandom.com/wiki/Dakara_superweapon){:target="blank"} was a Ancient device capable of reducing all matter to its basic elemental components, and/or restructuring it. Possessing the ability to pass through the shields of known ships it also functions (and has been used) as a devastating weapon to kill the entire crew of orbiting ships or wipe out all life on the surface of hundreds of planets at a time. "It is not only capable of destroying the Replicators but all life in the galaxy."

![Azure Dakara superweapon](/images/posts/CleanupyourUnwantedAzureResourcesonaSchedule.png "Azure Dakara superweapon")

Using the latest Windows PowerShell release - 7.2 (Preview), this script is built around the following capabilities:

* **Delete ALL resource groups** (without a specific Tag) **under all subscriptions**, under a specific **Management Group**
* **Delete all resources** within those resource groups
* **Delete** Azure **Recovery Vaults** and their backed up items
* **Delete** any **Azure policy assignments**, assigned directly to any subscription under the Management Group
* **Delete** any Azure **RBAC role assignments**, assigned directly to any subscription under the Management Group.

In my demo environment, I have a range of Management Groups, and 2 Azure subscriptions.

![Luke's Azure Management Group structure](/images/posts/VisualStudio_Luke_MG_Structure.png "Luke's Azure Management Group structure")\`

For my purposes, I created a [System Managed Identity](https://learn.microsoft.com/azure/automation/enable-managed-identity-for-automation?WT.mc_id=AZ-MVP-5004796 "Using a system-assigned managed identity for an Azure Automation account"){:target="blank"} from the Azure Automation account, and applied it to the: 'mg' Management Group as 'Owner' (Contributor will work, as long as you don't plan on removing the rights from the Azure subscriptions - theoretically, so could Contributor + User Access Administrator roles).

Again - this was created for my own environment - if you decide to run this, TEST IT! And Make sure it has as limited permissions as possible, potentially the Managed Identity will only have access to a specific test Subscription that you may not care about. I take no responsibily.

The System Identity will be used to execute the runbook.

I also needed a Tag *(ie a Safe word)* to save the Resource Groups that I need to remain, an example is a project I am working on, demo etc. This Tag is in name only - as Tags are Key/Value pairs in Azure - in this case I only cared about the Key (ie NotDelete) - what was in the value, didn't matter.

![NotDelete - Azure Tag](/images/posts/Initiate-DakaraSuperWeapon_SafeWord.png "NotDelete - Azure Tag")

Important: When importing the Runbook it is imperative that you Tag the Resource Group it is in, with your safe word! Or else could will be deleted!

The script has a couple of parameters:

| Parameter                | Type    | Notes                                                                                                                                                                                                                                                                                                                                           |
| ------------------------ | ------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| ManagementGroupId        | String  | The ID of the management group to delete resource groups under. WARNING: This script will delete all resource groups under the specified management group except for the ones with the specified tag. Make sure you have specified the correct management group ID, or you may accidentally delete resources that you did not intend to delete. |
| TagName                  | String  | The name of the tag to check for. WARNING: This script will delete all resource groups that do not have this tag. Make sure you have specified the correct tag name, or you may accidentally delete resources that you did not intend to delete.                                                                                                |
| RemoveResourceGroups     | Boolean | True or False, do you want to Remove the Resource Groups? True means it will, and False means it will skip the Resource Group deletion.                                                                                                                                                                                                         |
| DeletePolicyAssignments  | Boolean | True or False, do you want to Remove the Azure Policy assignments on the subscriptions? True means it will, and False means it will skip the Azure Policy assignment deletion.                                                                                                                                                                  |
| DeleteSubRoleAssignments | Boolean | This will need Owner rights (or User Administrator role) in order to remove roles from a Subscription. Make sure your rights are set to be inherited from a Management Group, before running this. True or False, True means it will delete the Subscription direct assignments, False means it will skip it.                                   |

As you can tell, you can enable or disable specific parts of the script, for example - if you just want to use it to clean up direct role assignments on your subscriptions, while not deleting Azure resources you can by entering True or False.

![Initiate-DakaraSuperWeapon - Azure Runbook Parameters](/images/posts/Initiate-DakaraSuperWeapon_Parameters.png "Initiate-DakaraSuperWeapon - Azure Runbook Parameters")

When ran it will stream the Logs to the Azure Automation Log Stream, there is no waiting time or approval - it will just run.

![Initiate-DakaraSuperWeapon - Azure Automation Log Stream](/images/posts/Initiate-DakaraSuperWeapon_Delete.png "Initiate-DakaraSuperWeapon - Azure Automation Log Stream")

As below, you can see the Resource Groups get removed (at the time of this recording, I had a limit on the amount of [parallel](https://devblogs.microsoft.com/powershell/powershell-foreach-object-parallel-feature/?WT.mc_id=AZ-MVP-5004796 "PowerShell ForEach-Object Parallel Feature"){:target="blank"} delete tasks:

![Remove Azure Resoure Groups](/images/posts/Remove_AzResource.gif "Remove Azure Resoure Groups")

{% gist 367e94e408ffafce7eb55a643a364ee3 %}

Using the Azure Automation schedule, I can then set this Runbook to run every Day, Week etc - knowing my environment will be fresh for my next project, learning exercise.