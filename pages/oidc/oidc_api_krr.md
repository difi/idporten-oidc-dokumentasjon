---
title: OAuth2 beskytta REST-API for Kontakt- og Reservasjonsregisteret
description: OAuth2 beskytta REST-API for Kontakt- og Reservasjonsregisteret
summary: "Funksjonaliteten i Kontakt- og Reservasjonsregisteret sin webservice *Oppslagstjenesten* er nå også tilgjengelig over et Oauth2-beskyttet REST-API."
permalink: oidc_api_krr.html

layout: page
sidebar: oidc
---

## Introduksjon

Kontaktregisteret sin oppslagstjeneste tilbyr i dag eit SOAP-basert grensesnitt beskytta med WS-Security. Som del av lanseringen av OpenID Connect i ID-porten  tilbyr Difi nå også et alternativt REST-basert grensesnitt som gjør det enklere å implementere nye integrasjoner mot registeret.

## Bruk av Oauth2

Tilgangskontrollen til api'et benytter seg av flyten som er beskrevet i [Server til server autorisasjon med Oauth2](oidc_auth_server-to-server-oauth2.html)

Merk at funksjoanlitet for lokal kopi (endringsmeldinger) ikke er støttet over Oauth2-grensenittet.

### Krav til JWT for token-forespørsel

Klienten må generere og signere ein jwt med følgende elementer for å forespørre tokens fra autorisasjonsserveren:


**JWT-Header:**

| Parameter  | Verdi |
| --- | --- |
| x5c | Inneholde klientens virksomhetssertifikat som er brukt for signering av JWT'en |
| alg | RS256 - Vi støtter kun RSA-SHA256 som signeringsalgoritme |


**JWT-Body:**

| Parameter  | Verdi |
| --- | --- |
|aud| Audience - identifikator for ID-portens OIDC Provider - skal være: https://eid-vag-opensso.difi.local/idporten-oidc-provider/|
|iss| issuer - klient-ID som er registert hos ID-porten OIDC-provider|
|scope| Scope som klient forespør tilgang til, kan sende inn liste av scope separert med whitespace|
|iat| issued at - tidsstempel for når jwt'en ble generert - **MERK:** Tidsstempelet tar utgangspunkt i UTC-tid|
|exp| expiration time - tidsstempel for når jwt'en utløper - **MERK:** Tidsstempelet tar utgangspunkt i UTC-tid **MERK:** ID-porten godtar kun maks levetid på jwt'en til 120 sekunder (exp - iat <= 120 )|
|jti| Optional - JWT ID - unik id på jwt'en som settes av klienten. **MERK:** JWT'er kan ikke gjenbrukes. ID-porten håndterer dette ved å sammenligne en hash-verdi av jwt'en mot tidligere brukte jwt'er. Dette impliserer at dersom klienten ønsker å sende mer enn en token-request i sekundet må jti elementet benytttes.|


### Tilgjenglige scopes

Det er 1-1 mapping mellom OAuth2 scopes og informasjonsbehov-elementet brukt i SOAP-API’et. Se [begrepskatalogen for Kontaktregisteret](https://begrep.difi.no/Oppslagstjenesten/).

| scope | beskrivelse |
|-|-|
| global/kontaktinformasjon.read | Returnerer epostadresse og mobilnummer + tidspunkt for sist oppdatering |
| global/varslingsstatus.read | Returnerer status for om kontaktinfomasjonen kan brukast for varsling iht. eForvaltningsforskrifta sin §32 |
| global/sikkerdigitalpost.read | Returnerer adresse for digital post til innbygger |
| global/sertifikat.read | Returnerer brukerens krypteringssertifikat ved sending av digital post |

Det vil alltid returneres reservasjonsstatus for brukeren.

## Endepunkt

Oppslagstjenesten sin REST-tjeneste tilbyr følgende endepunkt for søk på 1...1000 personer:


|miljø|url|
|-|-|
|VER1|[https://oidc-ver1.difi.no/kontaktinfo-oauth2-server/rest/v1/personer](https://oidc-ver1.difi.no/kontaktinfo-oauth2-server/rest/v1/personer)|
|VER2|[https://oidc-ver2.difi.no/kontaktinfo-oauth2-server/rest/v1/personer](https://oidc-ver2.difi.no/kontaktinfo-oauth2-server/rest/v1/personer)|
|YT2|[https://oidc-yt2.difi.eon.no/kontaktinfo-oauth2-server/rest/v1/personer](https://oidc-yt2.difi.eon.no/kontaktinfo-oauth2-server/rest/v1/personer)|
|PROD|[https://oidc.difi.no/kontaktinfo-oauth2-server/rest/v1/personer](https://oidc.difi.no/kontaktinfo-oauth2-server/rest/v1/personer)|

**Merk:** Man vil oppnå vesentlig bedre ytelse (målt i personer/sekund) ved å slå opp 1000 personer i gangen kontra 1000 enkelt-oppslag.

Følgende header-parametere må brukes på request:

| Parameter  | Verdi |
| --- | --- |
| Http-metode | POST |
| Authorization | Bearer ``<access_token utstedt fra idporten-oidc-provider på bakgrunn av generert JWT-bearer grant>`` |

Body i requesten er en JSON-struktur med et element `personidentifikatorer` som skal inneholde en liste med inntil 1000 personidentifikatorer.

### Eksempel på forespørsel

```
POST /rest/v1/personer
Content-type: application/json
Authorization: Bearer SWDQ_pVct3HIzsIaC3zHDuMmIqffr4ORr508N3p0Mtg=

{
 "personidentifikatorer" : [ "23079422568" ]
}
```

### Eksempel på respons

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
