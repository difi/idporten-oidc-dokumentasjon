---
title: OpenID Connect i ID-porten
description: OpenID Connect i ID-porten
summary: "Ta kontakt med oss på idporten@difi.no for å komme i gang!"
permalink: index.html

layout: page
sidebar: oidc
---

## Hvem kan bruke ID-porten OpenID Connect provider?
Tjenesteeiere som har godtatt ID-porten sine bruksvilkår kan ta i bruk denne tjenesten i produksjonsmiljø.
Man kan gjerne teste løsningen i testmiljø før man godtar bruksvilkårene.
Se bruksvilkår i [samarbeidsportalen](https://samarbeid.difi.no/bruksvilkar/bruksvilkar-difis-felleslosninger)

## Rutine for tilgang
Følg stegvis veiledning på [samarbeidsportalen](https://samarbeid.difi.no/felleslosninger/id-porten/ta-i-bruk-id-porten)

### Planlegging
Følgende punkter bør være en del av planleggingen (gjøres i samarbeid med Difi)

    Tidsplan
    Produksjonsplan
    Beskrivelse av tjenesten
    Forventninger omkring volum og bruk, herunder eventuelle høy-trafikkperioder
    Påvirkning på ID-porten sin brukerstøtte

## Etablerering i test og produksjon
Difi krever at tjenester som skal beskyttes av ID-porten må gjennom et verifikasjonsløp før tjenesten kan produksjonssettes. Det er derfor nødvendig at tjenesteleverandør fødererer en testbasert versjon av sin tjeneste i Difi sitt verifikasjonsmiljø. Her skal det kun benyttes fiktive data, og ikke reelle brukere. Ved test av varsling pr sms og/eller e-post må tjenesteleverandør passe på at en bruker egnet kontaktinformasjon da varslingen sender ut reelle meldinger til eieren av kontaktinformasjonen.

### Verifikasjonstester
Difi har etablert et standardsett av tester for verifisering og godkjenning av integrasjon mot ID-porten. Tjenesteleverandør skal utføre denne og bekrefte til Difi at testing er utført ok før tjenesten kan i produksjon.  [Les om verifikasjonstester ](https://difi.github.io/idporten-integrasjonsguide//96_verifikasjonstest.html)

### Bruk av sertifikater
Anbefalt autentiseringsmetode vil være klientautentisering basert på JWT'er signert med virksomhetssertifikater. Metoden krever at tjenesteeier anskaffer test-virksomhetssertifikat for testmiljø, og virksomhetssertifikat for produksjonsmiljø. Det kreves at tjenesteleverandør benytter nøkler utstedt som virksomhetssertifikater iht. [kravspesifikasjon PKI](https://www.difi.no/fagomrader-og-tjenester/digitalisering-og-samordning/standarder/referansekatalogen/bruk-av-pki-med-og-i-offentlig-sektor), og at sertifikatutstederen er selvdeklarert for utstedelse av virksomhetssertifikater hos Nasjonal kommunikasjonsmyndighet (NKOM). Pr dags dato er det bare Buypass og Commfides som er selvdeklarert hos NKOM, og dermed kun disse som kan utstede gyldige virksomhetssertifikater for bruk mot ID-porten.

### Tjenesteeiers logo
Innloggingbildet i ID-porten viser tjenesteeiers logo. Logo i rett format må derfor utveksles med Difi på forhånd.

### Informasjon om Difi sine miljøer
I tillegg til produksjonsmiljøet tilbyr Difi tre testmiljøer; ”verifikasjon 1”, ”verifikasjon 2” og ”ytelsestestmiljø”. Testmiljøene er beregnet til funksjonell testing av integrasjoner. Les om bruk av de ulike miljøene i [samarbeidsportalen](https://samarbeid.difi.no/node/232)

### Testbrukere
For VER1 og VER2 vil allerede tildelte testbrukere kunne gjenbrukes. Ta kontakt med **idporten@difi.no** dersom du mangler testbukere.

## Fremgangsmåte
 1.  Be om å få en klient-integrasjon ved å sende mail til  idporten@difi.no.  Husk å oppgi ønskede redirect-uri'er og annen nødvendig informasjon, se [klient-registrering](oidc_func_clientreg.html)

 2. Konfigurer din føderasjonsprogramvare med informasjonen mottatt i punkt 1, og pek den mot ID-portens well-known endepunkt.
 3. Det skal nå være mulig å logge inn

### Well-known endepunkt:

|Miljø|URL|
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

Vi ber kunder om å studere [OAuth 2.0 Threat Model and Security Considerations](https://tools.ietf.org/html/rfc6819) og
[OpenID Connect Basic Client Implementer's Guide 1.0](https://openid.net/specs/openid-connect-basic-1_0.html).
Kunder som skal lage mobil-apper bør lese [OAuth 2.0 for Native Apps](https://tools.ietf.org/html/rfc8252).


En ikke-uttømmende liste over biblioteker og programvare som skal støtte OpenID Connect finnes [hos OpenID Foundation](http://openid.net/developers/certified/). Per juli 2017 har vi notert oss at iallefall IdentityServer4, .Net Core og ADFS fungerer opp mot ID-portens OIDC Provider.

## Kommentarer, feil eller problemer ?

Difi foretrekker at kunder [oppretter GitHub-issues i repoet *difi/idporten-oidc-dokumentasjon*](https://github.com/difi/idporten-oidc-dokumentasjon/issues) dersom de har problemer, opplever feil, eller har kommentarar til dokumentasjon og funksjonalitet, slik at kunnskap blir delt med andre etater.

Ta ellers kontakt på epost på idporten@difi.no.  Se og kontaktinfomasjonen på [samarbeidsportalen](https://samarbeid.difi.no/).
