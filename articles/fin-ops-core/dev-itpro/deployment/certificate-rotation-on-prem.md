---
# required metadata

title: Certificate rotation
description: This topic explains how to place existing certificates and update the references within the environment to use the new certificates.
author: PeterRFriis
manager: AnnBe
ms.date: 05/21/2020
ms.topic: article
ms.prod: 
ms.service: dynamics-ax-applications
ms.technology: 

# optional metadata

# ms.search.form:  [Operations AOT form name to tie this topic to]
audience: IT Pro
# ms.devlang: 
ms.reviewer: sericks
ms.search.scope: Operations
# ms.tgt_pltfrm: 
# ms.custom: [used by loc for topics migrated from the wiki]
ms.search.region: Global 
# ms.search.industry: [leave blank for most, retail, public sector]
ms.author: perahlff
ms.search.validFrom: 2019-04-30
ms.dyn365.ops.version: Platform update 25 

---

# Certificate rotation

[!include[banner](../includes/banner.md)]

You may need to rotate the certificates used by your Dynamics 365 Finance + Operations (on-premises) environment as they approach their expiration date. In this topic, you will learn how to replace the existing certificates and update the references within the environment to use the new certificates.

> [!WARNING]
> The certificate rotation process should be initiated well before the certificates expire. This is very important for the Data Encryption certificate, which could  cause data loss for encrypted fields. For more information, see [After certificate rotation](#aftercertrotation). 
> 
> Old certificates must remain in place until the certificate rotation process is complete, removing them in advance will cause the rotation process to fail.

## Preparation steps 

1. Rename the original **Infrastructure** folder that you created during the process to [Download setup scripts from LCS](setup-deploy-on-premises-pu12.md#downloadscripts). Rename the folder to **InfrastructureOld**.

2. Download the latest setup scripts from [Download setup scripts from LCS](setup-deploy-on-premises-pu12.md#downloadscripts). Unzip the files into a folder that is named **Infrastructure**.

3. Copy **ConfigTemplate.xml** and **ClusterConfig.json** from **InfrastructureOld** to **Infrastructure**.

4. Configure certificates as needed in **ConfigTemplate.xml**. Follow the steps in [Configure certificates](setup-deploy-on-premises-pu12.md#configurecert), specifically these steps:

    ```powershell
    # Create self-signed certs
    .\New-SelfSignedCertificates.ps1 -ConfigurationFilePath .\ConfigTemplate.xml
    ```

    > [!IMPORTANT]
    > Self-signed certificates should never be used in production environments. If you're using trusted certificates, manually update the values of those certificates in the ConfigTemplate.xml file.

    ```powershell
    # Export Pfx files into a directory VMs\<VMName>, all the certs will be written to infrastructure\Certs folder
    .\Export-PfxFiles.ps1 -ConfigurationFilePath .\ConfigTemplate.xml
    ```

5. Continue to [Setup VMs](setup-deploy-on-premises-pu12.md#setupvms). The specific steps that are needed for this process include:

    1. Export the scripts that must be run on each VM.
    
        ```powershell
        # Export the script files to be executed on each VM into a directory VMs\<VMName>
        .\Export-Scripts.ps1 -ConfigurationFilePath .\ConfigTemplate.xml
        ```

    2. Copy the contents of each infrastructure\\VMs<VMName> folder into the corresponding VM (if remoting scripts are used, they will automatically copy the content to the target VMs), and then run the following scripts, if they exist. Perform these steps as an Administrator.
	
        ```powershell
        # If remoting, only execute
        # .\Complete-PreReqs-AllVMs.ps1 -ConfigurationFilePath .\ConfigTemplate.xml -ForcePushLBDScripts

        .\Import-PfxFiles.ps1
        .\Set-CertificateAcls.ps1
        ```
        
    3. Run the following script to validate the VM setup.
    
        ```powershell
        # If remoting, only execute
        # .\Test-D365FOConfiguration-AllVMs.ps1 -ConfigurationFilePath .\ConfigTemplate.xml
        .\Test-D365FOConfiguration.ps1
        ```

6. If axdataenciphermentcert certificates are rotated, you need to regenerate the credentials.json file. For more information, see [Encrypt credentials](https://docs.microsoft.com/dynamics365/unified-operations/dev-itpro/deployment/setup-deploy-on-premises-pu12#encryptcred).

7. Run the following PowerShell command to have values that can be used in LCS later. For more information, see [Deploy your on-premises environment from LCS](setup-deploy-on-premises-pu12.md#deploy).

    ```powershell
    .\Get-DeploymentSettings.ps1 -ConfigurationFilePath .\ConfigTemplate.xml
    ```


## Activate new certificates within Service Fabric cluster

### <a name="sfcertrotationnotexpired"></a>Service Fabric with certificates that aren't expired

1. Open the **Clusterconfig.json** file for editing, and find the following section. If a secondary thumbprint is defined, go to [Clean up old Service Fabric certificates](#cleanupoldsfcerts) before you go any further.

    ```json
    "security": {
        "metadata":  "The Credential type X509 indicates this cluster is secured using X509 Certificates. 
        The thumbprint format is - d5 ec 42 3b 79 cb e5 07 fd 83 59 3c 56 b9 d5 31 24 25 42 64.",
        "ClusterCredentialType":  "X509",
        "ServerCredentialType":  "X509",
        "CertificateInformation":  {
            "ClusterCertificate":  {
                                       "X509StoreName":  "My",
                                        "Thumbprint": "*Old server thumbprint(Star/SF)*"
                                   },
            "ServerCertificate":   {
                                        "X509StoreName":  "My",
										"Thumbprint": "*Old server thumbprint(Star/SF)*"
                                   },
            "ClientCertificateThumbprints":  [
                                       {
                                            "CertificateThumbprint": "*Old client thumbprint*",
                                            "IsAdmin":  true
                                       }
                                             ]
                                   }
                },
    ```

2. Replace that section in the file with following section.

    ```json
    "security":  {
        "metadata":  "The Credential type X509 indicates this cluster is secured using X509 Certificates. 
        The thumbprint format is - d5 ec 42 3b 79 cb e5 07 fd 83 59 3c 56 b9 d5 31 24 25 42 64.",
        "ClusterCredentialType":  "X509",
        "ServerCredentialType":  "X509",
        "CertificateInformation":  {
            "ClusterCertificate":  {
                                       "X509StoreName":  "My",
                                        "Thumbprint": "*New server thumbprint(Star/SF)*",
                                        "ThumbprintSecondary": "Old server thumbprint(Star/SF)"
                                   },
            "ServerCertificate":   {
                                        "X509StoreName":  "My",
										"Thumbprint": "*New server thumbprint(Star/SF)*",
                                        "ThumbprintSecondary": "Old server thumbprint(Star/SF)"
                                   },
            "ClientCertificateThumbprints":  [
                                       {
                                            "CertificateThumbprint": "*Old client thumbprint*",
                                            "IsAdmin":  false
                                       },
                                       {
                                            "CertificateThumbprint": "*New client thumbprint*",
                                            "IsAdmin":  true
                                       }
                                             ]
                                   }
                },
    ```

3. Edit the new and old thumbprint values. 

4. Change clusterConfigurationVersion to the new version, for example 2.0.0.

    ```json
    {
    "name": "Dynamics365Operations",
    "clusterConfigurationVersion": "2.0.0",
    "apiVersion": "10-2017",
    ```
    
5. Save the new ClusterConfig.json file.

6. Run the following PowerShell command.

    ```powershell
    # Connect to the Service Fabric cluster
    Connect-ServiceFabricCluster

    # Get path of ClusterConfig.json for following command
    # Note that after running the following command, you need to manually cancel using the red button (Stop Operation) in Windows PowerShell ISE or Ctrl+C in Windows PowerShell, otherwise you will receive the following notification, "Start-ServiceFabricClusterConfigurationUpgrade : Operation timed out.". Be aware that the upgrade will proceed.
    Start-ServiceFabricClusterConfigurationUpgrade -ClusterConfigPath ClusterConfig.json

    # If you are using a single Microsoft SQL Server Reporting Services node, use UpgradeReplicaSetCheckTimeout to skip PreUpgradeSafetyCheck check, otherwise it will timeout
    Update-ServiceFabricClusterUpgrade -UpgradeReplicaSetCheckTimeoutSec 30
    
    # To monitor the status of the upgrade, run the following and note UpgradeState and UpgradeReplicaSetCheckTimeout
    Get-ServiceFabricClusterUpgrade
    
    # While monitoring the status of the upgrade, if UpgradeReplicaSetCheckTimeout was reset to the default (example 49710.06:28:15), run the following command again
    Update-ServiceFabricClusterUpgrade -UpgradeReplicaSetCheckTimeoutSec 30
    
    # When UpgradeState shows RollingForwardCompleted, the upgrade is finished
    ```

### Service Fabric with or without expired certificates (cluster not accessible)

Continue this process following [Troubleshoot on-premises deployments](troubleshoot-on-prem.md#clean-up-an-existing-environment-and-redeploy).

## Update the LocalAgent certificate

You must reinstall the LocalAgent if:

- You changed the service fabric cluster/server certificate.
- You changed the service fabric client certificate.
- You changed the LocalAgent certificate.

1. Run the following PowerShell command on one of the Orchestrator nodes.

    ```powershell
    .\LocalAgentCLI.exe Cleanup <path of localagent-config.json>
    ```

2. Run the following PowerShell command and note the new LocalAgent thumbprint.

    ```powershell
    .\Get-AgentConfiguration.ps1 -ConfigurationFilePath .\ConfigTemplate.xml
    ```

3. Follow the steps in [Configure LCS connectivity for the tenant](setup-deploy-on-premises-pu12.md#configurelcs).

	> [!NOTE] 
	> If you receive the error **Update to existing credential with KeyId '\<key\>' is not allowed**, follow the instructions in [Error: "Updates to existing credential with KeyId '<key>' is not allowed"](troubleshoot-on-prem.md#error-updates-to-existing-credential-with-keyid-key-is-not-allowed).

4. Continue with [Configure a connector and install an on-premises local agent](setup-deploy-on-premises-pu12.md#configureconnector), specifically the following changes:

	- Client certificate thumbprint
	- Server certificate thumbprint
	- Tenant service principle certificate thumbprint

## Update your current deployment configuration

Because you've updated your certificates, the configuration file that is present in your environment is outdated and must be manually updated. Otherwise, the cleanup job will probably fail. (This manual update must be done just this one time.)

1. Open your configuration file. You can find the location of this file by running the following command.

    ```sql
    select Location from DeploymentInstanceArtifact where AssetId='config.json' and DeploymentInstanceId = 'LCSENVIRONMENTID'
    ```

    > [!NOTE]
    > Replace **LCSENVIRONMENTID** with the ID of your environment. You can obtain this ID from the page for your environment in LCS. 

    The beginning of the file should resemble the following example.

    ```json
    {
    "serviceFabric": {
        "connectionEndpoint": "192.168.8.22:19000",
        "clusterId": "Orch",
        "certificateSettings": {
        "serverCertThumbprint": "Old server thumbprint(Star/SF)",
        "clientCertThumbprint": "Old client thumbprint"
        }
    },
    ```

2. Replace the **serverCertThumprint** and **clientCertThumbprint** values with the new thumbprints.

    ```json
    {
    "serviceFabric": {
        "connectionEndpoint": "192.168.8.22:19000",
        "clusterId": "Orch",
        "certificateSettings": {
        "serverCertThumbprint": "New server thumbprint(Star/SF)",
        "clientCertThumbprint": "New client thumbprint"
        }
    },
    ```

3. Save and close the file. Remember to close any programs that are accessing this network location. Otherwise, the cleanup process might fail.

## Update deployment settings in LCS

> [!NOTE]
>  Note that the Client, Data Signing, and Encipherment certificates will only be replaced. You will also need to recreate the Credentials.json file, as described in [Encrypt credentials](setup-deploy-on-premises-pu12.md#encryptcred).
>
> Before you continue, you need to make a backup of the local Dynamics database.

1. In LCS, select the "Full Details" link for the environment where you want to change the certificates.

2. Select **Maintain** and then select **Update Settings**.

	![Apply update settings](media/addf4f1d0c0a86d840a6a412f774e474.png)

3. Change the thumbprints to the new thumbprints that you previously configured. (You can find them in the ConfigTemplate.xml file in the InfrastructureScripts folder.)

	![Deployment settings thumbprint](media/07da4d7e02f11878ee91c61b4f561a50.png)

	![Deployment settings thumbprint](media/785caaf4ee652d66c0d88cf615a57e26.png)

4. Select **Prepare**.

5. After downloading and preparation is complete, the **Update environment** button will display.

	![Update environment button](media/0a9d43044593450f1a828c0dd7698024.png)

6. Select **Update environment** to start updating your environment.

7. During the update, the environment will be unavailable.

8. After the environment is successfully updated with the new certificates, you can view the new thumbprints in Service Fabric Cluster Explorer. The names of the thumbprints in Service Fabric Explorer might differ from the names in LCS. However, the values should be the same.

	Here is an example of how the name of the same thumbprint might differ.

	![Deployment settings thumbprint example](media/038173714b2fb6cf12acc4bda2a3dde5.png)

	![Deployment settings thumbprint example](media/642f6434da9cdeac3651b765acca08fa.png)

## Update other certificates as needed

1. Always check if the SQL server certificate has expired. For more information, see [Set up SQL Server](https://docs.microsoft.com/dynamics365/unified-operations/dev-itpro/deployment/setup-deploy-on-premises-pu12#setupsql).

2. Check to be sure that the Active Directory Federation Service (ADFS) certificate has not expired.

## <a name="cleanupoldsfcerts"></a>Clean up old Service Fabric certificates

This procedure should be completed either after a successful certificate rotation or before the next certificate rotation.

1. Remove the old/secondary thumbprints from the cluster configuration. After you've removed them, the appropriate section should resemble the following example.

    ```json
    "security": {
        "metadata":  "The Credential type X509 indicates this is cluster is secured using X509 Certificates.
        The thumbprint format is - d5 ec 42 3b 79 cb e5 07 fd 83 59 3c 56 b9 d5 31 24 25 42 64.",
        "ClusterCredentialType":  "X509",
        "ServerCredentialType":  "X509",
        "CertificateInformation":  {
            "ClusterCertificate":  {
                                       "X509StoreName":  "My",
                                        "Thumbprint": "server thumbprint(Star/SF)"
                                   },
            "ServerCertificate":   {
                                        "X509StoreName":  "My",
										"Thumbprint": "server thumbprint(Star/SF)"
                                   },
            "ClientCertificateThumbprints":  [
                                       {
                                            "CertificateThumbprint": "client thumbprint",
                                            "IsAdmin":  true
                                       }
                                             ]
                                   }
                },
    ```

1. Follow steps 4 through 6 in the [Service Fabric with certificates that are not expired](#sfcertrotationnotexpired) section earlier in this topic. 

## <a name="aftercertrotation"></a> After certificate rotation

### Data encryption certificate

This certificate is used to encrypt data stored in the database. By default there are certain fields that are encrypted with this certificate, you can check those fields [here](https://docs.microsoft.com/dynamics365/fin-ops-core/dev-itpro/database/dbmovement-scenario-goldenconfig#document-the-values-of-encrypted-fields). However, our API can be used to encrypt other fields that customers deem should be encrypted. 

In Platform update 33 and later, the batch job that is named "Encrypted data rotation system job that needs to run at off hours when the data encryption certificate rotated" will use the newly rotated certificate to re-encrypt data. This batch job crawls through your data to re-encrypt all the encrypted data by using the new certificate. It will run for two hours per day for three consecutive days. Depending on the amount of data, the batch job might be able to finish running in less time.

> [!WARNING]
> Make sure that the old Data Encryption certificate is not removed before all encrypted data has been re-encrypted and it has not expired. Otherwise, this could lead to data loss.