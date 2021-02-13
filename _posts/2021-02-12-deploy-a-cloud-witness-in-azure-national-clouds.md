---
date: 2021-02-12 00:00:00 -05:00
layout: post
categories: Azure FailoverCluster WindowsServer
title: Deploy a Cloud Witness in Azure national clouds 
---

Deploying a [Cloud Witness](https://docs.microsoft.com/windows-server/failover-clustering/deploy-cloud-witness) for [Windows Server Failover Clusters](https://docs.microsoft.com/windows-server/failover-clustering/failover-clustering-overview) is a great quorum option. However, if storage account which is to be used as the Cloud Witness is in a [national cloud](https://docs.microsoft.com/azure/active-directory/develop/authentication-national-cloud), the the [Create Cluster Quorum Wizard](https://docs.microsoft.com/windows-server/failover-clustering/deploy-cloud-witness#to-configure-cloud-witness-as-a-quorum-witness) and [PowerShell cmdlets](https://docs.microsoft.com/windows-server/failover-clustering/deploy-cloud-witness#configuring-cloud-witness-using-powershell) may not work. Fortunately the [CIM method](https://docs.microsoft.com/previous-versions/windows/desktop/cluswmiext/mscluster-clusterservice-createcloudwitness) used by wizard and PowerShell cmdlets allow us to successfully create the Cloud Witness.

>### Assumptions
>
>- The commands are executed on a cluster node which will use the Cloud Witness
>- The Az.Accounts module is installed

1. Get the cluster service

    ```PowerShell
    $cluster = Get-CimInstance -Namespace root/MSCluster -ClassName MSCluster_ClusterService
    ```

1. Get the storage endpoint URL for the desired cloud

    ```PowerShell
    $endpoint = Get-AzEnvironment -Name AzureUSGovernment | Select-Object -ExpandProperty StorageEndpointSuffix
    ```

1. Create a [hashtable](https://docs.microsoft.com/powershell/scripting/learn/deep-dives/everything-about-hashtable) with the required arguments. Obtain the storage account name and access key following [Manage storage account access keys](https://docs.microsoft.com/azure/storage/common/storage-account-keys-manage?tabs=azure-portal#view-account-access-keys).

    ```PowerShell
    $createCloudWitnessArguments = @{
        AccountName = '<Storage account name>'
        AccountKey = '<key>'
        EndpointInfo = $endpoint
    }
    ```

1. Call the `CreateCloudWitness` method

    ```PowerShell
    Invoke-CimMethod -InputObject $cluster -MethodName CreateCloudWitness -Arguments $createCloudWitnessArguments
    ```

There you have it. I created a complete script which utilizes this method called [New-CloudWitness.ps1](https://gist.github.com/randomnote1/78e57468abef1ef739a2322ffe2c7143).
