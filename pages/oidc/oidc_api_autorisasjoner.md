---
title: OAuth2 beskytta REST-API for autorisasjoner i OIDC-provider
description: API for autorisasjoner i OIDC provider
summary: "Innbygger sine autorisasjoner i ID-portens OIDC provider er tilgjengelig på et REST-api, for kundens egen oversikt.  Typiske autorisasjoner er såkalte langt-levende innlogginger til mobil-apper."
permalink: oidc_api_autorisasjoner.html

layout: page
sidebar: oidc
isHome: true
---

## Introduksjon

Enkelte av innbygger sine autorisasjoner i ID-porten OIDC provider er tilgjengelig på et enkelt REST-api.

Typiske autorisasjoner som er tilgjengelig over APIet er såkalte langt-levende innlogginger til mobil-apper, men teknisk er det slik at ethvert Oauth2 scope kan "tagges" for å synes på lista av scope-eier, og vil da bli tilgjengelig.

Kunden mottar opplysningene for å vise disse til innbygger i egne løsninger, og evt. gi innbygger anledning til å revokere en autorisasjonen.  Et annet typisk bruksmønster er der innbygger tar kontakt med kundens brukerstøtte, som da trenger å fjerne en autorisasjon på vegne av innbyggeren.

## Hvordan få tilgang ?

Denne tjenesten er en tilleggstjeneste i ID-porten. Se [https://samarbeid.difi.no/difis-felleslosninger/tilleggstjenester](https://samarbeid.difi.no/difis-felleslosninger/tilleggstjenester) for generelle vilkår for tilleggstjenester.


## REST-grensesnittet

REST-grensesnittet er basert på at innkommende access token tilhører innlogget bruker,  såkalt  [autentiseringsnær autorisasjon](https://difi.github.io/idporten-oidc-dokumentasjon/oidc_auth_oauth2.html), kunden skal derfor ikke oppgi fødselsnummer selv.

Oauth2 scopet som gir tilgang til grensesnittet er `idporten:authorizations.read` eller `idporten:authorizations.revoke`.

Grensesnittet er dokumentert vha. Swagger [her-TBD]().

REST-grensesnittet tilbyr to hovedmetoder:

### Hente innbyggers autorisasjoner

Dette kallet henter alle innbyggers synlige autorisasjoner til klienter (tjenester) som tilhører kundens organisasjonsnummer.  Eksempel:

```
GET /authorizations
Authorization: Bearer xxxxx

[
    {
        "authorization_id": "oHZZlFkupHTerMfC6uyHPMHXF_stX7wFwFMvq4reaC4=",
        "client_id": "oidc_authorization_api_client_test",
        "client_name": "En eksempel-tjeneste hos Difi",
        "authorized_at": 1551358681942,
        "expires": 1581358681942,
        "scopes": [
            {
                "name": "idporten:authorizationstest.read",
                "description": "Hei, jeg er en synlig autorisasjon!"
            }
        ],
        "user_agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/72.0.3626.119 Safari/537.36"
    }
]

```

Autorisasjonene kan tilhøre scopes som kunden selv eier, samt scopes som eies av en tredjepart.


### Slette en autorisasjon

Normalt vil en kunde slette en autorisasjon ved at klienten som fikk utstedt det aktuelle tokenet kaller /revoke-endepunktet til ID-porten med tokenet (access eller fortrinnvis refresh) som skal slettes.

Dersom kunden ønsker å revokere fra en annen klient enn den som fikk tokenet utdelt, må denne klienten kalle
```
DELETE /authorizations/{authorization_id}
Authorization: Bearer yyyyy
```
Bearer-tokenet i forespørselen må ha `idporten:authorizations.revoke` scope.
