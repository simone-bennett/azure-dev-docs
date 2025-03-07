---
title: Tutorial - Configure Azure Kubernetes Service (AKS) clusters in Azure using Ansible
description: Learn how to use Ansible to create and manage an Azure Kubernetes Service cluster in Azure
keywords: ansible, azure, devops, bash, cloudshell, playbook, aks, container, aks, kubernetes
ms.topic: tutorial
ms.date: 06/17/2020
ms.custom: devx-track-ansible
---

# Tutorial: Configure Azure Kubernetes Service (AKS) clusters in Azure using Ansible

[!INCLUDE [ansible-28-note.md](includes/ansible-28-note.md)]

[!INCLUDE [open-source-devops-intro-aks.md](../includes/open-source-devops-intro-aks.md)]

AKS can be configured to use [Azure Active Directory (AD)](/azure/active-directory/) for user authentication. Once configured, you use your Azure AD authentication token to sign into the AKS cluster. The RBAC can be based on a user's identity or directory group membership.

In this article, you learn how to:

> [!div class="checklist"]
>
> * Create an AKS cluster
> * Configure an AKS cluster

## Prerequisites

[!INCLUDE [open-source-devops-prereqs-azure-subscription.md](../includes/open-source-devops-prereqs-azure-subscription.md)]
[!INCLUDE [open-source-devops-prereqs-create-service-principal.md](../includes/open-source-devops-prereqs-create-service-principal.md)]
[!INCLUDE [ansible-prereqs-cloudshell-use-or-vm-creation2.md](includes/ansible-prereqs-cloudshell-use-or-vm-creation2.md)]

## Create a managed AKS cluster

The sample playbook creates a resource group and an AKS cluster within the resource group.

Save the following playbook as `azure_create_aks.yml`:

```yml
- name: Create Azure Kubernetes Service
  hosts: localhost
  connection: local
  vars:
    resource_group: myResourceGroup
    location: eastus
    aks_name: myAKSCluster
    username: azureuser
    ssh_key: "your_ssh_key"
    client_id: "your_client_id"
    client_secret: "your_client_secret"
    aks_version: aks_version
  tasks:
  - name: Create resource group
    azure_rm_resourcegroup:
      name: "{{ resource_group }}"
      location: "{{ location }}"
  - name: Create a managed Azure Container Services (AKS) cluster
    azure_rm_aks:
      name: "{{ aks_name }}"
      location: "{{ location }}"
      resource_group: "{{ resource_group }}"
      dns_prefix: "{{ aks_name }}"
      kubernetes_version: "{{aks_version}}"
      linux_profile:
        admin_username: "{{ username }}"
        ssh_key: "{{ ssh_key }}"
      service_principal:
        client_id: "{{ client_id }}"
        client_secret: "{{ client_secret }}"
      agent_pool_profiles:
        - name: default
          count: 2
          vm_size: Standard_D2_v2
      tags:
        Environment: Production
```

Before running the playbook, see the following notes:

- The first section within `tasks` defines a resource group named `myResourceGroup` within the `eastus` location.
- The second section within `tasks` defines an AKS cluster named `myAKSCluster` within the `myResourceGroup` resource group.
- For the `your_ssh_key` placeholder, enter your RSA public key in the single-line format - starting with "ssh-rsa" (without the quotes).
- For the `aks_version` placeholder, use the [az aks get-versions](/cli/azure/aks#az_aks_get_versions) command.

Run the playbook using [ansible-playbook](https://docs.ansible.com/ansible/latest/cli/ansible-playbook.html)

```bash
ansible-playbook azure_create_aks.yml
```

Running the playbook shows results similar to the following output:

```Output
PLAY [Create AKS] 

TASK [Gathering Facts] 
ok: [localhost]

TASK [Create resource group] 
changed: [localhost]

TASK [Create an Azure Container Services (AKS) cluster] 
changed: [localhost]

PLAY RECAP 
localhost                  : ok=3    changed=2    unreachable=0    failed=0
```

## Scale AKS nodes

The sample playbook in the previous section defines two nodes. You adjust the number of nodes by modifying the `count` value in the `agent_pool_profiles` block.

Save the following playbook as `azure_configure_aks.yml`:

```yml
- name: Scale AKS cluster
  hosts: localhost
  connection: local
  vars:
    resource_group: myResourceGroup
    location: eastus
    aks_name: myAKSCluster
    username: azureuser
    ssh_key: "your_ssh_key"
    client_id: "your_client_id"
    client_secret: "your_client_secret"
  tasks:
  - name: Scaling an existed AKS cluster
    azure_rm_aks:
        name: "{{ aks_name }}"
        location: "{{ location }}"
        resource_group: "{{ resource_group }}"
        dns_prefix: "{{ aks_name }}"
        linux_profile:
          admin_username: "{{ username }}"
          ssh_key: "{{ ssh_key }}"
        service_principal:
          client_id: "{{ client_id }}"
          client_secret: "{{ client_secret }}"
        agent_pool_profiles:
          - name: default
            count: 3
            vm_size: Standard_D2_v2
```

Before running the playbook, see the following notes:

- For the `your_ssh_key` placeholder, enter your RSA public key in the single-line format - starting with "ssh-rsa" (without the quotes).

Run the playbook using [ansible-playbook](https://docs.ansible.com/ansible/latest/cli/ansible-playbook.html)

```bash
ansible-playbook azure_configure_aks.yml
```

Running the playbook shows results similar to the following output:

```Output
PLAY [Scale AKS cluster] 

TASK [Gathering Facts] 
ok: [localhost]

TASK [Scaling an existed AKS cluster] 
changed: [localhost]

PLAY RECAP 
localhost                  : ok=2    changed=1    unreachable=0    failed=0
```

## Delete a managed AKS cluster

The sample playbook deletes an AKS cluster.

Save the following playbook as `azure_delete_aks.yml`:


```yml
- name: Delete a managed Azure Container Services (AKS) cluster
  hosts: localhost
  connection: local
  vars:
    resource_group: myResourceGroup
    aks_name: myAKSCluster
  tasks:
  - name:
    azure_rm_aks:
      name: "{{ aks_name }}"
      resource_group: "{{ resource_group }}"
      state: absent
  ```

Run the playbook using [ansible-playbook](https://docs.ansible.com/ansible/latest/cli/ansible-playbook.html)

```bash
ansible-playbook azure_delete_aks.yml
```

Running the playbook shows results similar to the following output:

```Output
PLAY [Delete a managed Azure Container Services (AKS) cluster] 

TASK [Gathering Facts] 
ok: [localhost]

TASK [azure_rm_aks] 

PLAY RECAP 
localhost                  : ok=2    changed=1    unreachable=0    failed=0
```

## Next steps

> [!div class="nextstepaction"]
> [Tutorial: Scale application in Azure Kubernetes Service (AKS)](/azure/aks/tutorial-kubernetes-scale)