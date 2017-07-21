---
title: SSO og SLO
description: SSO og SLO
summary: "Single Signon (SSO) og Single Logout er støttet ved bruk av OIDC."
permalink: oidc_func_sso.html

layout: page
sidebar: oidc
---

## Om funksjonaliteten

ID-portens OIDC Provider har ingen egen Single Signon-sesjon. Istedet benytter den seg av den eksisterende SSO-sesjonen i den SAML2-baserte ID-porten.

## Single Signon (SSO)

Dette medfører at innbygere kan får SSO ikke bare mellom OIDC-baserte tjenester, men også mellom OIDC og SAML2-baserte tjenester. En annen følge av gjenbruken, er at sesjonslevetid er felles for alle tjenester uavhengig av sikkerhetsnivå, og denne er da 30 minutter, men kan forlenges uten brukerinteraksjon inntil maksimalt 120 minutter, ved å sende en ny autentiseringsforespørsel.

Alle tjenester er i utgangspunktet med i samme circle-of-trust, men tjenester kan tvinge frem re-autentisering ved å sette attributten *prompt* til `login` i [autentiseringsforespørselen](http://openid.net/specs/openid-connect-core-1_0.html#AuthRequest) (tilsvarende *forceAuth* i SAML2)


## Single Logout (SLO)

SLO i OpenID Connect IDC skiller seg noe fra SAML2, og er etter Difis vurdering en
mer robust metode enn SAML2 sin kjede av redirects.

Vi baserer oss på følgende spesifikasjoner:

* [OIDC Session Management](http://openid.net/specs/openid-connect-session-1_0.html)
* [OIDC Front Channel Logout](http://openid.net/specs/openid-connect-frontchannel-1_0.html)

Merk at begge disse spesifikasjonene er per juli 2017 i draft status.


{% include note.html content="TODO Her trengs det grundigere dok." %}

- kva må TE gjere?
  - post logout redirect uri?
  - lage en usynlig iframe ?

- korleis virker det i detalj.
  - end Session endepunktet
  
