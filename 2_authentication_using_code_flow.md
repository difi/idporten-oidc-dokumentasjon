---
title: Autentisering med bruk av autorisasjonskode-flyten
pageid: authentication-using-code-flow
layout: default
description: Bruk av Idporten sin OpenID Connect provider til autentisering med autorisasjonskode-flyten
isHome: false
resource: true
category: vStaging
---

## Introduksjon

ID-porten sin OpenID Connect provider tilbyr funksjonalitet for autentisering av sluttbrukere basert på autorisasjonskode-flyten, slik den er spesifisert i OpenID Connect Core 1.0 spesifikasjonen

## Overordna beskrivelse av scenariet

OpenID Connect tilbyr autentisering av brukere til sluttbrukertjenester. Autentiseringen blir utført av ein OpenID Connect provider som utseder ID Token til den aktuelle tjenesten.

Følgende aktører inngår:

* *Sluttbrukeren* \- Ønsker å logge inn til en gitt tjeneste (relying_party)
* *Relying party* (også kjent som klienten) - Tjenesten som brukeren skal logge inn til
* *OpenID Connect provider* \- Autentiseringstjeneste der autentiseringen blir utført og som usteder *ID Token* til aktuelle tjenesten

![](/idporten-oidc-dokumentasjon/assets/images/openid_scenario.png "OpenID Connect scenario")


## Beskrivelse av autorisasjonskode-flyten

![](/idporten-oidc-dokumentasjon/assets/images/openid_auth_code_flow.png "Sekvensdiagram som viser server-til-server Oauth2-flyten")

* Flyten starter med at en sluttbruker prøver å aksessere en gitt tjeneste (klient)
* Tjenesten krever innlogging og en redirect url til OpenID Connect provideren blir generert og returnert til sluttbrukeren. Denne redirecten representerer en **autentiseringsforespørsel**, og har parametere som identifiserer den aktuelle klienten for provideren.
* Sluttbrukeren kommer til **autorisasjonsendepunktet** hos provideren hvor forespørselen blir validert (f.eks. gyldig klient og gyldig redirect_uri tilbake til klienten).
* Brukeren gjennomfører **innlogging i provideren**
* Provideren redirect'er brukeren tilbake til klienten. redirect url'en har satt en **autorisasjonskode**.
* Klienten bruker den mottatte autorisasjonskoden til å gjøre et direkteoppslag mot providerens **token-endepunkt**. Klienten må autentisere seg mot token-endepunktet (enten med client_secret eller en signert forespørsel)
* Dersom klienten er autentisert valideres den mottatte autorisasjonskoden og et **ID token** blir returnert til klienttjenesten.
* Brukeren er nå autentisert for klienttjenesten og ønsket handling kan utføres

Merk: OpenID Connect bygger på OAuth2, og denne flyten er derfinert i OAuth2-spesifikasjonen. Siden *autentisering* ikke er et bergrep i OAuth2 vil en ofte se at begrepet *autorisasjon* blir brukt selv om man egentlig snakker om *autentisering*

## Struktur på Id token

Det returnerte ID tokenet er en signert JWT struktur i henhold til OpenID Connect spesifikasjonen:

```
{
  "kid" : "igb5CyFMAmFeei4MnXBo6mc93-7mEp7ogrIqWhMTcKc",
  "alg" : "RS256"
}
```

```
{
  "sub" : "9lTykiZN9MzrLvLiAdKgPiiA7Q6OEvW1_Q3801Onv3g=",
  "aud" : "test_rp_eid_exttest_difi",
  "acr" : "Level3",
  "auth_time" : 1478698504,
  "amr" : "Minid-PIN",
  "iss" : "https://eid-exttest.difi.no/idporten-oidc-provider/",
  "pid" : "23079422487",
  "exp" : 1478698626,
  "locale" : "nb",
  "iat" : 1478698506,
  "jti" : "R1ao4koEKFr7R5d5W-Ys8e13sbxF6ms8o3QhCChI6fk="
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
 

## Autorisasjons-endepunkt

Klienten redirecter sluttbrukeren til autorisasjonsendepunktet. Etter at brukeren har logget inn vil det sendes en redirect url tilbake til klienten. Denne url'en vil inneholde et autorisasjonskode-parameter som kan brukes til oppslag på tokens.

```
URL: http://eid-exttest.difi.no/idporten-oidc-provider/authorization
```

&nbsp; 

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
| state | Verdi som settes av klient og returneres i callback-responsen etter fullført autentisering. Kan benyttes til å implementere CSRF-beskyttelse |
| nonce | Verdi som settes av klient og returneres som en del av ID token. Kan brukes til å binde en klient-sesjon til et gitt ID-token, og hindre replay attacks  |
| acr\_values | Ønsket sikkerhetsnivå, kan være *Level3* eller *Level4* |
| ui\_locales | Ønsket språk brukt i Id-porten. støtter *nb*, *nn*, *en* eller *se* |
| prompt | Brukes til å styre providerens interaksjon med sluttbrukeren. Foreløpig er dette parameteret lite relevant da piloten ikke ivaretar noen sentral brukersesjon |


## Token-endepunkt

Token-endepunktet brukes for utstedelse av tokens. 

```
URL: http://eid-exttest.difi.no/idporten-oidc-provider/token
```

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
Content-type: application/x-www-form-urlencoded
Authorization: basic afjkpafjpfm2rpjnpfwjjfp2mfkwp
 
grant_type=authorization_code&code=myrecievedcode&client_id=myclientid
```

### Eksempel på respons:

```
{
    "access_token": "fK0dhs5vQsuAUguLL2wxbXEQSE91XbOAL3foY5VR0Uk=",
    "expires_in": 599,
    "scope": "global/kontaktinformasjon.read"
}
```

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