---
title: Persisting files in Azure Cloud Shell (Preview) | Microsoft Docs
description: Walkthrough of how Azure Cloud Shell persists files.
services: 
documentationcenter: ''
author: jluk
manager: timlt
tags: azure-resource-manager
 
ms.assetid: 
ms.service: 
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.devlang: na
ms.topic: article
ms.date: 05/10/2017
ms.author: juluk
---

# Persisting Files in Azure Cloud Shell
On initial start, Azure Cloud Shell asks for your subscription to create an LRS storage account and Azure file share for you.

![](media/storage-prompt.png)

### Three resources will be created on your behalf in a supported region nearest to you:
1. Resource Group named: `cloud-shell-storage-<region>`
2. Storage Account named: `cs-uniqueGuid`
3. File Share named: `cs-<user>-<domain>-com-uniqueGuid`

This file share will mount as `clouddrive` under your $Home directory. This file share is also used to store a 5-GB image created for you that automatically updates and persists your $Home directory. This is a one-time action and automatically mounts for subsequent sessions.

### Cloud Shell persists files with both methods below:
1. Create a disk image of your $Home directory to persist files within $Home. 
This disk image is saved in your specified file share as `<User>.img` at `fileshare.storage.windows.net/fileshare/.cloudconsole/<User>.img`

2. Mount specified file share as `clouddrive` in your $Home directory for direct file share interaction. 
`/Home/<User>/clouddrive` is mapped to `fileshare.storage.windows.net/fileshare`.
 
## Using clouddrive
Cloud Shell allows users to run a command called `clouddrive` that enables manually updating the file share mounted to Cloud Shell.
![](media/clouddrive-h.png)

## Mount a new clouddrive

### Pre-requisites for manual mounting
Cloud Shell will create a storage account and file share for you on first launch, however you may update the file share with the `clouddrive mount` command.

If mounting an existing file share, storage accounts must be:
1. LRS or GRS to support file shares.
2. Located in your assigned region. When onboarding, the region you are assigned to is listed in the resource group name `cloud-shell-storage-<region>`.

### Supported storage regions
Azure Files must reside in the same region as the machine being mounted to. Cloud Shell machines exist in the below regions:
|Area|Region|
|---|---|
|Americas|East US, South Central US, West US|
|Europe|North Europe, West Europe|
|Asia Pacific|India Central, Southeast Asia|

### Mount command

> [!NOTE]
> If mounting a new file share, a new user image will be created for your $Home directory as your previous $Home image is held in the previous file share.

1. Run `clouddrive mount` with the following parameters <br>

```
clouddrive mount -s mySubscription -g myRG -n storageAccountName -f fileShareName
```

To see more details run `clouddrive mount -h`: <br>
![](media/mount-h.png)

## Unmount clouddrive
You may unmount a file share mounted to Cloud Shell at any time. However, Cloud Shell requires a mounted file share so you will be prompted to create and mount a new file share on next session if removed.

To detach a file share from Cloud Shell:
1. Run `clouddrive unmount`
2. Acknowledge and confirm prompts

Your file share will continue to exist unless manually deleted. Cloud Shell will no longer search for this file share on subsequent sessions.

To see more details run `clouddrive mount -h`: <br>
![](media/unmount-h.png)

> [!WARNING]
> This command will not delete any resources. However, manually deleting the resource group, storage account, or file share mapped to Cloud Shell will erase your $Home directory disk image and any files in your file share. This cannot be undone.

## List clouddrive
To discover which file share is mounted as `clouddrive`:
1. Run `df` 

The filepath to clouddrive will show your storage account name and fileshare in the url.

`//storageaccountname.file.core.windows.net/filesharename`

```
justin@Azure:~$ df
Filesystem                                          1K-blocks   Used  Available Use% Mounted on
overlay                                             29711408 5577940   24117084  19% /
tmpfs                                                 986716       0     986716   0% /dev
tmpfs                                                 986716       0     986716   0% /sys/fs/cgroup
/dev/sda1                                           29711408 5577940   24117084  19% /etc/hosts
shm                                                    65536       0      65536   0% /dev/shm
//mystoragename.file.core.windows.net/fileshareName 5368709120    64 5368709056   1% /home/justin/clouddrive
justin@Azure:~$
```

## Upload or download local files
Use the Azure portal to manage transferring local files to or from the file share.
Updating files from within Cloud Shell reflects in the File Storage GUI on blade refresh.

1. Navigate to the mounted fileshare
![](media/touch-txt-storage.png)
2. Select target file in Portal
3. Hit "Download"
![](media/download-storage.png)

If you need to download a file that exists outside of `clouddrive`:
1. Copy the file to `/<User>/clouddrive` <br>
2. Follow [previous steps](#upload-or-download-local-files) <br>

## Cloud Shell tagging
Cloud Shell adds a "tag" to mounted storage accounts using the format: <br>

| Key | Value |
|:-------------:|:-------------:|
|cloud-console-files-for-user@domain.com|fileshareName|

Use these tags to see which users map to certain file shares and where certain $Home images can be found.

## Next steps
[Cloud Shell Quickstart](quickstart.md) 
[Learn about Azure File storage](https://docs.microsoft.com/azure/storage/storage-introduction#file-storage) 
[Learn about Storage tags](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags) 