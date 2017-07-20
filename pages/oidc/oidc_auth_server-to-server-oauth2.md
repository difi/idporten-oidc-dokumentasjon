---
title: Server-til-server autorisasjon med Oauth2
description: Server-til-server autorisasjon med Oauth2
summary: "Bruk av ID-porten sin OpenID Connect Provider til autorisasjon for kommunikasjon mellom server til server"
permalink: oidc_auth_server-to-server-oauth2.html

layout: page
sidebar: oidc
---

## Introduksjon

ID-porten sin OpenID Connect provider tilbyr funksjonalitet for server-til-server autorisasjon av API'er basert på [RFC7523 JSON Web Token (JWT) Profile for OAuth 2.0 Client Authentication and Authorization Grants](https://tools.ietf.org/html/rfc7523).

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
  OIDC -->|2.utsteder token|ny
  ny -->|1. forspør tilgang|OIDC
  ny -->|3.bruker token mot|API
</div>

Kunder og API-eiere kan bruke denne funksjonaliteten for å styre tilgang i de tilfellene der informasjonsverdiene APIet tilbyr er regulert av lovhjemmel, og ikke krever samtykke av brukeren.

## Beskrivelse av flyt

<div class="mermaid">
sequenceDiagram
  note over Klient:  Generer og signer JWT
  Klient ->> OpenID Provider: Bruk JWT til å forespørre token
  note over OpenID Provider: Valider virksomhetssertifak og utfør tilgangskontroll
  OpenID Provider ->> Klient: Returnere access_token
  Klient ->> API: Bruk token mot API
  opt eventuelt
    API ->> OpenID Provider: validere token
    OpenID Provider ->> API: token info
  end
  API ->> Klient: Resultat av API-kall

</div>


I dette scenariet ønsker en **klient** å bruke en **ressurs (API)** tilbudt av en **ressursserver**. Tilgangen (autorisasjonen) til api'et blir utstedt av en **autorisasjonsserver**, i dette tilfellet ID-portens OpenID Connect Provider. For å aksessere ressursen må klienten forespørre et access_token fra autorisasjonsserveren som klienten så kan bruke til aksessere den aktuelle ressursen.

* Flyten starter med at klienten må generere en **JWT-basert tokenforespørsel** (JWT-bearer authorization grant). Denne inneholder informasjon om hvilke ressurser (*scope*) klienten ønsker å aksessere og blir signert med klienten sitt virksomhetssertifikat.

* Når autorisasjonsserveren mottar tokenforespørselen vil den først **validere gyldigheten av JWT'en**. Deretter vil virksomhetssertifikatet (brukt til signering av JWT'en) valideres og en **klientautentisering** utføres på bakgrunn av dette.

* Dersom den autentiserte klienten har tilgang til de forespurte ressursene returneres et **access_token** til klienten

* Klienten kan nå aksessere den ønska ressursen ved bruk av access_tokenet.

* Ressursserveren må nå validere det mottatte access_tokenet mot autorisasjonsservern sitt tokeninfo-endepunkt.  Dersom tokenet er såkalt self-contained, er dette steget unødvendig.

* Dersom access_tokenet er gyldig kan det forespurte ressursen returneres til klienten.

## Krav til JWT for token-forespørsel

Klienten må generere og signere ein jwt med følgende elementer for å forespørre tokens fra autorisasjonsserveren:


**Header:**

| Parameter  | Verdi |
| --- | --- |
| x5c | Inneholde klientens virksomhetssertifikat som er brukt for signering av JWT'en |
| alg | RS256 - Vi støtter kun RSA-SHA256 som signeringsalgoritme |

&nbsp;

**Body:**

| Parameter  | Verdi |
| --- | --- |
|aud| Audience - identifikator for ID-portens OIDC Provider.  Se ID-portens `well-known`-endepunkt for aktuelt miljø for å finne riktig verdi. |
|iss| issuer - client ID som er registert hos ID-porten OIDC-provider|
|scope| Scope som klient forespør tilgang til, kan sende inn liste av scope separert med whitespace|
|iat| issued at - tidsstempel for når jwt'en ble generert - **MERK:** Tidsstempelet tar utgangspunkt i UTC-tid|
|exp| expiration time - tidsstempel for når jwt'en utløper - **MERK:** Tidsstempelet tar utgangspunkt i UTC-tid **MERK:** ID-porten godtar kun maks levetid på jwt'en til 120 sekunder (exp - iat <= 120 )|
|jti| Optional - JWT ID - unik id på jwt'en som settes av klienten. **MERK:** JWT'er kan ikke gjenbrukes. ID-porten håndterer dette ved å sammenligne en hash-verdi av jwt'en mot tidligere brukte jwt'er. Dette impliserer at dersom klienten ønsker å sende mer enn en token-request i sekundet må jti elementet benytttes.|

