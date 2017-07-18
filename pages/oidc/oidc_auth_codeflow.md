---
title: Autentisering med bruk av autorisasjonskode-flyten
description: Bruk av Idporten sin OpenID Connect provider til autentisering med autorisasjonskode-flyten
summary: "Autorisasjonskode-flyten er den vanlige flyten som blir brukt i OpenID Connect, og er anbefalt flyt for dei fleste tjenester."
permalink: oidc_auth_codeflow.html

layout: page
sidebar: oidc
isHome: true
---

ID-porten sin OpenID Connect provider tilbyr funksjonalitet for autentisering av sluttbrukere basert på autorisasjonskode-flyten, slik den er spesifisert i OpenID Connect Core 1.0 spesifikasjonen

## Overordna beskrivelse av scenariet

OpenID Connect tilbyr autentisering av brukere til sluttbrukertjenester. Autentiseringen blir utført av en OpenID Connect provider som utsteder ID Token til den aktuelle tjenesten.

Følgende aktører inngår:

 Aktør | Beskrivelse | Begrep OIDC | Begrep Oauth2 | Begrep SAML2
 -|-|-|-|-|
 Innbygger | Ønsker å logge inn til en offentlig tjeneste | End User | User | End User
 Nett-tjeneste | Sluttbruker-tjeneste tilbudt av en offentlig etat | Relying Party (RP) | Client | Service Provider (SP) |
 ID-porten | ID-porten sin autentiseringstjeneste som usteder *ID Token* til aktuelle tjenesten| OpenID Provider (OP) | Authorization server (AS) | Identity Provider (IDP)

```mermaid {class:"mermaid"}
graph LR
  end_user(Sluttbruker)
  OP(OpenID Connect provider)
  RP(Nett-tjeneste)

  end_user -. autentiserer seg hos .-> OP
  OP -. utsteder id_token .-> RP
  end_user -. logger inn i  .-> RP
```

## Beskrivelse av autorisasjonskode-flyten

<div class="mermaid">
sequenceDiagram

  Sluttbruker ->> Relying_Party: Klikker login-knapp
  Relying_Party ->> Sluttbruker: Redirect med authorizationsforespørsel
  Sluttbruker ->> OpenID Provider: følg redirect...
  note over Sluttbruker,OpenID Provider: Sluttbruker autentiserer seg (og evt. samtykker til førespurte scopes)
  OpenID Provider ->> Sluttbruker: Redirect med autorisasjonscode
  Sluttbruker ->> Relying_Party: følg redirect...
  Relying_Party ->> OpenID Provider: forespørre token (/token)
  OpenID Provider ->> Relying_Party: id_token (evt. flere tokens)
  note over Sluttbruker,Relying_Party: Innlogget i tjenesten

</div>


asdf asdf asdf sadf sadf saN26WBlRyvBRH1j8A9smQv5XxJoXssfxMr-t1ZB5wDM37MOkwMF4zTNPVmyeQ0qM0PAudG7ZpT0gWPksQIWOoSk4A--MoOHPBy41xXWSpOvUh3jBqrnWEcZpqS785Ufofc6cDfXk_wM_-EMAlS-UExMq-hH60nPwXmR0cBNW3GV2xm_frYyqBYnxXoELmzREijpeSyiELTqn2k4nwCjeiGDXXs_Nw12D2KpWLDctqqsUtTTRUhsnCPSoDng


* Flyten starter med at en sluttbruker prøver å aksessere en gitt tjeneste (klient)
* Tjenesten krever innlogging og en redirect url til OpenID Connect provideren blir generert og returnert til sluttbrukeren. Denne redirecten representerer en **autentiseringsforespørsel**, og har parametere som identifiserer den aktuelle klienten for provideren.
* Sluttbrukeren kommer til **autorisasjonsendepunktet** hos provideren hvor forespørselen blir validert (f.eks. gyldig klient og gyldig redirect_uri tilbake til klienten).
* Brukeren gjennomfører **innlogging i provideren**
* Provideren redirect'er brukeren tilbake til klienten. redirect url'en har satt en **autorisasjonskode**.
* Klienten bruker den mottatte autorisasjonskoden til å gjøre et direkteoppslag mot providerens **token-endepunkt**. Klienten må autentisere seg mot token-endepunktet (enten med client_secret eller en signert forespørsel)
* Dersom klienten er autentisert valideres den mottatte autorisasjonskoden og et **ID token** blir returnert til klienttjenesten.
* Brukeren er nå autentisert for klienttjenesten og ønsket handling kan utføres

Merk: OpenID Connect bygger på OAuth2, og denne flyten er derfinert i OAuth2-spesifikasjonen. Siden *autentisering* ikke er et bergrep i OAuth2 vil en ofte se at begrepet *autorisasjon* blir brukt selv om man egentlig snakker om *autentisering*




## Autentiseringsforespørsel til autorisasjons-endepunktet

Klienten sender en autentiseringsforespørsel ved å redirecter sluttbrukeren til autorisasjonsendepunktet.



Følgende header-parametere må brukes på request:

| Parameter  | Verdi |
| --- | --- |
|Http-metode|GET|

&nbsp;

Følgende attributter må settes på request:

