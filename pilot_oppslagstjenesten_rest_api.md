---
title: OAuth2 beskytta REST-api for oppslagstjenesten
pageid: pilot-oppslagstjensten-rest-api
layout: default
description: Pilot av REST-api for Oppslagstjensten.
isHome: false
---

## Introduksjon

Kontaktregistert sin oppslagstjeneste tilbyr i dag eit SOAP-basert grensesnitt beskytta med WS-Security. Målet med denne piloten er å tilby et REST-basert grensesnitt som er enklere å integrere mot.

## Flyt


## Krav til JWT for token-forespørsel
 
Klienten må generere og signere ein jwt med følgende elementer for å forespørre tokens fra autorisasjonsserveren:


**Header:**

| Parameter  | Verdi |
| --- | --- |
| x5c | Inneholde klientens virksomhetssertifikat som er brukt for signering av JWT'en |
| alg | RS256 - Vi støtter kun RSA-SHA256 som signeringsalgoritme |

**Body:**

| Parameter  | Verdi |
| --- | --- |
|aud| Audience - identifikator for ID-portens OIDC Provider - skal være: https://eid-vag-opensso.difi.local/idporten-oidc-provider/|
|iss| issuer - client ID som er registert hos ID-porten OIDC-provider|
|scope| Scope som klient forespør tilgang til, kan sende inn liste av scope separert med whitespace|
|iat| issued at - tidsstempel for når jwt'en ble generert - **MERK:** Tidsstempelet tar utgangspunkt i UTC-tid|
|exp| expiration time - tidsstempel for når jwt'en utløper - **MERK:** Tidsstempelet tar utgangspunkt i UTC-tid **MERK:** ID-porten godtar kun maks levetid på jwt'en til 120 sekunder (exp - iat <= 120 )|
|jti| Optional - JWT ID - unik id på jwt'en som settes av klienten. **MERK:** JWT'er kan ikke gjenbrukes. ID-porten håndterer dette ved å sammenligne en hash-verdi av jwt'en mot tidligere brukte jwt'er. Dette impliserer at dersom klienten ønsker å sende mer enn en token-request i sekundet må jti elementet benytttes.|

 

## Tilgjenglige scopes




## Eksempel på forespørsel

```
POST /rest/v1/personer
Content-type: application/json
Authorization: Bearer SWDQ_pVct3HIzsIaC3zHDuMmIqffr4ORr508N3p0Mtg=
 
{
 "personidentifikatorer" : [ "23079422568" ]
}
```

## Eksempel på respons

```
{
  "personer":
    [
      {
         "personidentifikator": "23079421936",
         "reservasjon": "NEI",
         "status": "AKTIV",
         "kontaktinformasjon":
         {
            "epostadresse": "23079421936-test@minid.norge.no",
            "epostadresse_oppdatert": "2010-12-16T13:32:05.000+01:00"
         }
      }
   ]
}
```

