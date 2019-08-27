---
title: Autentisering med bruk av autorisasjonskode-flyten
description: Bruk av Idporten sin OpenID Connect provider til autentisering med autorisasjonskode-flyten
summary: "Autorisasjonskode-flyten er den vanlige flyten som blir brukt i OpenID Connect, og er anbefalt flyt for dei fleste tjenester."
sidebar: oidc
permalink: oidc_auth_codeflow.html 
redirect_to: https://difi.github.io/felleslosninger/oidc_auth_codeflow.html
---
Flyttet til https://difi.github.io/felleslosninger/oidc_auth_codeflow.html#### Eksempel på forespørsel:

```
POST /token
Content-Type: application/x-www-form-urlencoded

grant_type=authorization_code&
   code=n0esc3NRze7LTCu7iYzS6a5acc3f0ogp4&
   client_assertion_type=urn%3Aietf%3Aparams%3Aoauth%3Aclient-assertion-type%3Ajwt-bearer&
   client_assertion=< jwt >
```


### Eksempel på respons fra token-endepunktet:

```
{
  "access_token" : "IxC0B76vlWl3fiQhAwZUmD0hr_PPwC9hSIXRdoUslPU=",
  "id_token" : "eyJraWQiOiJtcVQ1QTNMT1NJSGJwS3JzY2IzRUhHcnItV0lGUmZMZGFxWl81SjlHUjlzIiwiYWxnIjoiUlMyNTYifQ.eyJzdWIiOiItdi1sY2FlNXJHRy1qbHZ6dXY5WTlIN1I4Tm1BZU0yLWtoMHFXYi12UElFPSIsImF1ZCI6InRlc3RfcnBfeXQyIiwiYWNyIjoiTGV2ZWw0IiwiYXV0aF90aW1lIjoxNDk3NjA1MjE4LCJhbXIiOiJCYW5rSUQiLCJpc3MiOiJodHRwczpcL1wvb2lkYy15dDIuZGlmaS5lb24ubm9cL2lkcG9ydGVuLW9pZGMtcHJvdmlkZXJcLyIsInBpZCI6IjIzMDc5NDEwOTE4IiwiZXhwIjoxNDk3NjA1MzgyLCJsb2NhbGUiOiJuYiIsImlhdCI6MTQ5NzYwNTI2Miwibm9uY2UiOiJtaW5fZmluZV9ub25jZV92ZXJkaSIsImp0aSI6IkhnYjN6d085ZzBiam1TYkNDdFFDeE1vd3NaRXUwMGxDSjJFeGc0Wmh2M2c9In0.Pl9APC3_GGJBLYR3AqZRC8-fjOWdIW3eQAn2zbqstGEyv8AJ6yPLiH0EA4e1RgHxK-dPwtydJF0fV-1aiPjDGYM8d-saN26WBlRyvBRH1j8A9smQv5XxJoXssfxMr-t1ZB5wDM37MOkwMF4zTNPVmyeQ0qM0PAudG7ZpT0gWPksQIWOoSk4A--MoOHPBy41xXWSpOvUh3jBqrnWEcZpqS785Ufofc6cDfXk_wM_-EMAlS-UExMq-hH60nPwXmR0cBNW3GV2xm_frYyqBYnxXoELmzREijpeSyiELTqn2k4nwCjeiGDXXs_Nw12D2KpWLDctqqsUtTTRUhsnCPSoDng",
  "token_type" : "Bearer",
  "expires_in" : 599,
  "refresh_token" : "yBtapz3ThC3uVWufWhxsLtbEidPnEsL7atvfHSBANDs=",
  "scope" : "openid"
}
```






## Struktur på Id token {#idtoken}

Det returnerte ID tokenet er en signert JWT struktur i henhold til OpenID Connect spesifikasjonen:

```
{
  "kid" : "mqT5A3LOSIHbpKrscb3EHGrr-WIFRfLdaqZ_5J9GR9s",
  "alg" : "RS256"
}
```

```
{
  "sub" : "-v-lcae5rGG-jlvzuv9Y9H7R8NmAeM2-kh0qWb-vPIE=",
  "aud" : "test_rp_yt2",
  "acr" : "Level4",
  "auth_time" : 1497605218,
  "amr" : "BankID",
  "iss" : "https://oidc-yt2.difi.eon.no/idporten-oidc-provider/",
  "pid" : "23079410918",
  "exp" : 1497605382,
  "locale" : "nb",
  "iat" : 1497605262,
  "nonce" : "min_fine_nonce_verdi",
  "jti" : "Hgb3zwO9g0bjmSbCCtQCxMowsZEu00lCJ2Exg4Zhv3g="
}
```

