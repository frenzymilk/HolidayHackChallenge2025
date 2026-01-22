---
icon: material/text-box-outline
---

# Act 1: Blob storage challenge in the neighborhood

**Difficulty**: :fontawesome-solid-star::fontawesome-regular-star::fontawesome-regular-star::fontawesome-regular-star::fontawesome-regular-star:<br/>

## Objective

!!! question "Request"
    Help the Goose Grace near the pond find which Azure Storage account has been misconfigured to allow public blob access by analyzing the export file.

??? quote "Grace"
    The Neighborhood HOA uses Azure storage accounts for various IT operations.
    You've been asked to audit their storage security configuration to ensure no sensitive data is publicly accessible.
    Recent security reports suggest some storage accounts might have public blob access enabled, creating potential data exposure risks.

Let's audit the access policies for Azure storage accounts  to ensure no sensitive data is publicly accessible.


## Solution

1. explore the account you have access to <br/>
`az account show`<br/>
![Explore](../img/objectives/o8/o8_1.png)

2. Examine the azure storage accounts:<br/>
`az storage account list | less`<br/>

One shows the most common misconfiguration, which is a publicly accessible storage account.<br/>
![Common misconfiguration](../img/objectives/o8/o8_2.png)

3. Examine that storage account:<br/>
`az storage account show --name neighboorhood2`<br/>
![Examine account](../img/objectives/o8/o8_3.png)

4. List containers in neighborhood2:<br/>
`az storage container list --account-name neighborhood2 --auth-mode login|less`<br/>
![List Containers](../img/objectives/o8/o8_4.png)
One of them is publicly accessible

5.  Look at blob list in the public container for neighborhood2<br/>
`az storage blob list --container-name 'public' --account-name neighborhood2 --auth-mode login -o table`<br/>
![Blob List](../img/objectives/o8/o8_5.png)

We see an interesting file: `admin_credentials.txt`<br/>

6.  Download the credentials file and read it<br/>
`az storage blob download --container-name 'public' --name admin_credentials.txt   --account-name neighborhood2  --file  /dev/stdout | less`<br/>

![Read file](../img/objectives/o8/o8_6.png)
