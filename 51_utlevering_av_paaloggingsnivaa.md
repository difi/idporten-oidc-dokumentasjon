---
title: OAuth2 beskytta REST-api for innbyggers påloggingsnivå
pageid: pilot-oppslagstjensten-rest-api
layout: default
description: REST-api for innbyggers påloggingsnivå
isHome: false
---

## Introduksjon

Innbygger sitt høyeste brukte påloggingsnivå i ID-porten kan utleveres over et Oauth2-beskyttet REST-grensesnitt.  På denne måten kan virksomheten tilpasse sin dialog med innbygger, basert på kunnskapen om innbyggeren kan bruke en nivå4-pålogging eller ikke.


## Hvordan få tilgang ?

Denne tjenesten er en tilleggstjeneste i ID-porten. Se [https://samarbeid.difi.no/difis-felleslosninger/tilleggstjenester](https://samarbeid.difi.no/difis-felleslosninger/tilleggstjenester) for generelle vilkår for tilleggstjenester. 


## Bruk av Oauth2

REST-grensesnittet er sikret vha. [server-til-server Oauth](https://difi.github.io/idporten-oidc-dokumentasjon//4_server-to-server-oauth2.html).

Scopet som gir tilgang til grensesnittet er *global/idporten.authlevel.read*.

Kunden må i bestillingen oppi organisasjonsnummeret som brukes i virksomhetssertifikatet som generer JWT-førespørselen.  Sertifikatet må være virksomhetssertifikat fra Buypass eller Commfides, med key usage: Digital Signature.

For test-miljøer brukes virksomshetssertifiakter fra Buypass / Commfides sine TEST verdikjeder. 

## REST-grensesnittet

REST-grensesnittet er basert på oppslag på enkelt-personer, og er dokumentert vha. Swagger på følgende URL:
[https://kontaktinfo-ws-ver2.difi.no/authlevel/swagger-ui.html#/](https://kontaktinfo-ws-ver2.difi.no/authlevel/swagger-ui.html#/)

Mulige feilkoder er: 

| --- | --- |
|400 | Bad request – syntaktisk ugyldig request body.|
|401 | Ugyldig access token, f.eks. utløpt|
|403 |  Gyldig access token men bundet til feil scope ( sær feilsituasjon ) |
|404 | Forespurt bruker er ikkje registrert som id-porten bruker|


## Eksempel-kode

Eksempel på bruk av tjenesten kan finnes her:
[https://github.com/difi/idporten-authlevel-example](https://github.com/difi/idporten-authlevel-example)

## Informasjon om miljøer

Tjenesten er tilgjengelig i VER2, YT2 og produksjonsmiljøet til ID-proten


