---
icon: material/text-box-outline
---

# Act 1: Spare key

**Difficulty**: :fontawesome-solid-star::fontawesome-regular-star::fontawesome-regular-star::fontawesome-regular-star::fontawesome-regular-star:<br/>

## Objective

!!! question "Request"
    Help Goose Barry near the pond identify which identity has been granted excessive Owner permissions at the subscription level, violating the principle of least privilege.

??? quote "Barry"
    The Neighborhood HOA hosts a static website on Azure Storage.<br/>
    An admin accidentally uploaded an infrastructure config file that contains a long-lived SAS token.<br/>
    Use Azure CLI to find the leak and report exactly where it lives.

Find the location of a spare key left in the open on an Azure CLI session in "The Neighborhood" tenant

## Solution

Exploration: 
- List all the resource groups <br/>
`az group list -o table `<br/>
![Explore](../img/objectives/o9/o9_1.png)

- Find storage accounts in the neighborhood resource group<br/>
`az storage account list --resource-group rg-the-neighborhood -o table`<br/>
![Storage](../img/objectives/o9/o9_2.png)

- Explore the properties of the accounts available to determine if one of the storage contains a static website<br/>
`az storage blob service-properties show --account-name neighborhoodhoa --auth-mode login`<br/>
![Properties](../img/objectives/o9/o9_3.png)

- Identify what containers are available in the storage account and its public access levels<br/>
`az storage container list --account-name neighborhoodhoa --auth-mode login`<br/>
![Containers](../img/objectives/o9/o9_4.png)

- Explore the files in the static website container and look for files that should not be publicly accessible<br/>
`az storage blob list --container-name '$web' --account-name neighborhoodhoa --auth-mode login -o table`<br/>
![Containers](../img/objectives/o9/o9_5.png)

- Read the suspect the file<br/>
`az storage blob download --container-name '$web' --name iac/terraform.tfvars   --account-name neighborhoodhoa  --file  /dev/stdout | less`<br/>
![Containers](../img/objectives/o9/o9_6.png)