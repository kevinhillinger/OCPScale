---
title: Hybrid Cloud
has_children: true
---

# Tech Intensity Workshop - Azure Hybrid Cloud

## Before You Begin

### Student ID
You will receive a unique student ID prior to the workshop. This will be used throughout the labs to consistently name resources. The format of the identifier will be "tiw[workshop number][ student number]"; for example, "tiw63001". If you can't find your ID, ask the instructor.

### Default Credentials
Whenever a a resources is created, including VM credentials, accounts, etc. the following credentials should be used:

| Username  | Password         |
|-----------|------------------|
| azureuser | Azuret1workshop! |

### Regions

If you are using a Microsoft Azure subscription that was provided to you by Microsoft for the workshop, you are limited to a specific set of Microsoft Azure regions that you can use. An error may occur during provisioning of a resource if a chosen region is unsupported. Please use the following regions for the lab: 

- East US
- South Central US
- West Europe
- Southeast Asia
- West US 2
- West Central US

> NOTE: We recommend (unless explicitly required by a lab) to use the same region when creating resources.

## Labs
Below are the labs for the Tech intensity Workshop's Azure Hybrid Cloud.

### Lab 1 - Hybrid Identity
1. [AD Connect Infrastructure Setup](hybrid-identity/adconnect.md)
2. [AD Connect High Availability (Optional)](01_HybridCloud_IdentityLab02_ADConnectOptionalFeatures.md)
3. [AD Connect Publishing a SSO Application (Optional)](01_HybridCloud_IdentityLab03_SSOApp(Optional).md)

### Lab 2 - Networking
1. [Virtual Networks](03_HybridCloud_Networking_Lab01_VirtualNetworks.md)
2. [Load Balancer](03_HybridCloud_Networking_Lab02_LoadBalancer.md)
3. [Traffic Manager](03_HybridCloud_Networking_Lab03_TrafficManager.md)
4. [Network Watcher](03_HybridCloud_Networking_Lab04_NetworkWatcher.md)
5. [Hub and Spoke Challenge (Optional)](03_HybridCloud_Networking_Lab05_HubSpokeChallenge.md)

### Lab 3 - Migration
1. [Migration Lab 1 - Database Migration Service (DMS)](02_HybridCloud_Migration_Lab01_DMS.md)
2. [Migration Lab 2 - Azure Site Recovery (ASR)](02_HybridCloud_Migration_Lab02_ASR.md)
3. [Migration Lab 3 - Azure Policy](02_HybridCloud_Migration_Lab03_AzurePolicy.md)
4. [Migration Lab 4 - Azure Migrate (Optional)](HybridCloud/02_HybridCloud_Migration_Lab04_AzureMigrate.md)

### Lab 4 - Solution Design
5. [Case Study](04_Hybrid_Cloud_Hackathon_CaseStudy.md)


## Contributions
If you see any changes that need to be made (typos, new steps, etc.), we welcome any pull requests!

Thanks!