| Parameter  | Verdi |
| --- | --- |
| response_type | Her støtter vi kun 'code'|
| client\_id | Klientens tildelte id |
| redirect\_uri | URI som sluttbruker skal redirectes tilbake til etter fullført authentisering. Kun forhåndsregistrerte url'er kan brukes |
| scope | Scope som forespørres. Kan være en liste separert med whitespace. For autentiseringer må _openid_ brukes |
| state | Verdi som settes av klient og returneres i callback-responsen etter fullført autentisering. Bør benyttes til å implementere CSRF-beskyttelse |
| nonce | Verdi som settes av klient og returneres som en del av ID token. Bør brukes til å binde en klient-sesjon til et gitt ID-token, og hindre replay attacks  |
| acr\_values | Ønsket sikkerhetsnivå, kan være *Level3* eller *Level4* |
| ui\_locales | Ønsket språk brukt i Id-porten. støtter *nb*, *nn*, *en* eller *se* |
| prompt | Brukes til å styre providerens interaksjon med sluttbrukeren. Foreløpig er dette parameteret lite relevant da piloten ikke ivaretar noen sentral brukersesjon |


Etter at brukeren har logget inn vil det sendes en redirect url tilbake til klienten. Denne url'en vil inneholde et autorisasjonskode-parameter som kan brukes til oppslag for å hente tokens.


### Eksempel på forespørsel

```

GET /authorize

  scope=openid&
  acr_values=Level3&
  client_id=test_rp_yt2&
  redirect_uri=https://eid-exttest.difi.no/idporten-oidc-client/authorize/response&
  response_type=code&
  state=min_fine_state_verdi&
  nonce=min_fine_nonce_verdi&
  ui_locales=nb

```

### Eksempel på respons:

```
{
  "code" : "1JzjKYcPh4MIPP9YWxRfL-IivWblfKdiRLJkZtJFMT0=",
  "state" : "min_fine_state_verdi"
}
```





## Utstedelse av token fra token-endepunktet

Token-endepunktet brukes for utstedelse av tokens.

&nbsp;

Følgende header-parametere må brukes på request:

| Parameter  | Verdi |
| --- | --- |
| Http-metode | POST |
| Content-type | application/x-www-form-urlencoded |

&nbsp;

Følgende attributter må sendes inn i requesten:

| Attributt  | Verdi |
| --- | --- |
| grant_type | authorization\_code \| refresh\_token|
| code | authorization\_gode dersom dette benyttes som grant |
| client_id | Klientens ID |
| client_secret |

### Klientautentisering mot endepunktet

I pilotfasen støttes to typer autentisering:

* client_secret_basic / client_secret_post - Klientautentisering basert på client_secret
* private_key_jwt - Klientautentisering basert på JWT'er signert med virksomhetssertifikater

### Eksempel på forespørsel

```
POST /token
Content-Type: application/x-www-form-urlencoded
Authorization: Basic dGVzdF9ycF95dDI6cGFzc3dvcmQ=

grant_type=authorization_code&redirect_uri=https%3A%2F%2Feid-exttest.difi.no%2Fidporten-oidc-client%2Fauthorize%2Fresponse&code=1JzjKYcPh4MIPP9YWxRfL-IivWblfKdiRLJkZtJFMT0%3D
```

### Eksempel på respons:

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


## Struktur på Id token

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
| acr | "Authentication Context Class Reference" - Angir sikkerhetsnivå for utført autentisering. I ID-porten sammenheng er mulige verdier "Level3" (dvs. MinID) eller "Level4" (De andre eID'ene) |
| auth_time | Tidspunktet for når autentiseringen ble utført. Dvs tidspunktet når brukeren logget inn i ID-porten |
| amr | "Authentication Methods References" - Autentiseringsmetode. For ID-porten er mulige verdier her *Minid-PIN*, *Minid-OTC*, *Commfides*, *Buypass*, *BankID* eller *BankID-mobil* |
| iss | Identifikator for provideren som har utstedt token'et. For ID-porten sitt ext-test miljø er dette *https://eid-exttest.difi.no/idporten-oidc-provider/* |
| pid | Personidentifikator - Id-porten spesifikt claim som gir brukerens fødselsnummer |
| exp | Expire - Utløpstidspunktet for Id tokenet. Klienten skal ikke akseptere token'et etter dette tidspunktet |
| locale | Språk valgt av sluttbrukeren under innlogging i Id-porten |
| iat | Tidspunkt for utstedelse av tokenet |
| jti | jwt id - unik identifikator for det aktuelle Id tokenet |


## Validering av Id token

Korrekt validering av Id token på klientsiden er kritisk for sikkerheten i løsningen. Tjenesteleverandører som tar i bruk tjenesten må utføre validering i henhold til kapittel *3.1.3.7 - ID Token Validation* i OpenID Connect Core 1.0 spesifikasjonen.


## Userinfo-endepunkt

Ved å forespørre scopet *profile* vil klienttjenesten sammen med id tokenet også få utstedt et access_token (og evnt. refresh_token)
som kan benyttes mot providerens userinfo-endepunkt. Dette endepunktet kan benyttes for å hente ytterligere data om brukeren enn det som blir eksponert via ID tokenet.
Da ID-porten generelt har lite data om sluttbrukeren har dette endepunktet begrenset verdi for denne tjenesten. Personnummer og valgt språk under innlogging er de
dataene som vil bli eksponert her.


```
URL: http://eid-exttest.difi.no/idporten-oidc-provider/userinfo
```

&nbsp;

Følgende header-parametere må brukes på request:

| Parameter  | Verdi |
| --- | --- |
| Http-metode: | GET |
| Authorization: | Bearer \<utstedt access_token\> |

## Eksempel på respons:

```
{
  "sub" : "NR8vTTPrM3T7rWf8dXxeWLZpxEMsug4E7pxqJuh9wIM=",
  "pid" : "23079421936",
  "locale" : "nb"
}
```
