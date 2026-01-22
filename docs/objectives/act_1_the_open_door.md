---
icon: material/text-box-outline
---

# Act 1: The open door

**Difficulty**: :fontawesome-solid-star::fontawesome-regular-star::fontawesome-regular-star::fontawesome-regular-star::fontawesome-regular-star:<br/>


## Objective

!!! question "Request"
    Help Goose Lucas in the hotel parking lot find the dangerously misconfigured Network Security Group rule that's allowing unrestricted internet access to sensitive ports like RDP or SSH.

??? quote "Lucas"
    Please make sure the towns Azure network is secured properly.<br/>
    The Neighborhood HOA uses Azure for their IT infrastructure.<br/>
    Audit their network security configuration to ensure production systems aren't exposed to internet attacks.<br/>
    They claim all systems are properly protected, but you need to verify there are no overly permissive NSG rules.

## Solution

- See the resources group in json format<br/>
`az group list -o json`<br/>
![Resources](../img/objectives/o10/o10_1.png)

- Show it in table format now<br/>
`az group list -o table`<br/>
![[Capture d'écran 2025-12-11 133043.png]]

- Let's list all NSGs across resource groups<br/>
`az network nsg list -o table`<br/>
![Resources](../img/objectives/o10/o10_2.png)

- Inspect the security group "web" and show the details<br/>
`az network nsg show  --name nsg-web-eastus --resource-group theneighborhood-rg1 -o json | less`<br/>

![Resources](../img/objectives/o10/o10_3.png)

- List the nsg rules for the management group<br/>
`az network nsg rule list  --name nsg-mgmt-eastus --resource-group theneighborhood-rg2 -o json | less`<br/>

![Resources](../img/objectives/o10/o10_4.png)

- Explore other groups<br/>
`az network nsg rule list  --name nsg-production-eastus --resource-group theneighborhood-rg1 -o json | less`<br/>

![Resources](../img/objectives/o10/o10_5.png)

- Analyze the suspicious rule:<br/>
`az network nsg rule show  --nsg-name nsg-production-eastus --resource-group theneighborhood-rg1 --name  Allow-RDP-From-Internet -o json`<br/>
![Resources](../img/objectives/o10/o10_6.png)
