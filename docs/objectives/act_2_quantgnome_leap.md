---
icon: material/text-box-outline
---

# Act 2: Quantgnome leap

**Difficulty**: :fontawesome-solid-star::fontawesome-solid-star::fontawesome-regular-star::fontawesome-regular-star::fontawesome-regular-star:<br/>


## Objective

!!! question "Request"
    Charlie in the hotel has quantum gnome mysteries waiting to be solved. What is the flag that you find?

??? quote "Charlie Goldner"
    I just spotted a mysterious gnome - he winked and vanished, or maybe he’s still here?<br/>
    Things are getting strange, and I think we’ve wandered into a quantum conundrum!<br/>
    If you help me unravel these riddles, we might just outsmart future quantum computers.<br/>
    Cryptic puzzles, quirky gnomes, and post-quantum secrets—will you leap with me?

A gnome appeared, then vanished.
We need to unravel quantum riddles.

## Solution


- Find the "PQC" executable<br/>
`find / pqc | grep pqc`<br/>
![Terminal output](../img/objectives/o17/o17_1.png)

- Run executable<br/>
`/usr/local/bin/pqc-keygen`<br/>
![Terminal output](../img/objectives/o17/o17_2.png)

- Examine keys<br/>
`/usr/local/bin/pqc-keygen`<br/>
![Terminal output](../img/objectives/o17/o17_3.png)

- Find the username for the ssh connection based on the pre existing ssh key:<br/>
-` more .ssh/id_rsa.pub `<br/>
![Terminal output](../img/objectives/o17/o17_4.png)<br/>

`ssh gnome1@pqc-server.com`<br/>

![Terminal output](../img/objectives/o17/o17_5.png)


- Round and round you go<br/>
![Terminal output](../img/objectives/o17/o17_6.png)

![Terminal output](../img/objectives/o17/o17_7.png)

![Terminal output](../img/objectives/o17/o17_8.png)
![Terminal output](../img/objectives/o17/o17_9.png)
![Terminal output](../img/objectives/o17/o17_10.png)

- ssh daemon location: `/opt/oqs-ssh/sbin`
```
 find / sshd|grep sshd
```

![Terminal output](../img/objectives/o17/o17_11.png)
![Terminal output](../img/objectives/o17/o17_12.png)
![Terminal output](../img/objectives/o17/o17_13.png)

#### The flag is: HHC{L3aping_0v3r_Quantum_Crypt0}