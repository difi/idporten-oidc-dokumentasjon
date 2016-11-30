---
title: Pilot av Id-porten OpenID Connect provider
pageid: pilot-introduction
layout: default
description: Pilot av OpenID Connect funksjonalitet i Id-porten
isHome: true
---

## Introduksjon og bakgrunn for pilot

Difi tilbr nå en skarp pilot av OpenID Connect funksjonalitet i ID-porten. Denne skal gjøre det lettere å realisere en del scenarier som er tunge å implementere med dagens SAML2 baserte grensesnitt i ID-porten. 

## Om OpenID Connect

![](/idporten-oidc-dokumentasjon/assets/images/openid.png "OpenID logo")

OpenID Connect er en protokoll for autentisering basert på Oauth2. Se http://openid.net/connect

## Relevante spesifikasjoner

Dei implementerte tjenestene bygger på følgende standarder og spesifikasjoner:

* OpenID Connect Core 1.0 - http://openid.net/specs/openid-connect-core-1_0.html
* OpenID Connect Session Management 1.0 - http://openid.net/specs/openid-connect-session-1_0.html
* IETF RFC6749 The OAuth 2.0 Authorization Framework - https://tools.ietf.org/html/rfc6749
* IETF RFC7523 JSON Web Token (JWT) Profile for OAuth 2.0 Client Authentication and Authorization Grants - https://tools.ietf.org/html/rfc7523

## Beskrivelse av mulige scenarier

### OpenID Connect som alternativ autentiseringprotokoll for SAML2

### Mobil-app autentisering og langt-levende sesjoner.

### Tilgangskontroll og autorisasjon av API'er

* Brukerstyrt tilgang

* Server til server autorisasjon av api'er

![](/idporten-oidc-dokumentasjon/assets/images/server-to-server-oauth2.png "Server til server OAuth2")