&nbsp;

**Eksempel på JWT struktur:**

```
{
  "x5c": [ "MIIFOjCCBCKgAwI``````BXYF56Q==" ],
  "alg": "RS256"
}
.
{
  "aud": "https://eid-vag-opensso.difi.local/idporten-oidc-provider/",
  "scope": "global/kontaktinformasjon.read global/varslingsstatus.read",
  "iss": "test_rp",
  "exp": 1477924061,
  "iat": 1477923951,
}
.
Bsox4L7yqGi3d1HhW1yXTZGV95MAAFQTWr2J1iF5svzg1v5G-WwM3iGUYvgWlqG-ZrnIPu4
jNpuwrqFRWXyFgmSHCmKx9e-7pkDQlZMDwZfYC5VxDQzS1YvIkDir9H12AhF9b64REbG5sY
Za8gK64c4lshbk9biyGcihi5jWDENdA-Rb_eJipiOYHrDHGOjcN5GTN2XASfd1UEeET2mT-
mysPd4CUp99ol74cl3lhAviMReLI2kMTmFus8SBozHm3aGJysU2TyX7fBBS7MxbF4Hk7c6s
thN1oRgnpsziWIg08NDKW2cYOAIHBvz9bBf0D_dhi5kQsm9ippyrtgs5Q
```

## Endepunkter

ID-porten auth.server tilbyr følgende endepunkter:

### Token-endepunkt

Token-endepunktet benyttes for utstedelse av tokens basert på JWT-bearer grants.

```
URL: https://<miljø>/idporten-oidc-provider/token
```

Følgende header-parametere må brukes på request:

| Parameter  | Verdi |
| --- | --- |
|Http-metode:|POST|
|Content-type:|application/x-www-form-urlencoded|

&nbsp;

Følgende attributter må sendes inn i requesten:

| Attributt  | Verdi |
| --- | --- |
|grant_type|urn:ietf:params:oauth:grant-type:jwt-bearer|
|assertion|\<Den genererte JWT'en for token-requesten\>|

&nbsp;

Eksempel på forespørsel:

```
POST /token
Content-type: application/x-www-form-urlencoded

grant_type=urn%3Aietf%3Aparams%3Aoauth%3Agrant-type%3Ajwt-bearer&assertion=<jwt>
```

&nbsp;

Eksempel på respons:

```
{
    "access_token": "fK0dhs5vQsuAUguLL2wxbXEQSE91XbOAL3foY5VR0Uk=",
    "expires_in": 599,
    "scope": "global/kontaktinformasjon.read"
}
```

### Tokeninfo-endepunkt

Tokeninfo-endepunktet benyttes av ressursservere for validering av gyldigheten til mottatte tokens.

```
URL: http://eid-exttest.difi.no/idporten-oidc-provider/tokeninfo
```

&nbsp;

Følgende header-parametere må brukes på request:

| Parameter  | Verdi |
| --- | --- |
|Http-metode:|POST|
|Content-type:|application/x-www-form-urlencoded|

&nbsp;

Følgende attributter må sendes inn i requesten:

| Attributt  | Verdi |
| --- | --- |
|token|\<Tokenet som skal valideres\>|

&nbsp;

Eksempel på request:

```
POST /tokeninfo
Content-type: application/x-www-form-urlencoded

token=fK0dhs5vQsuAUguLL2wxbXEQSE91XbOAL3foY5VR0Uk=
```

&nbsp;

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

&nbsp;

Eksempel på en respons ved feilet validering av token:

```
{
    "active": false
}  
```

## Eksempel på generering av JWT for token-forespørsel i Java

Nimbus JOSE + JWT er et hendig bibliotekt for å håndtere jwt'er i JAVA , se [http://connect2id.com/products/nimbus-jose-jwt](http://connect2id.com/products/nimbus-jose-jwt)

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

&nbsp;

For .net og andre platformer gir [jwt.io](http://jwt.io) en fin oversikt over tilgjengelige biblioteker
