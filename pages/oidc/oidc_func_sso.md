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

### Initiering av SLO (Session Management)

Tjenesteeier ber om logout med en redirect til ID-porten OIDC Provider endsession-endepunkt.  Adressen til endepunktet er definert i ID-porten OpenID Connect-tjenestens konfigurasjonsendepunkt.  ID-porten OIDC Provider sender en redirect til *post_logout_redirect_uri*, dersom denne er angitt og definert for klient.

### Motta informasjon om SLO (Front Channel Logout)

ID-porten OIDC Provider støtter Front Channel Logout og kan sende melding til tjenesteiere integrert med OIDC som støtter dette.  

ID-porten OIDC Provider samler opp informasjon om hvilke tjenester en bruker benytter infor en sesjon.  For OIDC-klienter som støtter Front Channel Logout, sender ID-porten OIDC Provider en GET-forespørsel til klientens *frontchannel_logout_uri*.  Parameterne *iss* og *sid* inkluderes for klienter som krever *frontchannel_logout_session_required*.

Klient som starter utlogging med kall på endsession-endepunktet, mottar ikke melding via front channel logout.

### Samspill mellom SAML SLO og OpenID Connect SLO

ID-porten OIDC provider er integrert med ID-portens via SAML.  

Dersom en tjenesteeier starter SAML SLO, vil ID-porten OIDC Provider delta i SAML SLO.  ID-porten OIDC Provider finner alle klienter i samme sesjon, og sender melding til de av klientene som støtter Front Channel Logout.  Deretter sendes brukeren tilbae til ID-porten.

Dersom en tjenesteeier starter utlogging med OIDC endsession, vil ID-porten sende melding til de av klientene som støtter Front Channel Logout.  Den vil også sende beskjed om SAML utlogging til ID-porten.  Avslutningsvis vil det sendes en redirect til OIDC-klienten, dersom denne er konfigurert for det.

