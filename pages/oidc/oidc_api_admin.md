---
title: OAuth2 beskytta REST-API for administrasjon av OIDC-integrasjoner
description: API som gir tjenesteleverandørar mulighet til å administrere sine OIDC-integrasjoner.
summary: "Oauth2-beskyttet REST-grensesnitt som gir utvalgte kunder mulighet til å selv-administrere sine OIDC/Oauth2-integrasjoner i ID-porten."
permalink: oidc_api_admin.html

layout: page
sidebar: oidc
---


## Introduksjon

Utvalgte OIDC-klienter kan få tilgang til å administrere integrasjonar i ID-porten. APIet muliggjør for eksempel:
* Kunde kan opprette/endre nye klienter knyttet til eget org.nummer
* Leverandør kan opprette/endre selvstendige klienter knyttet til egne kunder
* Leverandør kan opprette/endre onbehalfof-klienter på vegne av egne kunder

## Hvordan få tilgang ?

Ta kontakt med idporten@difi.no for å få tilgang til å bruke tjenesten.


## Bruk av Oauth2{#scopes}

APIet er sikret vha. [server-til-server Oauth](https://difi.github.io/idporten-oidc-dokumentasjon//oidc_auth_server-to-server-oauth2.html), dvs. med bruk av virksomhetssertifikat.

Klienten må få tildelt scopes for å få tilgang til APIet:

| scope | beskrivelse |
|-|-|
|idporten:dcr.read|Gir tilgang til å lese klientregistreringer for klienter bundet mot samme org.nr. som gitt i access_token. Gir også lesetilgang til onbehalfof-registreringer|
|idporten:dcr.modify|Gir tilgang til å endre klientregistreringer for klienter bundet mot samme org.nr. som gitt i access_token. Gir også lesetilgang til onbehalfof-registreringer|
|idporten:dcr.write|Gir tilgang til å opprette nye klientregistreringer for klienter bundet mot samme org.nr. som gitt i access_token. Gir også lesetilgang til onbehalfof-registreringer|
|idporten:dcr/onbehalfof:write|Gir tilgang til å vise, opprette, endre og slette onbehalfofregistreringer tilhørende en gitt klient. Gir ikke mulighet til å endre andre parametere på selve klienten.|
|idporten:dcr/supplier|Gir leverandører tilgang til å vise, opprette, endre og slette selvstendige OIDC-integrasjoner for andre organisasjoner. Eget org.no blir koblet til disse integrasjonene.  |


## Eierskap til integrasjoner

Leverandører kan velge to måter å integrere sine kunder på:

### 1: onbehalfof-integrasjoner

Bruke *onbehalfof* "under-integrasjoner" knyttet til Leverandørens egen integrasjon, som dokumentert [her](/oidc_func_onbehalfof.html).

### 2: Selvstendige integrasjoner

Med selvstendige integrasjoner har hver integrasjon har egen client_id og klientautentisering.    
* Leverandøren må sette client_orgno lik egen kunde sitt organisasjonsnummer.
* Leverandøren kan opprette integrasjoner på vilkårlige client_orgno
* Leverandørens eget organisasjonnummer blir *automatisk* satt som `supplier_orgno` (basert på virksomhetssertifikatet som blir brukt mot admin-APIet)
* (På sikt vil) access_tokens utsted til klienten vil innholde både leverandørens og kundens organisasjonsnummer.

Ved endring og sletting tillater APIet kun operasjoner på integrasjoner der eget orgno er lagret som supplier_orgno fra før.


## Ulike typer integrasjonar

Man kan opprette to typer integrajonser over APIet:

### 1: OIDC-integrasjon for person-autentisering

Dersom minimum følgende claims er tilstede ved opprettelse/endring, vil klienten bli aktivert for person-innlogging.

|claim|beskrivelse|
|-|-|
|client_name|Navn på klient, blir vist ved innlogging|
|description|Beskrivelse av klienten, ikke synlig for innbyggere, men blir lagret i Difis støttesystemer|
|client_uri|URL til klient (blir brukt på tilbake-knapp og ved feil)|
|logo_uri| URL til logo som vises ved innlogging|
|scopes| Må være minst `["openid"]`|
|redirect_uris| liste med redirect-uri'er|

### 2: Maskin-til-maskin-integrasjonar

Registreringer som ikke oppfyller kravene i forrige avsnitt, blir opprettet som maskin-til-maskin integrasjoner.  Personinnlogging vil ikke fungere.

## Rotering av client_secret

For integrasjoner som bruker symmetrisk nøkkel (client_secret) som klientautentiseringsmetode, kan man generere ny secret ved å kalle [/clients/{client_id}/secret](https://integrasjon-ver2.difi.no/swagger-ui.html#/oidc-client-controller/updateSecretUsingPOST)

Merk: Difi vil på sikt innføre maks-levetid på client_secret.

## REST-grensesnittet

Se Open-API dokumentasjon her:

[https://integrasjon-ver2.difi.no/swagger-ui.html#/](https://integrasjon-ver2.difi.no/swagger-ui.html#/)

Merk at ID-porten vil opprette client_id og client_secret.


## Eksempel

### Eksempel på å lese klientregistrering:

```
GET /clients/oidc_eksempel_klient
Accept: application/json
Authorization: Bearer <my_access_token_value>


Respons:

{
	"client_id": "oidc_eksempel_klient",
	"active": true,
	"client_orgno": "991825827",
	"display_name": "En tilfeldig eksempelklient",
	"redirect_uris": ["https://eksempel.no/login", ],
	"post_logout_redirect_uris": ["https://eksempel.no/logout"],
	"scopes": ["openid", "profile"],
	"default_scopes": [],
	"authorization_lifetime": 7200,
	"access_token_lifetime": 300,
	"refresh_token_lifetime": 7200,
	"last_updated": "2017-11-02 T15:02:32 +0100",
	"client_type": "CONFIDENTIAL",
	"token_reference": "OPAQUE",
	"frontchannel_logout_session_required": false,
	"onbehalfof": [{
			"onbehalfof": "example_onbehalfof",
			"display_name": "Eksempelregistrering for onbehalof"
			"orgno": "991825828"
			"logo-url": "https://service.eksempel.no/logo.png"
			"url": "https://service.eksempel.no"
		}, {
			"onbehalfof": "example_onbehalfof_2",
			"display_name": "En annen eksempelregistrering for onbehalof"
			"orgno": "991825829"
			"logo-url": "https://otherservice.eksempel.no/logo.png"
			"url": "https://otherservice.eksempel.no"
		}
	],
	"force_pkce": false
}
```

### Eksempel på å lese onbehalfof-registrering:

Forespørsel
```
GET /clients/oidc_eksempel_klient/onbehalof/example_onbehalof
Accept: application/json
Authorization: Bearer <my_access_token_value>
```

Respons:
```
Status code 200

{
	"onbehalfof": "example_onbehalfof",
	"display_name": "Eksempelregistrering for onbehalof"
	"orgno": "991825828"
	"logo-url": "https://service.eksempel.no/logo.png"
	"url": "https://service.eksempel.no"
}

```

### Eksempel på å opprette onbehalfof-registrering:

Forespørsel
```
POST /clients/oidc_eksempel_klient/onbehalof/
Accept: application/json
Authorization: Bearer <my_access_token_value>
{
	"onbehalfof": "new_example_onbehalof",
	"display_name": "Eksempelregistrering for onbehalof"
	"orgno": "991825828"
	"logo-url": "https://service.eksempel.no/logo.png"
	"url": "https://service.eksempel.no"
}
```

Respons:
```
Status code 200

{
	"onbehalfof": "new_example_onbehalof",
	"display_name": "Eksempelregistrering for onbehalof"
	"orgno": "991825828"
	"logo-url": "https://service.eksempel.no/logo.png"
	"url": "https://service.eksempel.no"
}

```

###Eksempel på å endre onbehalfof-registrering:

Forespørsel
```
PUT /clients/oidc_eksempel_klient/onbehalof/example_onbehalof
Content-type: application/json
Authorization: Bearer <my_access_token_value>

{
	"onbehalfof": "example_onbehalof",
	"display_name": "Modified display_name value"
	"orgno": "991825828"
	"logo-url": "https://service.eksempel.no/logo.png"
	"url": "https://service.eksempel.no"
}
```

Tilsvarende respons som ved nyregistrering

### Eksempel på å slette onbehalfof-registrering:

Forespørsel
```
DELETE /clients/oidc_eksempel_klient/onbehalof/example_onbehalof
Authorization: Bearer <my_access_token_value>
```

Får respons med statuskode 200, og tom body.
