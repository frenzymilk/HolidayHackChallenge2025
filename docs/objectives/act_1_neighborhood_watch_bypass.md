---
icon: material/text-box-outline
---

# Neighborhood watch bypass

**Difficulty**: :fontawesome-solid-star::fontawesome-regular-star::fontawesome-regular-star::fontawesome-regular-star::fontawesome-regular-star:<br/>

## Objective

!!! question "Request"
    Assist Kyle at the old data center with a fire alarm that just won't chill.

??? quote "Kyle Parrish"
    This fire alarm keeps going nuts but there's no fire. I checked. <br/>
    I think someone has locked us out of the system. Can you see if you can get back in?

We have to find a way to bypass the current restriction of the fire alarm and elevate to the admin privileges and restore the system.

## Solution

This is a terminal challenge, so we will have to make good use of the command line interface.

![O2 Initial welcome](../img/objectives/o2/o2_1.png)

Upon doing a little reconnaissance, we notice that we have a symlink pointing to the executable we want to launch to restore the system.

![O2 Symlink](../img/objectives/o2/o2_2.png)

Let's take a look at the privileges we have in that terminal.

![O2 Privileges](../img/objectives/o2/o2_3.png)

Turns out we can execute a program to get the status of the fire alarm system.
The system also looks for executable inside the **/bin** folder of the current user.

![O2 Status](../img/objectives/o2/o2_4.png)

When reading this file, we notice that the system status executable makes use of the **free** command.

![O2 Read](../img/objectives/o2/o2_5.png)

When entering the name of an executable in the command line, the PATH environment variable is read from left to right to find the location of the program to run. 

What if we could create a fake **free** executable containing the instructions to run the executable **/etc/firealarm/restore_fire_alarm**?
We would also need to put that fake program at one of the first locations enumerated in the PATH, for instance, our **/bin** folder.
By doing this, upon checking for the fire alarm status, we would trigger the execution of the restoration program too.

![O2 Fake free](../img/objectives/o2/o2_6.png)

![O2 Success Screen](../img/objectives/o2/o2_7.png)

## The problem

An attacker can trick your program into running the wrong script by abusing the PATH environment variable, when absolute paths are not used.