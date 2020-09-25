---
title:  "Kubernetes on AzureStackHub with AKS-Engine and external AD/DNS intigration"
date:   2020-09-24 16:30:00 +0200
categories: [Azure]
tags: ["Azure", "AzureStackHub"]
---
We can use AKS Engine to deploy a Kubernetes-Cluster on Azure Stack Hub. So i started to create a Standard Kubernetes-Cluster on Azure Stack Hub, but today we are talking about the ability to use a custom Windows-DNS and Windows-ADS with aks-engine.
First of all, we need to create a Virtual Network, two subnets and a ressource group.

## What is AKS Engine
AKS Engine provides convenient tooling to quickly bootstrap Kubernetes clusters on Azure. By leveraging ARM (Azure Resource Manager), AKS Engine helps you create, destroy and maintain clusters provisioned with basic IaaS resources in Azure. 

## Check your Quotas
You  need to check if you have enough space left to deploy your servers, if you choose to deploy 3 Master and 3 Worker Nodes with a VM Size of [Standard_DS2_v2](https://docs.microsoft.com/en-us/azure-stack/user/azure-stack-vm-sizes?view=azs-2005){:target="_blank"}, you should have space for 12 CPUs. 
If your subscription doesn’t contain enough quotas, your deployment will fail.

## Create a Service Priciple (service account)
You need to create an Azure service principal for the deployment. I used one with Password-based authentication.

```bash
az ad sp create-for-rbac --name $AKS-SP
```

please store the service principal name, id and password at a save place. You can create your own keyvault for that. Cause we need the ID and password later.
The service principal needs enough privileges to customize the virtual networks and the subnets as well the privileges on the ressource group to add ressources and make changes.

## Create a Virtual Network
If you don't already have a virtual network for your AKS installation, you need to create one. You can decide where to create your virtual network, maybe you already have a network resource group. If you already habe a vnet, you can skip this step.

```bash
az network vnet create --address-prefixes 10.222.0.0/16 --name $k8sVnetName --resource-group $RG-NETWORK --subnet-name $KubernetesMasterSubnet --subnet-prefixes 10.222.1.0/25
```

### Create a Subnet
Here you have to decide on the network size and think about the names of the subnet. The following are really only examples of what it might look like.

```bash
az network vnet subnet create -g $RG-NETWORK --vnet-name $k8sVnetName -n $KubernetesMasterSubnet --address-prefixes 10.222.1.0/25
az network vnet subnet create -g $RG-NETWORK --vnet-name $k8sVnetName -n $KubernetesWorkerSubnet --address-prefixes 10.222.1.128/25
```

## Create a Ressource Group
We have to create a Ressource Group for our deployment of AKS if we don't have already a RG.

```bash
az group create -l $LOCATION -g $RG-NAME-k8s
```

## Create the deployment file
If we have taken the above steps and our service principle has the right permissions, e.g. at subscription level, we can start to prepare the deployment.
For that, we need to install an [aks-engine version](https://github.com/Azure/aks-engine/releases){:target="_blank"}, after that we have to create a deployment-file [kubernetes-azurestack.json](https://github.com/Azure/aks-engine/blob/master/examples/azure-stack/kubernetes-azurestack.json){:target="_blank"}.

### Deploymentfile without a custom DNS-Server
```json
{
    "apiVersion": "vlabs",
    "location": "$LOCATION",
    "properties": {
        "orchestratorProfile": {
            "orchestratorType": "Kubernetes",
            "orchestratorRelease": "1.17",
            "orchestratorVersion": "1.17.5",
            "kubernetesConfig": {
                "cloudProviderBackoff": true,
                "cloudProviderBackoffRetries": 1,
                "cloudProviderBackoffDuration": 30,
                "cloudProviderRateLimit": true,
                "cloudProviderRateLimitQPS": 3,
                "cloudProviderRateLimitBucket": 10,
                "cloudProviderRateLimitQPSWrite": 3,
                "cloudProviderRateLimitBucketWrite": 10,
                "kubernetesImageBase": "mcr.microsoft.com/k8s/azurestack/core/",
                "useInstanceMetadata": false,
                "networkPlugin": "kubenet",
                "kubeletConfig": {
                    "--node-status-update-frequency": "1m",
                    "--kube-reserved": "memory=500Mi",
                    "--system-reserved": "memory=500Mi",
                    "--eviction-hard": "memory.available<500Mi"
                },
                "controllerManagerConfig": {
                    "--node-monitor-grace-period": "5m",
                    "--pod-eviction-timeout": "5m",
                    "--route-reconciliation-period": "1m"
                },
                "privateCluster": {
                    "enabled": true
                },
                "etcdDiskSizeGB": "32"
            }
        },
        "customCloudProfile": {
            "portalURL": "https://$AzureStackHubURL"
        },
        "featureFlags": {
            "enableTelemetry": true
        },
        "masterProfile": {
            "dnsPrefix": "master",
            "distro": "aks-ubuntu-16.04",
            "count": 3,
            "vmSize": "Standard_DS2_v2",
            "vnetSubnetId": "/subscriptions/$SUBSCRIPTIONID/resourceGroups/$RG-NETWORK/providers/Microsoft.Network/virtualNetworks/$k8sVnetName/subnets/$KubernetesMasterSubnet",
            "firstConsecutiveStaticIP": "..REDACTED...",
            "customVMTags": {
                "Application": "k8s Test"
            }
        },
        "agentPoolProfiles": [
            {
                "name": "worker",
                "count": 3,
                "vmSize": "Standard_DS2_v2",
                "distro": "aks-ubuntu-16.04",
                "availabilityProfile": "AvailabilitySet",
                "AcceleratedNetworkingEnabled": false,
                "vnetSubnetId": "/subscriptions/$SUBSCRIPTIONID/resourceGroups/$RG-NETWORK/providers/Microsoft.Network/virtualNetworks/$k8sVnetName/subnets/$KubernetesWorkerSubnet",
                "customVMTags": {
                  "Application": "k8s Test"
                }
            }
        ],
        "linuxProfile": {
            "adminUsername": "$ROOTUserName",
            "ssh": {
                "publicKeys": [
                    {
                        "keyData": "$ROOTUserSSHKey"
                    }
                ]
            }
        },
        "servicePrincipalProfile": {
            "clientId": "$SPID",
            "secret": "$SPPassword"
        }
    }
}
```

### Deploymentfile with custom DNS-Server

```json
{
    "apiVersion": "vlabs",
    "location": "$LOCATION",
    "properties": {
        "orchestratorProfile": {
            "orchestratorType": "Kubernetes",
            "orchestratorRelease": "1.17",
            "orchestratorVersion": "1.17.5",
            "kubernetesConfig": {
                "cloudProviderBackoff": true,
                "cloudProviderBackoffRetries": 1,
                "cloudProviderBackoffDuration": 30,
                "cloudProviderRateLimit": true,
                "cloudProviderRateLimitQPS": 3,
                "cloudProviderRateLimitBucket": 10,
                "cloudProviderRateLimitQPSWrite": 3,
                "cloudProviderRateLimitBucketWrite": 10,
                "kubernetesImageBase": "mcr.microsoft.com/k8s/azurestack/core/",
                "useInstanceMetadata": false,
                "networkPlugin": "kubenet",
                "kubeletConfig": {
                    "--node-status-update-frequency": "1m",
                    "--kube-reserved": "memory=500Mi",
                    "--system-reserved": "memory=500Mi",
                    "--eviction-hard": "memory.available<500Mi"
                },
                "controllerManagerConfig": {
                    "--node-monitor-grace-period": "5m",
                    "--pod-eviction-timeout": "5m",
                    "--route-reconciliation-period": "1m"
                },
                "privateCluster": {
                    "enabled": true
                },
                "etcdDiskSizeGB": "32"
            }
        },
        "customCloudProfile": {
            "portalURL": "https://$AzureStackHubURL"
        },
        "featureFlags": {
            "enableTelemetry": true
        },
        "masterProfile": {
            "dnsPrefix": "master",
            "distro": "aks-ubuntu-16.04",
            "count": 3,
            "vmSize": "Standard_DS2_v2",
            "vnetSubnetId": "/subscriptions/$SUBSCRIPTIONID/resourceGroups/$RG-NETWORK/providers/Microsoft.Network/virtualNetworks/$k8sVnetName/subnets/$KubernetesMasterSubnet",
            "firstConsecutiveStaticIP": "..REDACTED...",
            "customVMTags": {
                "Application": "k8s Test"
            }
        },
        "agentPoolProfiles": [
            {
                "name": "worker",
                "count": 3,
                "vmSize": "Standard_DS2_v2",
                "distro": "aks-ubuntu-16.04",
                "availabilityProfile": "AvailabilitySet",
                "AcceleratedNetworkingEnabled": false,
                "vnetSubnetId": "/subscriptions/$SUBSCRIPTIONID/resourceGroups/$RG-NETWORK/providers/Microsoft.Network/virtualNetworks/$k8sVnetName/subnets/$KubernetesWorkerSubnet",
                "customVMTags": {
                  "Application": "k8s Test"
                }
            }
        ],
        "linuxProfile": {
            "adminUsername": "$ROOTUserName",
            "ssh": {
                "publicKeys": [
                    {
                        "keyData": "$ROOTUserSSHKey"
                    }
                ]
            },
            "customSearchDomain": {
                "name": "$ADDOMAINNAME",
                "realmUser": "$ADSERVICEACCOUNT",
                "realmPassword": "$ADSERVICEACCOUNTPASSWORD"
            },
            "customNodesDNS": {
                "dnsServer": "$CustomDNSIP"
            }
        },
        "servicePrincipalProfile": {
            "clientId": "$SPID",
            "secret": "$SPPassword"
        }
    }
}
```

### Deployment

```bash
aks-engine deploy -f --azure-env AzureStackCloud --api-model kubernetes-azurestack.json --location $LOCATION --resource-group $RG-NAME-k8s --client-id $SPID --client-secret $SPPassword --subscription-id $SUBSCRIPTIONID --output-directory $RG-NAME-k8s
```

### Hotfix

with the above deploymentfile you can start a deployment with a custom WindowsDNS, but since we have backwords compatibility for Windows 2000 and below, the Active-Directory computer section is limited to 15 chars, so we needed a ugly hotfix.
We needed to overwrite the fallowing file Node by node, so if we checked the log in /var/log/azure/cluster-provision.sh the script ist waiting for /opt/azure/containers/setup-custom-search-domains.sh to get ready, cause the #EOF isn't included in the script at the moment.
So we overwrote the script node by node, but only, when the current server is successfully joined the AD.

```sh
cat <<EOF > /opt/azure/containers/setup-custom-search-domains.sh
#!/bin/bash
set -x

source /opt/azure/containers/provision_source.sh

echo "  dns-search $ADDOMAINNAME" | tee -a /etc/network/interfaces.d/50-cloud-init.cfg
systemctl_restart 20 5 10 networking
wait_for_apt_locks
retrycmd 10 5 120 apt-get update
wait_for_apt_locks
retrycmd 10 5 120 apt-get -y install realmd sssd sssd-tools samba-common samba samba-common python2.7 samba-libs packagekit
wait_for_apt_locks
echo "$ADSERVICEACCOUNTPASSWORD" | realm join -U $ADSERVICEACCOUNT@$(echo "$ADDOMAINNAME" | tr /a-z/ /A-Z/) $(echo "$ADDOMAINNAME" | tr /a-z/ /A-Z/)
#EOF
EOF
```

You need to do it on all master and worker Nodes. Step by Step.

## If your deployment was successful
If the deployment is successful, all files from the output directory $RG-NAME-k8s must be placed in a secure location, using a keyvault and a storage account with blob storage is a nice place. 
Cause some files containing secrets, they should be stored safely. Microsoft recommends an extra [virtual instance](https://docs.microsoft.com/en-us/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-linux?view=azs-2005){:target="_blank"}.

These files from the folder are needed for later changes, upgrades, scaling and other changes of the cluster.

## Problem or Feature i'm not sure
I don;t know if backwards compatibility is a feature, but i think its a bug, that an older AD doesn't work with aks-engine, so opened an issue for my problem, cause I don't know how to solve it, concerning the backwards compatibility of the AD.

## Suggestions welcome
I am trying to do my best, if you have a suggestion for improvement, feel free to write me a message.
I need good and negative criticism to improve myself, maybe you have a solution for the problem or need help yourself.


#### don't trust me...
I do not know what I am doing, so I cannot take responsibility for the accuracy of the information provided here