---
title: OAuth2-beskytta bruker-spesifikt REST-API for Kontakt- og Reservasjonsregisteret
description: OAuth2-beskytta bruker-spesifikt REST-API for Kontakt- og Reservasjonsregisteret
summary: "Kontaktinformasjon fra Kontakt- og Reservasjonsregisteret tilhørende  innlogget bruker er tilgjengelig på et eget Oauth2-beskyttet REST-API."
permalink: oidc_api_krr_user.html

layout: page
sidebar: oidc
---

## Introduksjon

Kontaktopplysninger fra Kontakt- og Reservasjonsregisteret er oftest utlevert globalt gjennom [Oppslagstjenesten](https://begrep.difi.no/Oppslagstjenesten/) (eller [Oauth2-varianten](oidc_api_krr.html) av denne).  

Men kunder kan også motta kontaktopplysninger kun tilhørende innlogget bruker, og dette kan i noen sammehenger være mer hensiktsmessig.

## Hvordan få tilgang ?

Denne tjenesten er en tilleggstjeneste i ID-porten. Se [https://samarbeid.difi.no/difis-felleslosninger/tilleggstjenester](https://samarbeid.difi.no/difis-felleslosninger/tilleggstjenester) for generelle vilkår for tilleggstjenester.


## Bruk av Oauth2

Tjenesten baserer seg på [autentiseringsnær autorisasjon](oidc_auth_oauth2.html).


Man må forespørre ett eller flere av følgende scopes:


| scope | beskrivelse |
|-|-|
| user/kontaktinformasjon.read | Returnerer epostadresse og mobilnummer + tidspunkt for sist oppdatering |
| user/varslingsstatus.read | Returnerer status for om kontaktinfomasjonen kan brukast for varsling iht. eForvaltningsforskrifta sin §32 |
| user/sikkerdigitalpost.read | Returnerer adresse for digital post til innbygger |
| user/sertifikat.read | Returnerer brukerens krypteringssertifikat ved sending av digital post |

og vil da motta et access_token som kan benyttes mot Kontakt- og Reservasjonsregisteret sitt endepunkt.

## API-endepunkt

|miljø|url|
|-|-|
|VER1|[https://oidc-ver1.difi.no/kontaktinfo-oauth2-server/rest/v1/person](https://oidc-ver1.difi.no/kontaktinfo-oauth2-server/rest/v1/person)|
|VER2|[https://oidc-ver2.difi.no/kontaktinfo-oauth2-server/rest/v1/person](https://oidc-ver2.difi.no/kontaktinfo-oauth2-server/rest/v1/person)|
|YT2|[https://oidc-yt2.difi.eon.no/kontaktinfo-oauth2-server/rest/v1/person](https://oidc-yt2.difi.eon.no/kontaktinfo-oauth2-server/rest/v1/person)|
|PROD|[https://oidc.difi.no/kontaktinfo-oauth2-server/rest/v1/person](https://oidc.difi.no/kontaktinfo-oauth2-server/rest/v1/person)|


{% include note.html content="Merk den lille forskjellen mellom 'person' (dette endepunktet) og 'personer' ([server-til-server endepunktet](oidc_api_krr.html)" %}


Følgende header-parametere må brukes på request:

| Parameter  | Verdi |
| --- | --- |
| Http-metode: | GET |
| Accept: | application/jose  (evt. application/json ) |
| Authorization: | Bearer \<utstedt access_token\> |

### Eksempel på respons:


Se https://begrep.difi.no/Oppslagstjenesten/Person for definisjon av kodeverket.


```
      {
         "personidentifikator": "23079421936",
         "reservasjon": "NEI",
         "status": "AKTIV",
		 "varslingsstatus" : "KAN_VARSLES",
         "kontaktinformasjon":
         {
            "epostadresse": "23079421936-test@minid.norge.no",
            "epostadresse_oppdatert": "2010-12-16T13:32:05.000+01:00"
         }
      }
```
