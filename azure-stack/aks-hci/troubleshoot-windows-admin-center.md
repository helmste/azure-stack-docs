---
title: Troubleshoot Windows Admin Center issues on Azure Kubernetes Service on Azure Stack HCI
description: Learn how to resolve Windows Admin Center issues on Azure Kubernetes Service (AKS) on Azure Stack HCI.
author: v-susbo
ms.topic: how-to
ms.date: 03/23/2021
ms.author: v-susbo
---

# Windows Admin Center troubleshooting

When you use Windows Admin Center (WAC) to create or manage AKS on Azure Stack HCI clusters, you might occasionally come across problems. This article details some common problems and troubleshooting steps that are specific to WAC.

## Create Windows Admin Center logs
When you report problems with Windows Admin Center, it's a good idea to attach logs to help the development team diagnose your problem. Errors in Windows Admin Center generally come in one of two forms: 
- Events that appear in the event viewer on the machine running Windows Admin Center 
- JavaScript problems that surface in the browser console 

To collect logs for Windows Admin Center, use the `Get-SMEUILogs.ps1` script that's provided in the public preview package. 
 
To use the script, run this command in the folder where your script is stored: 
 
```PowerShell
./Get-SMEUILogs.ps1 -ComputerNames [comp1, comp2, etc.] -Destination [comp3] -HoursAgo [48] -NoCredentialPrompt
```
 
The command has the following parameters:
 
- `-ComputerNames`: A list of machines you want to collect logs from.
- `-Destination`: The machine you want to aggregate the logs to.
- `-HoursAgo`: The start time for collecting logs, expressed in hours before the time you run the script.
- `-NoCredentialPrompt`: A switch to turn off the credentials prompt and use the default credentials in your current environment.
 
If you have difficulties running this script, you can run the following command to view the Help text: 
 
```PowerShell
GetHelp .\Get-SMEUILogs.ps1 -Examples
```

## Troubleshoot CredSSP issues

When deploying AKS on Azure Stack HCI using WAC, and the deployment hangs for an extended period, you might be having CredSSP or connectivity problems. Try the following steps to troubleshoot your deployment:
 
1. On the machine running Windows Admin Center, run the following command in a PowerShell window: 

   ```PowerShell
      Enter-PSSession <servername>
   ```
2. If this command succeeds, you can connect to the server and there's no connectivity issue.
    
3. If you're having CredSSP problems, run this command to test the trust between the gateway machine and the target machine: 

   ```PowerShell
      Enter-PSSession –ComputerName <server> –Credential company\administrator –Authentication CredSSP
   ``` 
   You can also run the following command to test the trust in accessing the local gateway: 

   ```PowerShell
      Enter-PSSession -computer localhost -credential (Get-Credential)
   ``` 

## A WinRM error is displayed when creating a new workload cluster

When switching from DHCP to static IP, WAC displayed an error that said the WinRM client cannot process the request. This error also occurred outside of WAC. WinRM broke when static IP addresses were used, and the servers were not registering an Service Principal Name (SPN) when moving to static IP addresses. 

To resolve this issue, use the **SetSPN** command to create the SPN. From a command prompt on the WAC gateway, run the following command: 

```
Setspn /Q WSMAN/<FQDN on the Azure Stack HCI Server> 
```

Next, if any of the servers in the environment return the message `No Such SPN Found`, then log in to that server and run the following commands:  

```
Setspn /S WSMAN/<server name> <server name> 
Setspn /S WSMAN/<FQDN of server> <server name> 
```

Finally, on the WAC gateway, run the following to ensure that it gets new server information from the domain controller:

```
Klist purge 
```

## Deployment fails when using Azure Arc with multiple tenant IDs
If you're using Azure Arc and have multiple tenant IDs, run the following command to specify the tenant you plan to use before starting the deployment. If you don't specify a tenant, your deployment might fail.

```Azure CLI
az login –tenant <tenant>
```

## Issues occurred when registering the Windows Admin Center gateway with Azure
If you've just created a new Azure account and haven't signed in to the account on your gateway machine, you might experience problems with registering your Windows Admin Center gateway with Azure. To mitigate this problem, sign in to your Azure account in another browser tab or window, and then register the Windows Admin Center gateway to Azure.

## Next steps
- [Troubleshoot common issues](./troubleshoot.md)
- [Resolve known issues](./troubleshoot-known-issues.md)

If you continue to run into problems when you're using Azure Kubernetes Service on Azure Stack HCI, you can file bugs through [GitHub](https://aka.ms/aks-hci-issues).
