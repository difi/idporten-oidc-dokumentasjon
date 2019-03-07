---
title: OAuth2 beskytta REST-API for innloggingshistorikk i ID-porten
description: API for innloggingshistorikk i ID-porten
summary: "Innbygger sin innloggingshistorikk fra ID-porten kan konsumeres via API og synliggjøres for innbygger i andre tjenester. Dette gir innbygger innsyn i bruk av egne data fra Difis løsninger"
permalink: oidc_api_logghistorikk.html

layout: page
sidebar: oidc
isHome: true
---

## Introduksjon

Innbygger sin innloggingshistorikk fra ID-porten kan konsumeres via API og synliggjøres for innbygger i andre tjenester.

Kunden mottar opplysningene for at innbyggeren kan lettere deteketere misbruk av eID.

## Hvordan få tilgang ?

Denne tjenesten er en tilleggstjeneste i ID-porten. Se [https://samarbeid.difi.no/difis-felleslosninger/tilleggstjenester](https://samarbeid.difi.no/difis-felleslosninger/tilleggstjenester) for generelle vilkår for tilleggstjenester.


## Bruk av Oauth2

REST-grensesnittet er sikret vha. [autentiseringsnær autorisasjon](https://difi.github.io/idporten-oidc-dokumentasjon/oidc_auth_oauth2.html).


Følgende scopes aksepteres av grensesnittet:

| scope | beskrivelse |
| - | - |   
| `idporten:user.log.read`  |  Lese ut innloggingshistorikk  som innlogget bruker har til mine tjenester  |


## REST-grensesnittet

REST-grensesnittet er basert på at innkommende access token tilhører innlogget bruker, kunden trenger derfor ikke oppgi fødselsnummer selv.  

Kunden får kun se innloggingshistorikk til tjenester tilknyttet eget organisasjonsnummer.

Grensesnittet er dokumenter vha. Swagger [her-TBD]().

## Eksempel

```
GET /logg/idporten?maxhits=5 HTTP/1.1
Authorization: Bearer U_3uI4GjxKbhc4jo54mbnozbSZdHLyw28ZQGnxsScmI=

[
    {
        "tidspunkt": "2019-03-07T09:26:36",
        "tjeneste": "DIREKTORATET FOR FORVALTNING OG IKT",
        "eid": "MinID PIN",
        "integrasjon_id": "oidc_authorization_api_client_test",
        "orgno": "991825827"
    },
    {
        "tidspunkt": "2019-03-06T16:04:40",
        "tjeneste": "DIREKTORATET FOR FORVALTNING OG IKT",
        "eid": "Federated",
        "integrasjon_id": "oidc_authorization_api_client_test",
        "orgno": "991825827"
    }
]

```
