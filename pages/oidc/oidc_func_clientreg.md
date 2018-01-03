---
title: Registrering av OIDC klienter
description: Registrering av OIDC klienter
summary: "Konfigurasjon av klienter (tjenester) er enkelt ved hjelp av .well-known-endepunktet, men noe informasjon må manuelt utveksles først"
permalink: oidc_func_clientreg.html

layout: page
sidebar: oidc
---

## Klientregistrering

For å ta i bruk tjenesten må en klientregistrering gjennomføres. Foreløpig er dette en manuell prosess og interesserte tjenesteleverandører kan henvende seg til **idporten@difi.no**

Følgende informasjon må registreres om klienter:

| attributt | beskrivelse |
|-|-|
| client_id | Unik identifikator for klienten |
| client_secret | Hemmlighet som benyttes ved *client_secret_basic* klientautentisering. |
| client_orgno | Klientens organisasjonsnummer. Brukes ved validering av virksomhetssertifikat |
| display_name | Klientens organisasjonsnavn som benyttes ved visning på web |
| redirect_uris | Liste over gyldige url'er som provideren kan redirecte tilbake til etter vellykket autorisasjonsforespørsel. |
| post_logout_redirect_uris | Liste over url'er som provideren redirecter til etter fullført utlogging. |

Difi vil tildele *client_secret*. Kunde kan til en viss grad velge *client_id* selv, men vi foretrekker en bestemt syntaks.

I tillegg _kan_ man registrere følgende:

| attributt | beskrivelse |
|-|-|
| scopes | Liste over scopes som klienten kan forespørre. For OpenID Connect er aktuelle scopes *openid* og *profile* |
| authorization_lifetime | Levetid for registrert autorisasjon. I en OpenID Connect sammenheng vil dette være tilgangen til userinfo-endepunktet |
| access_token_lifetime | Levetid for utstedt access_token |
| refresh_token_lifetime |Levetid for utstedt refresh_token |
| frontchannel_logout_uri | URL som provideren sender request til ved utlogging trigget av annen klient i samme sesjon |
| frontchannel_logout_session_required | Flagg som bestemmer om parameterne for issuer og sesjons-id skal sendes med frontchannel_logout_uri |


## Well-known endepunkt

OpenID Connect baserer seg på at aktørene tilgjengeliggjør sin konfigurasjon på et såkalt well-known endepunkt. Her skal både metadata som URLer og sertifikater være tilgjengelig.

ID-portens well-known endepunkt er her:
```
https://oidc.difi.no/.well-known/openid-configuration
```
