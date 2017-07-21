---
title: Hvordan ta i bruk tjenesten
description: Hvordan ta i bruk tjenesten
summary: "Ta kontakt med oss på idporten@difi.no for å komme i gang!"
permalink: oidc_hvordan_komme_igang.html

layout: page
sidebar: oidc
---

## Hvem kan bruke ID-porten OpenID Connect provider?

Tjenesteeiere som har godtatt ID-porten sine bruksvilkår kan ta i bruk denne tjenesten i produksjonsmiljø.

Man kan gjerne teste løsningen i testmiljø før man godtar bruksvilkårene.

Se
[https://samarbeid.difi.no/difis-felleslosninger/hvilke-virksomheter-kan-ta-i-bruk-difis-felleslosningene](https://samarbeid.difi.no/difis-felleslosninger/hvilke-virksomheter-kan-ta-i-bruk-difis-felleslosningene)
 for mer informasjon om hvordan du kan bli tjenesteeierr i ID-porten

  **Merk:** Dette er en pilottjeneste og vil ikke være omfattet av ID-porten sine normale SLA-krav. Tjenesteleverandører må også påregne at det kommer endringer i tjenesten sine api'er, selv om disse selvsagt vil tilstrebes å være bakoverkompatible.

## Fremgangsmåte
 1.  Be om å få en klient-integrasjon ved å sende mail til  idporten@difi.no.  Hugs å oppgi ønska redirect-uri'er og annen nødvendig informasjon, se [klient-registreing](oidc_func_clientreg.html])

 2. Konfigurer din føderasjonsprogramvare med informasjonen mottatt i punkt 1, og pek den mot ID-portens well-known endepunkt.
 3. Det skal nå være mulig å logge inn

En ikke-uttømmende liste over biblioteker og programvare som skal støtte OpenID Connect finnes [hos OpenID Foundation](http://openid.net/developers/certified/). Per juli 2017 har vi notert oss at iallefall IdentityServer4, .Net Core og ADFS fungerer opp mot ID-portens OIDC Provider.

## Kommentarer, feil eller problemer ?

Difi foretrekker at kunder [oppretter GitHub-issues i repoet *difi/idporten-oidc-dokumentasjon*](https://github.com/difi/idporten-oidc-dokumentasjon/issues) dersom de har problemer, opplever feil, eller har kommentarar til dokumentasjon og funksjonalitet, slik at kunnskap blir delt med andre etater.

Ta ellers kontakt på epost på idporten@difi.no.  Se og kontaktinfomasjonen på [samarbeidsportalen](https://samarbeid.difi.no/).

## Informasjon om Difi sine miljøer

OpenID Connect-provideren er på plass i de fleste av Difi sine miljøer.


### Well-known endepunkt:
|miljø|url|
|-|-|
|VER1|[https://oidc-ver1.difi.no/idporten-oidc-provider/.well-known/openid-configuration](https://oidc-ver1.difi.no/idporten-oidc-provider/.well-known/openid-configuration)|
|VER2|[https://oidc-ver2.difi.no/idporten-oidc-provider/.well-known/openid-configuration](https://oidc-ver2.difi.no/idporten-oidc-provider/.well-known/openid-configuration)|
|YT2|[https://oidc-yt2.difi.eon.no/idporten-oidc-provider/.well-known/openid-configuration](https://oidc-yt2.difi.eon.no/idporten-oidc-provider/.well-known/openid-configuration)|
|PROD|[https://oidc.difi.no/idporten-oidc-provider/.well-known/openid-configuration](https://oidc.difi.no/idporten-oidc-provider/.well-known/openid-configuration)|


Eksempel på konfigurasjon slik den var i VER2 per 2017-06-14:
```
{
	"issuer": "https://oidc-ver2.difi.no/idporten-oidc-provider/",
	"authorization_endpoint": "https://oidc-ver2.difi.no/idporten-oidc-provider/authorize",
	"token_endpoint": "https://oidc-ver2.difi.no/idporten-oidc-provider/token",
	"end_session_endpoint": "https://oidc-ver2.difi.no/idporten-oidc-provider/endsession",
	"revocation_endpoint": "https://oidc-ver2.difi.no/idporten-oidc-provider/revoke",
	"jwks_uri": "https://oidc-ver2.difi.no/idporten-oidc-provider/jwk",
	"response_types_supported": ["code"],
	"response_modes_supported": ["query","form_post"],
	"subject_types_supported": ["pairwise"],
	"userinfo_endpoint": "https://oidc-ver2.difi.no/idporten-oidc-provider/userinfo",
	"scopes_supported": ["openid",	"profile"],
	"ui_locales_supported": ["nb",	"nn",	"en",	"se"]
}
```



## Testbrukere

For ver2 vil allerede tildelte testbrukere kunne gjenbrukes. Ta kontakt med **idporten@difi.no** dersom du mangler testbukere.



## Oauth2-beskyttet endepunkt for kontaktregisteret

Følgende URLer gjelder for KRR:

|---|---|---|
|Miljø|URL|beskrivelse|
|---|---|---|
|VER1|https://oidc-ver1.difi.no/kontaktinfo-oauth2-server/rest/v1/person|Utlevering av enkelt-bruker ifbm. innlogging|
|    |https://oidc-ver1.difi.no/kontaktinfo-oauth2-server/rest/v1/personer|Server-to-server beskyttet endepunkt|
