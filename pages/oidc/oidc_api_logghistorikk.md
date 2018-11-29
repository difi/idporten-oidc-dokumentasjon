---
title: OAuth2 beskytta REST-API for logg-historikk i ID-porten og Kontaktregisteret
description: API for logg-historikk i ID-porten og Kontaktregisteret
summary: "Innbygger sin logg-historikk fra ID-porten og Kontaktregisteret kan konsumeres via API og synliggjøres for innbygger i andre tjenester.  Dette gir innbygger innsyn i bruk av egne data fra Difis løsninger"
permalink: oidc_api_logghistorikk.html

layout: page
sidebar: oidc
isHome: true
---

## Introduksjon

Innbygger sin logg-historikk fra ID-porten og Kontaktregisteret kan konsumeres via API og synliggjøres for innbygger i andre tjenester.

Kunden mottar opplysningene for at innbyggeren kan lettere deteketere misbruk av eID.

## Hvordan få tilgang ?

Denne tjenesten er en tilleggstjeneste i ID-porten. Se [https://samarbeid.difi.no/difis-felleslosninger/tilleggstjenester](https://samarbeid.difi.no/difis-felleslosninger/tilleggstjenester) for generelle vilkår for tilleggstjenester.


## Bruk av Oauth2

REST-grensesnittet er sikret vha. [autentiseringsnær autorisasjon](https://difi.github.io/idporten-oidc-dokumentasjon/oidc_auth_oauth2.html).

Scopet som gir tilgang til grensesnittet er `idporten:user.log.read`.


## REST-grensesnittet

REST-grensesnittet er basert på at innkommende access token tilhører innlogget bruker,  kunden skal derfor ikke oppgi fødseslnummer selv.

Grensesnittet er dokumenter vha. Swagger [her-TBD]().

## Eksempel
