---
title: OpenID Connect i ID-porten
description: Pilot av OpenID Connect funksjonalitet i Id-porten
summary: "ID-porten lanserer i 2017 en ny protokoll for integrasjon: OpenID Connect"
permalink: index.html

layout: page
sidebar: oidc
isHome: true
---

## Bakgrunn

ID-porten støtter idag autentisering av innbyggere basert på SAML2-protokollen. Dette er et teknologivalg som i praksis ble tatt midt på 2000-tallet. De siste 10 årene har det skjedd en betydlig teknologiutvikling, som gjør at det er naturlig å nå se på alternative løsninger.

En av de viktigste driverne for å vurder alternative protokoller er fremveksten av API-drevne tjenester samt nye mobile platformer. SAML2 er designet for tradisjonelle webtjenester, og er på mange måter lite egnet for bruk på disse nye tjeneste-typene.

Selv etter 10 år med SAML2, opplever Difi fremdels at enkelte virksomheter bruker betydlige ressurser på å sette opp en ordinær SAML2-integrasjon også for tradisjonelle webtjenester. Det er derfor naturlig å evaluere om det finst andre og mer moderne protokoller som har både høy utbredelse og høy sikkerhet.

Difi introduserser derfor OpenID Connect (OIDC) som ny føderasjonsprotokoll i ID-porten fra høsten 2017. Vi tar samtidig ibruk den underliggende protokollen Oauth2 til å tilby alterantivt sikrede grensesnitt inn mot våre APIer til Kontakt- og Reservasjonsregisteret.

## Om OpenID Connect

![](/idporten-oidc-dokumentasjon/images/oidc.png "OpenID Connect logo")

OpenID Connect er en protokoll for autentisering basert på OAuth2. Se [http://openid.net/connect/faq/](http://openid.net/connect/faq/) for mer informasjon.

De implementerte tjenestene bygger på (deler av) følgende standarder og spesifikasjoner:

* OpenID Connect Core 1.0 - [http://openid.net/specs/openid-connect-core-1_0.html](http://openid.net/specs/openid-connect-core-1_0.html)
* OpenID Connect Discovery
[http://openid.net/specs/openid-connect-discovery-1_0.html](http://openid.net/specs/openid-connect-discovery-1_0.html)
*OpenID Connect Session Management
[http://openid.net/specs/openid-connect-session-1_0.html](http://openid.net/specs/openid-connect-session-1_0.html)
*OpenID Connect Front-Channel Logout
[http://openid.net/specs/openid-connect-frontchannel-1_0.html](http://openid.net/specs/openid-connect-frontchannel-1_0.html)
*OAuth 2.0 Form Post Response Mode
[http://openid.net/specs/oauth-v2-form-post-response-mode-1_0.html](http://openid.net/specs/oauth-v2-form-post-response-mode-1_0.html)
*OAuth 2.0 Token Introspection
[https://tools.ietf.org/html/rfc7662](https://tools.ietf.org/html/rfc7662)
*Proof Key for Code Exchange by OAuth Public Clients
[https://tools.ietf.org/html/rfc7636](https://tools.ietf.org/html/rfc7636)

* IETF RFC6749 The OAuth 2.0 Authorization Framework - [https://tools.ietf.org/html/rfc6749](https://tools.ietf.org/html/rfc6749)
* IETF RFC7523 JSON Web Token (JWT) Profile for OAuth 2.0 Client Authentication and Authorization Grants - [https://tools.ietf.org/html/rfc7523](https://tools.ietf.org/html/rfc7523)
