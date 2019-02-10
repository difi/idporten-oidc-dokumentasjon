---
title: Autentisering til mobil-app'er
description: Bruk av Idporten sin OpenID Connect provider til autentisering til mobil-app'er
summary: "Ved innlogging til en mobil-app, er det anbefalt å bruke PKCE sammen med autorisasjonskode-flyten"
permalink: oidc_auth_app.html

layout: page
sidebar: oidc
isHome: true
---

## Overordna beskrivelse av bruksområdet

Tilsvarende [Single-page applikasjoner](oidc_auth_spa.html), så kan ikke en mobil-app beskytte hemlighetene sine på en tilfredstillende måte. Den er altså en Oauth2 public klient, og kalles ofte **native app** i oauth2-dokumentasjonen.

I [OAuth 2.0 for Native Apps](https://tools.ietf.org/html/draft-ietf-oauth-native-apps-12) er det gitt anbefalinger for hvordan bruke Oauth2/OpenID Connect på mobil-app'er. Vi anbefaler kunder  å studere dette dokumentet nøye. Vi vil trekke frem følgende:
* Autentisering må skje i ekstern browser (ikke embedded web-view).
* PKCE må brukes for å beskytte seg mot app'er som kan sniffe trafikken og stjele dialogen.
* Implisittflyt er _ikke_ anbefalt for mobil-app'er.

En mobil-app vil typisk integerere med kundens eget API ("app backend"). Det er opp til kunden om han vil bruke ID-portens access_token til sikring av dette APIet direkte, eller omsette ID-portens token til sine egne tokens.

## Sentral oversikt og revokasjon

Difi krever at kunder som omsetter punkt-autentiseringen fra ID-porten til en langt-levende innlogging, oppfyller følgende krav:

- Har lokal sikring av app vha. touchID, ansiktsgjenkjenning el. lignende
- Aktive innlogginger skal vises på sentrale oversikt i ID-portens brukerprofil
- Håndterer innbygger-initiert revokasjon av innloggingen fra ID-portens brukerprofil

Oversikt over innlogginger med revokasjonsmulighet for innlogget innbygger er tilgjengelig på et API, slik at kunden kan velge å også tilby slik funksjonalitet i egen selvbetjeningsløsning.

### Teknisk beskrivelse

Den enkleste måten å håndtere kravene ovenfor, er som følger:
- Kunde oppretter eget *oauth2 scope* med innbygger-venlig beskrivelse og egen, valgfri levetid
- Ved innlogging mottar klienten access_token med dette scopet (mao. ikke id_token som vanlig) og lagrer dette trygt i egen backend så lenge innloggingen er gyldig
- Validerer at access_tokenet fremdeles er gyldig ved å sjekke det mot ID-portens /tokeninfo-endepunkt ved brukerhandlinger
- På grunn av den lange levetiden, bør ikke ID-portens token flyte ut til app'en, men istedet omsettes til egne token for sikring mellom app og egen backend


## Beskrivelse av flyten for mobil-app'er

Flyten er identisk som for [autorisasjonskode-flyten](oidc_auth_codeflow.html), men med bruk av [PKCE](oidc_func_pkce.html).


### Autentiseringsforespørsel til autorisasjons-endepunktet

Følgende ekstra attributter må settes på request:

| Parameter  | Verdi |
| --- | --- |
| t(code_verifier) | hash'et hemmelighet |
| code_challenge_method | Kun `S256` er støttet i ID-porten.  |


### Utstedelse av token fra token-endepunktet

Følgende ekstra attributter må sendes inn i requesten:

| Attributt  | Verdi |
| --- | --- |
| code_verifier | hemmelighet i klartekst. Påkrevd dersom t(code_verifier) ble sendt i autentiseringsforspørsel. |


## Struktur på Id token

ID-tokenet er identisk som ved bruk av [autorisasjonskode-flyten](oidc_auth_codeflow#idtoken).
