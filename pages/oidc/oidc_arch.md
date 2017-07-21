---
title: OIDC arkitektur
description: Arkitekturen til ID-portens OIDC Provider
summary: "OIDC Provideren til ID-porten er realisert som en frittstående applikasjon 'foran' ID-porten"
permalink: oidc_arch.html

layout: page
sidebar: oidc
isHome: true
---

## Autentiseringstjenester i ID-porten

Arkitekturen for den nye løsningen ser slik ut:

<div class="mermaid">
graph LR
  subgraph Difi
    subgraph Eksisterende funksjonalitet
      idp[ID-porten]
      end
    OIDC[OIDC Provider]
  end
  subgraph Kunde
     ny[Nye tjenester]
     gammel[Eksiterende tjenester]
  end
  ny --  OpenID Connect  --- OIDC
  gammel --  SAML2 ---idp
  OIDC -- SAML2 ---idp
</div>

ID-portens OIDC provider tilbyr **autentisering** av sluttbrukere opp mot netttjenester.  Funksjonaliteten er grunnleggende den samme som dagens SAML2-basert løsning.

ID-portens OIDC Provider er en frittstående applikasjon som står foran den eksisterende ID-porten og snakker SAML2 med denne, tilsvarende eksisterende tjenester hos kundene.

Det er ID-porten som håndterer SSO-sesjoner både for SAML2 og OIDC.  Dette medfører at kunder får [single-signon (SSO)](oidc_func_sso.html) både mellom OIDC-baserte tjenester, og mellom SAML2- og OIDC-baserte tjenester.

## Autorisasjonstjenester

OpenID Connect-provideren kan også utstede **autorisasjoner** for API-tilgang hos 3dje.part.    

<div class="mermaid">
graph LR
  subgraph 3djepart
    API
  end
  subgraph Difi
    OIDC[OIDC Provider]
  end
  subgraph Kunde
     ny[Tjeneste]
  end
  OIDC -->|3.utsteder token|ny
  Innbygger ---|2.autentiserer og autoriserer|OIDC
  ny -->|1. forspør tilgang|OIDC
  ny -->|4.bruker token mot|API
</div>

API-tilgangen kan være innloggingsbasert (implisitt samtykke), brukerstyrt (eksplisitt samtykke), eller maskin-til-maskin-basert. I de to første tilfellene gjelder autorisasjonen kun en enkelt innbygger, mens det siste tilfellet er tiltenkt hjemmelsbaserte autorisasjoner.


## Oauth2-beskytta APIer fra Difi

<div class="mermaid">
graph LR
  subgraph Eksisterende funksjonalitet
    idp[ID-porten]
    Oppslagstjenesten
  end
  subgraph Oauth2-beskytta APIer
    KRR[KRR-Oauth2]
    authlevel
  end
  authlevel --- idp
  KRR -- SOAP --- Oppslagstjenesten
</div>

Difi tilbyr to Oauth2-beskytta APIer:

* [KRR-Oauth2](oidc_api_krr.html) replikerer funksjonaliteten og begrepene i Oppslagstjenesten
* [authlevel](oidc_api_authlevel.html) er et nytt API for utlevering av innbyggers høyeste brukte sikkertsnivå i ID-porten.  