```
OuFJaVWQvLY9... <signaturverdi> ...isvpDMfHM3mkI
```


### ID tokenets header:

| claim | verdi |
| --- | --- |
| kid | "Key identifier" - unik identifikator for signeringsnøkkel brukt av provideren. Nøkkel og sertifikat hentes fra providerens JWK-endepunkt |
| alg | "algorithm" - signeringsalgoritme, Id-porten støtter kun RS256 (RSA-SHA256)


### ID tokenets body:

| claim | verdi |
| --- | --- |
| sub | "subject identifier" - unik identifikator for den aktuelle brukeren. Verdien er her *pairwise* - dvs en klient får alltid samme verdi for samme bruker. Men ulike klienter vil få ulik verdi for samme bruker |
| aud | "audience" - client_id til klienten som er mottaker av dette tokenet |
| acr | "Authentication Context Class Reference" - Angir sikkerhetsnivå for utført autentisering. I ID-porten sammenheng er mulige verdier "Level3" (dvs. MinID) eller "Level4" (De andre eID'ene), denne skal brukes for å sikre at brukeren er autentisert på tilstrekkelig nivå |
| auth_time | Tidspunktet for når autentiseringen ble utført. Dvs tidspunktet når brukeren logget inn i ID-porten |
| amr | "Authentication Methods References" - Autentiseringsmetode. For ID-porten er mulige verdier her *Minid-PIN*, *Minid-OTC*, *Commfides*, *Buypass*, *BankID* eller *BankID Mobil*, dette kan endre seg med avtaler og tilgjengelige e-IDer |
| iss | Identifikator for provideren som har utstedt token'et. For ID-porten sitt ext-test miljø er dette *https://eid-exttest.difi.no/idporten-oidc-provider/* |
| pid | Personidentifikator - Id-porten spesifikt claim som gir brukerens norske personidentifikator (fødselsnummer eller d-nummer) |
| exp | Expire - Utløpstidspunktet for Id tokenet. Klienten skal ikke akseptere token'et etter dette tidspunktet |
| locale | Språk valgt av sluttbrukeren under innlogging i Id-porten |
| iat | Tidspunkt for utstedelse av tokenet |
| jti | jwt id - unik identifikator for det aktuelle Id tokenet |
| sid | sesjonsid - en unik identifikator for brukerens sesjon med ID-porten OIDC Provider |


## Validering av Id token

Korrekt validering av Id token på klientsiden er kritisk for sikkerheten i løsningen. Tjenesteleverandører som tar i bruk tjenesten må utføre validering i henhold til kapittel *3.1.3.7 - ID Token Validation* i OpenID Connect Core 1.0 spesifikasjonen.


## Userinfo-endepunkt

Ved å forespørre scopet *profile* vil klienttjenesten sammen med id tokenet også få utstedt et access_token (og evnt. refresh_token)
som kan benyttes mot providerens userinfo-endepunkt. Dette endepunktet kan benyttes for å hente ytterligere data om brukeren enn det som blir eksponert via ID tokenet.
Da ID-porten generelt har lite data om sluttbrukeren har dette endepunktet begrenset verdi for denne tjenesten. Personnummer og valgt språk under innlogging er de
dataene som vil bli eksponert her.


```
URL: https://<<miljø>>/idporten-oidc-provider/userinfo
```

&nbsp;

Følgende header-parametere må brukes på request:

| Parameter  | Verdi |
| --- | --- |
| Http-metode: | GET |
| Authorization: | Bearer \<utstedt access_token\> |

### Eksempel på respons:

```
{
  "sub" : "NR8vTTPrM3T7rWf8dXxeWLZpxEMsug4E7pxqJuh9wIM=",
  "pid" : "23079421936",
  "locale" : "nb"
}
```

## Kontaktopplysninger fra Kontakt- og Reservasjonsregisteret

Kontakt-opplysninger knyttet til innlogget bruker, er [tilgjengelig på et eget endepunkt](oidc_api_krr_user.html)
