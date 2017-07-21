---
title: Autentisering til SPA'er
description: Bruk av Idporten sin OpenID Connect provider til autentisering til Single  Page Applikasjoner
summary: "Ved innlogging til en SPA, er det anbefalt å bruke implicit flow, siden en SPA ikke kan beskytte klient-hemmelighet/virksomhetssertifikater på en trygg måte."
permalink: oidc_auth_spa.html

layout: page
sidebar: oidc
---

## Overordna beskrivelse av bruksområdet

{% include warning.html content="Støtten for implicit flow er eksperimentell i ID-porten. Gitt sikkerhetsutfordringene er det usikker om vi ønsker å tilby denne funksjonaliteten." %}

Single-page applikasjoner (SPA) har økende popularitet. Disse skiller seg fra tradisjonelle nettjenester ved at SPAen er realisert som en ren javascript-applikasjon i brukers browser, kontra tradisjonelle nettjtenester der en sentral applikasjonserver generer HTML som blir vist i browseren.

En utfordring med SPAer er at de ikke klarer å beskytte klient-hemmeligheten (evt. virksomhetssertifikatets privatnøkkel) siden hele klienten lever i brukers nettleser. SPAer er altså det som i Oauth2-verdenen kalles **public klienter**. For slike klienter er det anbefalt å bruke _implicit flow_, og ID-portens OpenID Connect provider tilbyr slik funksjonalitet.

Sidan det ikke er klient-autentisering ved bruk av implicit flow, kan i prinsippet hvem som helst lage en falsk SPA som bruker samme client-id, og da fremstår som tjenesteeiers tjeneste ovenfor ID-porten. Sikkerheten ved bruk av implisitt-flyten er derfor helt avhengig av at brukeren er tilstede og forstår om han har blitt lurt til å bruke en falsk applikasjon.

Man bør også vurdere om SPAen i det hele tatt har behov for å et id_token fra ID-porten.  En SPA vil alltid måtte aksessere bakenforliggende APIer, enten hos tjenesteeier selv eller hos andre, og kanskje kan man klare seg med "ren" Oauth2, siden autorisasjonsserveren ved utstedelse av access_token uansett vil gjennomføre en autentisering av brukeren.


## Beskrivelse av implisitt-flyten

Implisittflyt skiller seg fra autorisasjonskode-flyten ved at token utleveres direkte fra autorisasjonsendepunktet.  
Sjå [Oauth2 kapittel 4.2](https://tools.ietf.org/html/rfc6749#section-4.2) og Se [OIDC kapittel  3.2.2.1](http://openid.net/specs/openid-connect-core-1_0.html#ImplicitAuthRequest)


<div class="mermaid">
sequenceDiagram
  Sluttbruker  ->> SPA: Klikker login-knapp
  SPA ->> Browser : Redirect med autentiseringsforespørsel
  Browser ->> OpenID Provider: følg redirect...
  note over Sluttbruker,OpenID Provider: Sluttbruker autentiserer seg (og evt. samtykker til førespurte scopes)
  OpenID Provider ->> Browser: Redirect med id_token (og evt. access_token)
  Browser ->> SPA: uthenting av tokens
  note over Sluttbruker,SPA: Innlogget i tjenesten
  opt Eventuell videre bruk
    SPA ->> Eventuelle APIer: API-operasjon med access_token
  end
</div>

* Klienten sender en **autentiseringsforespørsel** til OpenID Connect provideren med _response_type_ lik `id_token` eller `id_token token`.  Bruk av _nonce_ er påkrevd.   
* Providerens **autorisasjonsendepunkt** validerer forespørselen (f.eks. gyldig tjeneste og gyldig redirect_uri tilbake til tjenesten).
* Brukeren gjennomfører **innlogging i provideren**
* Provideren redirect'er brukeren tilbake til tjenesten. Redirect url'en inneholder **id_token** direkte som et URI fragment.
* Brukeren er nå autentisert for tjenesten og ønsket handling kan utføres




## Autentiseringsforespørsel til autorisasjons-endepunktet

Denne er tilsvarende som ved [autorisasjonskodeflyten](oidc_auth_codeflow.html), med følgende forskjeller:

| Parameter  | Verdi |
| --- | --- |
| response_type | Må vere `id_token` eller `id_token token`|
| nonce | Påkrevd.|


Etter at brukeren har logget inn vil det sendes en redirect url tilbake til klienten. Denne url'en vil inneholde et autorisasjonskode-parameter som kan brukes til oppslag for å hente tokens.


### Eksempel på forespørsel

{% include note.html content="Ikkje oppdatert" %}

```
GET /authorize

  scope=openid&
  acr_values=Level3&
  client_id=test_rp_yt2&
  redirect_uri=https://eid-exttest.difi.no/idporten-oidc-client/authorize/response&
  response_type=id_token&
  state=min_fine_state_verdi&
  nonce=min_fine_nonce_verdi&
  ui_locales=nb
```

### Eksempel på respons:

{% include note.html content="Ikkje oppdatert" %}

```
{
  "code" : "1JzjKYcPh4MIPP9YWxRfL-IivWblfKdiRLJkZtJFMT0=",
  "state" : "min_fine_state_verdi"
}
```



## Struktur på Id token

ID-tokenet er identisk som ved bruk av [autorisasjonskode-flyten](oidc_auth_codeflow#idtoken).


## Validering av Id token

Korrekt validering av Id token på klientsiden er kritisk for sikkerheten i løsningen. Tjenesteleverandører som tar i bruk tjenesten må utføre validering i henhold til kapittel *3.1.3.7 - ID Token Validation* i OpenID Connect Core 1.0 spesifikasjonen.




## Andre aspekt

Implicit flow tillater ikke bruk av refresh_token, siden den ikke kan beskytte dette på en god måte over lengre tid.

En alternativ løsning til implicit flow er at tjenesteeier setter en minimal  backend-tjeneste imellom ID-porten og SPAen og bruker autorisasjonskode-flyten til innlogging. ID-porten kan da være sikker på at sterk identitet kun bli utlevert til riktig tjenesteeier.   id_token vil kun flyte til backend-tjenesten, som oversetter til en lokal sesjon/tokens mellom SPAen og egen APIer.  For eksterne APIer kan ID-porten utstede access_tokens by reference.
