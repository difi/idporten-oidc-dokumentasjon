---
title: Autentisering med bruk av autorisasjonskode-flyten
pageid: authentication-using-code-flow
layout: default
description: Bruk av Idporten sin OpenID Connect provider til autentisering med autorisasjonskode-flyten
isHome: false
---

## Introduksjon

ID-porten sin OpenID Connect provider tilbyr funksjonalitet for autentisering av sluttbrukere basert på autorisasjonskode-flyten, slik den er spesifisert i OpenID Connect Core 1.0 spesifikasjonen

## Overordna beskrivelse av scenariet

OpenID Connect tilbyr autentisering av brukere til sluttbrukertjenester. Autentiseringen blir utført av ein OpenID Connect provider som utseder ID Token til den aktuelle tjenesten.

Følgende aktører inngår:

* *Sluttbrukeren* \- Ønsker å logge inn til en gitt tjeneste (relaying_party)
* *Relying party* \- sluttbrukertjenesten som brukeren skal logge inn til
* *OpenID Connect provider* \- Autentiseringstjeneste der autentiseringen blir utført og som usteder *ID Token* til sluttbrukertjenesten

![](/idporten-oidc-dokumentasjon/assets/images/openid_scenario.png "OpenID Connect scenario")


## Flyt

![](/idporten-oidc-dokumentasjon/assets/images/openid_auth_code_flow.png "Sekvensdiagram som viser server-til-server Oauth2-flyten")

 
## Endepunkter

ID-porten OpenID Connect tilbyr følgende endepunkter:


### Autorisasjons-endepunkt

Klienten redirecter sluttbrukeren til autorisasjonsendepunktet. Etter at 

```
URL: http://eid-exttest.difi.no/idporten-oidc-provider/authorization
```

Følgende header-parametere må brukes på request:

| Parameter  | Verdi |
| --- | --- |
|Http-metode|GET|

Følgende attributter må settes på request:

| Parameter  | Verdi |
| --- | --- |
|grant_type| Her støtter vi kun _authorization\_code_ |
| client\_id | Klientens tildelte id |
| redirect\_uri | URI som sluttbruker skal redirectes tilbake til etter fullført authentisering. Kun forhåndsregistrerte url'er kan brukes |
| scope | Scope som forespørres. Kan være en liste separert med whitespace. For autentiseringer må _openid_ brukes |
| state | Verdi som settes av klient og returneres i callback-responsen etter fullført autentisering. Kan benyttes til å implementere CSRF-beskyttelse |
| nonce | Verdi som settes av klient og returneres som en del av ID token. Kan brukes til å binde en klient-sesjon til et gitt ID-token, og hindre replay attacks  |
| acr\_values | Ønsket sikkerhetsnivå, kan være *Level3* eller *Level4* |
| ui\_locales | Ønsket språk brukt i Id-porten. støtter *nb*, *nn*, *en* eller *se* |
| prompt | Brukes til å styre providerens interaksjon med sluttbrukeren TODO: Må beskrive lovlige verdier og oppførsel |


### Token-endepunkt

Token-endepunktet brukes for utstedelse av tokens. Endepunktet støtter flere typer klient-autentisering

```
URL: http://eid-exttest.difi.no/idporten-oidc-provider/token
```

Følgende header-parametere må brukes på request:

| Parameter  | Verdi |
| --- | --- |
| Http-metode | POST |
| Content-type | application/x-www-form-urlencoded |


Følgende attributter må sendes inn i requesten:

| Attributt  | Verdi |
| --- | --- |
| grant_type | authorization\_code \| refresh\_token|
| code | authorization\_gode dersom dette benyttes som grant |
| client_id | Klientens ID |
| client_secret | 

Eksempel på forespørsel:

```
POST /token
Content-type: application/x-www-form-urlencoded
 
grant_type=urn%3Aietf%3Aparams%3Aoauth%3Agrant-type%3Ajwt-bearer&assertion=<jwt>
```

Eksempel på respons:

```
{
    "access_token": "fK0dhs5vQsuAUguLL2wxbXEQSE91XbOAL3foY5VR0Uk=",
    "expires_in": 599,
    "scope": "global/kontaktinformasjon.read"
}
```

### Tokeninfo-endepunkt

Tokeninfo-endepunktet benyttes av sluttbrukertjenester for å validere gyldigheten av access_tokens

```
URL: http://eid-exttest.difi.no/idporten-oidc-provider/tokeninfo
```

Følgende header-parametere må brukes på request:

| Parameter  | Verdi |
| --- | --- |
|Http-metode:|POST|
|Content-type:|application/x-www-form-urlencoded|

Følgende attributter må sendes inn i requesten:

| Attributt  | Verdi |
| --- | --- |
|token|\<Tokenet som skal valideres\>|

Eksempel på request:

```
POST /tokeninfo
Content-type: application/x-www-form-urlencoded
 
token=fK0dhs5vQsuAUguLL2wxbXEQSE91XbOAL3foY5VR0Uk=
```
 
Eksempel på en respons ved suksessfull validering av token:

```
{
    "active": true,
    "token_type": "Bearer",
    "expires_in": 556,
    "exp": 1477990301,
    "iat": 1477989701,
    "scope": "global/kontaktinformasjon.read",
    "client_id": "test_rp",
    "client_orgno": "991825827"
}
```
 
Eksempel på en respons ved feilet validering av token:

```
{
    "active": false
}  
```
 
## Eksempel på generering av JWT for token-forespørsel i Java

Nimbus JOSE + JWT er et hendig bibliotekt for å håndtere jwt'er i JAVA , se http://connect2id.com/products/nimbus-jose-jwt

Her er ein enkel eksempelkode for å generere en JWT for å forespørre tokens:

```
PrivateKey myKey = ``` // Read from KeyStore
X509Certificate certificate = ``` // Read from KeyStore
 
List<Base64> certChain = new ArrayList<>();
certChain.add(Base64.encode(certificate));
 
JWSHeader jwtHeader = JWSHeader.Builder(JWSAlgorithm.RS256)
        .x509CertChain(certChain)
        .build();
 
JWTClaimsSet claims = new JWTClaimsSet.Builder()
        .audience("https://eid-vag-opensso.difi.local/idporten-oidc-provider/")
        .issuer("clientId")
        .claim("scope", "scope_to_request")
        .jwtID(UUID.randomUUID().toString())
        .issueTime(new Date(Clock.systemUTC().millis()))
        .expirationTime(new Date(Clock.systemUTC().millis() + 120000)) // Expiration time is 120 sec.
        .build();
 
JWSSigner signer = new RSASSASigner(myKey);
SignedJWT signedJWT = new SignedJWT(jwtHeader, claims);
signedJWT.sign(signer);
 
String serializedJwt = signedJWT.serialize();
```