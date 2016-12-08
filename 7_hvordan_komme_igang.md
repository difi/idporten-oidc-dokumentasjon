---
title: Hvordan ta i bruk tjenesten
pageid: getting_started
layout: default
description: Hvordan ta i bruk tjenesten som tjenesteleverandør i ID-porten
isHome: false
hiddenInToc: false
---

## Hvem kan bruke ID-porten OpenID Connect provider?

Registrerte tjenesteleverandører i ID-porten som har godtatt ID-porten sine bruksvilkår kan ta i bruk denne tjenesten.

Se [https://samarbeid.difi.no/id-porten/hvordan-ta-i-bruk-id-porten](https://samarbeid.difi.no/id-porten/hvordan-ta-i-bruk-id-porten) for mer informasjon om hvordan du kan bli tjenesteleverandør i ID-porten

**Merk:** Dette er en pilottjeneste og vil ikke være omfattet av ID-porten sine normale SLA-krav. Tjenesteleverandører må også påregne at det kommer endringer i tjenesten sine api'er.

## Klientregistrering

For å ta i bruk tjenesten må en klientregistrering gjennomføres. Foreløpig er dette en manuell prosess og interesserte tjenesteleverandører kan henvende seg til **idporten@difi.no**

Følgende informasjon må registreres om klienter:

| client_id | Unik identifikator for klienten |
| client_secret | Hemmlighet som benyttes ved *client_secret_basic* klientautentisering. |
| client_orgno | Klientens organisasjonsnummer. Brukes ved validering av virksomhetssertifikat |
| display_name | Klientens organisasjonsnavn som benyttes ved visning på web |
| redirect_uris | Liste over gyldige url'er som provideren kan redirecte tilbake til etter vellykket autorisasjonsforespørsel. |
| post_logout_redirect_uris | Liste over url'er som provideren redirecter til etter fullført utlogging. Foreløpig er denne ikke benyttet i pilottjenesten da denne ikke har sentrale brukersesjoner (for SSO), og dermed heller ikke noe behov for å logge ut av provideren |
| scopes | Liste over scopes som klienten kan forespørre. For OpenID Connect er aktuelle scopes *openid* og *profile* | 
| authorization_lifetime | Levetid for registrert autorisasjon. I en OpenID Connect sammenheng vil dette være tilgangen til userinfo-endepunktet |
| access_token_lifetime | Levetid for utstedt access_token |
| refresh_token_lifetime |Levetid for utstedt refresh_token |
   


## Informasjon om Difi sine miljøer



### eid-exttest.difi.no

eid-exttest.difi.no er et internt testmiljø driftet hos Difi. Dette medfører at dette miljøet ikke har de samme stabilitet og oppetidskrav som andre ID-porten-miljøer.

OpenID Connect provideren sitt discovery-endepunkt finnes her:
[https://eid-exttest.difi.no/idporten-oidc-provider/.well-known/openid-configuration](https://eid-exttest.difi.no/idporten-oidc-provider/.well-known/openid-configuration)

```
{
   "issuer": "https://eid-exttest.difi.no/idporten-oidc-provider/",
   "authorization_endpoint": "https://eid-exttest.difi.no/idporten-oidc-provider/authorize",
   "token_endpoint": "https://eid-exttest.difi.no/idporten-oidc-provider/token",
   "end_session_endpoint": "https://eid-exttest.difi.no/idporten-oidc-provider/endsession",
   "jwks_uri": "https://eid-exttest.difi.no/idporten-oidc-provider/jwk",
   "response_types_supported": [ "code" ],
   "subject_types_supported": [ "pairwise" ],
   "userinfo_endpoint": "https://eid-exttest.difi.no/idporten-oidc-provider/userinfo",
   "scopes_supported": [ "openid", "profile" ],
   "ui_locales_supported": [ "nb", "nn", "en", "se" ]
}
```

### idporten-ver2.difi.no

Tjenesten vil snart være tilgjengelig i ver2.

### idporten.difi.no (ID-porten sitt produksjonsmiljø)

Tjenesten vil snart være tilgjengelig i prod.

## Testbrukere

For ver2 vil allerede tildelte testbrukere kunne gjenbrukes. Ta kontakt med **idporten@difi.no** for tildeling av testdata til eid-extest.difi.no