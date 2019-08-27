---
title: SSO og SLO
description: SSO og SLO
summary: "Single Signon (SSO) og Single Logout er støttet ved bruk av OIDC."
redirect_to: https://difi.github.io/felleslosninger/oidc_func_sso.html

layout: page
sidebar: oidc
---

## Om funksjonaliteten

ID-portens OIDC Provider er integrert med ID-porten SAML sin Circle-of-trust, og støtter både SSO og SLO mellom OIDC- og SAML-tjenester.



## Single Signon (SSO)

SSO-sesjonen blir styrt av ID-porten SAML. Dette medfører at innbyggere kan få SSO ikke bare mellom OIDC-baserte tjenester, men også mellom OIDC og SAML2-baserte tjenester. En annen følge av gjenbruken, er at sesjonslevetid er felles for alle tjenester uavhengig av sikkerhetsnivå, og denne er da 30 minutter, men kan forlenges uten brukerinteraksjon inntil maksimalt 120 minutter, ved å sende en ny autentiseringsforespørsel.

Alle tjenester er i utgangspunktet med i samme circle-of-trust, men tjenester kan tvinge frem re-autentisering ved å sette attributten *prompt* til `login` i [autentiseringsforespørselen](http://openid.net/specs/openid-connect-core-1_0.html#AuthRequest) (tilsvarende *forceAuth* i SAML2)


## Single Logout (SLO)

SLO i OpenID Connect skiller seg noe fra SAML2, og er etter Difis vurdering en mer robust metode enn SAML2 sin kjede av redirects.

Vi baserer oss på følgende spesifikasjoner:

* [OIDC Session Management](http://openid.net/specs/openid-connect-session-1_0.html)
* [OIDC Front Channel Logout](http://openid.net/specs/openid-connect-frontchannel-1_0.html)

Merk at begge disse spesifikasjonene er per juli 2017 i draft status.

### Initiering av SLO (Session Management)

Når brukeren vil logge ut fra din tjeneste, må du sende en redirect til ID-porten OIDC Provider endsession-endepunkt.  Adressen til endepunktet er definert i konfigurasjonsendepunktet.  ID-porten OIDC Provider sender en redirect til *post_logout_redirect_uri*, dersom denne er angitt og definert for klient, og *id_token_hint* er inkludert.  Dersom disse mangler, vil brukeren ende opp i ID-porten.

Følgende attributer kan være del av requesten:

|attributt|kardinalitet | beskrivelse|
|---|---|---|
|```id_token_hint``` | anbefalt | Settes lik mottatt id-token.  Nødvendig for å kunne sende brukeren tilbake til tjenesteeiers *post_logout_redirect_uri* etter endt utlogging.|
|```post_logout_redirect_uri```| anbefalt | Må være forhåndsregistrert på klient som id_token er utstedt til |
|```state``` | valgfri | Verdi som klient kan bestemme selv.  ID-porten vil inkludere denne tilbake i redirecten tilbake til utloggings-urlen |


Eksempel:
```
https://oidc-ver2.difi.no/idporten-oidc-provider/endsession
	?id_token_hint=eyJraWQiOiJpZ2I1Q3lGT...
	&post_logout_redirect_uri=<min registrerte post-logout uri >
	&state=fe93c125-4d69-4ee3-8ca5-299ac6e3e499
```

### Motta informasjon om SLO (Front Channel Logout)

Dersom brukeren logger ut fra en annen tjeneste, vil ID-porten initere utlogging fra alle tjenester som er konfigurert med støtte for Front Channel Logout.  

ID-porten OIDC Provider samler opp informasjon om hvilke tjenester en bruker benytter infor en sesjon.  For OIDC-klienter som støtter Front Channel Logout, sender ID-porten OIDC Provider en GET-forespørsel til klientens *frontchannel_logout_uri*.  Parameterne *iss* og *sid* inkluderes for klienter som krever *frontchannel_logout_session_required*.  *sid* har samme verdi som claim *sid* i id_token.

Klient som starter utlogging med kall på endsession-endepunktet, mottar ikke melding via front channel logout.

Dersom en klient ikke er konfigurert med Front Channel Logout, vil klienten ikke motta utloggingsforespørsel fra ID-porten dersom brukeren logger ut fra en annen tjeneste i circle-of-trust.

Eksempel på kall fra ID-porten til klient:
```
GET https://client.example.com/myapp/logout
     ?iss=https://oidc-ver2.difi.no/idporten-oidc-provider/
     &sid=D8Fgz-jEXG7JXP_VAORmAm1sKB0LjZyA3wAy-rVyMYc=
```
`sid` er ID-porten OIDC sin sesjons-id som klient også har mottatt som claim i id-token.
## Samspill mellom SAML SLO og OpenID Connect SLO

ID-porten OIDC provider er integrert med ID-portens via SAML.  

Dersom en tjenesteeier starter SAML SLO, vil ID-porten OIDC Provider delta i SAML SLO. ID-porten OIDC Provider finner alle klienter i samme sesjon, og sender melding til de av klientene som støtter Front Channel Logout. Deretter sendes brukeren tilbake til ID-porten, for avsluttende redirect tilbake til tjenesteeieren som startet SAML SLO.

Dersom en tjenesteeier starter utlogging med OIDC endsession, vil ID-porten sende melding til de av klientene som støtter Front Channel Logout.  Den vil også sende beskjed om SAML utlogging til ID-porten.  Avslutningsvis vil det sendes en redirect til OIDC-klienten, dersom denne er konfigurert for det.


### Eksempel på SLO-utlogging startet fra OIDC-tjeneste

<div class="mermaid">
sequenceDiagram
	Initierende tjeneste OIDC ->> ID_porten OIDC: /endsession
	ID_porten OIDC ->> OIDC tjenester : GET frontchannel_logout_uri

	ID_porten OIDC ->> ID_porten SAML: LogoutRequest
	ID_porten SAML ->> SAML tjenester: LogoutRequest
	SAML tjenester ->> ID_porten SAML: LogoutResponse
	ID_porten SAML ->> ID_porten OIDC: LogoutResponse
	ID_porten OIDC ->> Initierende tjeneste OIDC: redirect post_logout_redirect_uri

</div>

### Eksempel på SLO-utlogging startet fra SAML-tjeneste

<div class="mermaid">
sequenceDiagram
	Initierende tjeneste SAML ->> ID_porten SAML: LogoutRequest
	ID_porten SAML ->> ID_porten OIDC: LogoutRequest
	ID_porten OIDC ->> OIDC tjenester : GET frontchannel_logout_uri
	ID_porten OIDC ->> ID_porten SAML: LogoutResponse
	ID_porten SAML ->> SAML tjenester: LogoutRequest
	SAML tjenester ->> ID_porten SAML: LogoutResponse
	ID_porten SAML ->> Initierende tjeneste SAML: LogoutResponse

</div>
