---
title: OpenID Connect i ID-porten
description: Pilot av OpenID Connect funksjonalitet i Id-porten
summary: "ID-porten lanserer i 2017 en ny protokoll for integrasjon: OpenID Connect"
permalink: oidc_introduksjon.html

layout: page
sidebar: oidc
isHome: true
---

## Introduksjon og bakgrunn for pilot

ID-porten støtter idag autentisering av innbyggere basert på SAML2-protokollen. Dette er et teknologivalg som i praksis ble tatt midt på 2000-tallet. De siste 10 årene har det skjedd en betydlig teknologiutvikling, som gjør at det er naturlig å nå se på alternative løsninger.

En av de viktigste driverne for å vurder alternative protokoller er fremveksten av API-drevne tjenester samt nye mobile platformer. SAML2 er designet for tradisjonelle webtjenester, og er på mange måter lite egnet for bruk på disse nye tjeneste-typene.

Selv etter 10 år med SAML2, opplever Difi fremdels at enkelte virksomheter bruker betydlige ressurser på å sette opp en ordinær SAML2-integrasjon også for tradisjonelle webtjenester. Det er derfor naturlig å evaluere om det finst andre og mer moderne protokoller som har både høy utbredelse og høy sikkerhet.

Dette prosjektet omfatter også en pilot av et enklere REST-api for Kontakt- og reservasjonsregisteret sin Oppslagstjeneste. Denne tjenesten har i dag et API basert på SOAP og WS-Security. Selv om dette ikke er direkte relatert til bruk av OpenID Connect så vil tilgangskontrollen til dette REST-api'et være basert på OAuth2, og det er derfor naturlig å realisere en felles OAuth2 autorisasjonsserver som gjenbrukes for begge bruksområder.

## Om OpenID Connect

![](/idporten-oidc-dokumentasjon/assets/images/openid.png "OpenID logo")

OpenID Connect er en protokoll for autentisering basert på OAuth2. Se [http://openid.net/connect/faq/](http://openid.net/connect/faq/) for mer informasjon.

## Relevante spesifikasjoner

De implementerte tjenestene bygger på (deler av) følgende standarder og spesifikasjoner:

* OpenID Connect Core 1.0 - [http://openid.net/specs/openid-connect-core-1_0.html](http://openid.net/specs/openid-connect-core-1_0.html)
* IETF RFC6749 The OAuth 2.0 Authorization Framework - [https://tools.ietf.org/html/rfc6749](https://tools.ietf.org/html/rfc6749)
* IETF RFC7523 JSON Web Token (JWT) Profile for OAuth 2.0 Client Authentication and Authorization Grants - [https://tools.ietf.org/html/rfc7523](https://tools.ietf.org/html/rfc7523)


