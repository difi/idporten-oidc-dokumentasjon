# Server til server autorisasjon med Oauth2

## Introduksjon

ID-porten sin OpenID Connect provider tilbyr funksjonalitet for server-til-server autorisasjon av API'er basert på RFC7523 JSON Web Token (JWT) Profile for OAuth 2.0 Client Authentication and Authorization Grants

## Flyt

![](images/server_to_server_oauth2_flow.png "Sekvensdiagram som viser server-til-server Oauth2-flyten")

## Krav til JWT for token-forespørsel
 
Klienten må generere og signere ein jwt med følgende elementer for å forespørre tokens fra autorisasjonsserveren:


**Header:**

| Parameter  | Verdi |
| --- | --- |
| x5c | Inneholde klientens virksomhetssertifikat som er brukt for signering av JWT'en |
| alg | RS256 - Vi støtter kun RSA-SHA256 som signeringsalgoritme |

**Body:**

| Parameter  | Verdi |
| --- | --- |
|aud| Audience - identifikator for ID-portens OIDC Provider - skal være: https://eid-vag-opensso.difi.local/idporten-oidc-provider/|
|iss| issuer - client ID som er registert hos ID-porten OIDC-provider|
|scope| Scope som klient forespør tilgang til, kan sende inn liste av scope separert med whitespace|
|iat| issued at - tidsstempel for når jwt'en ble generert - **MERK:** Tidsstempelet tar utgangspunkt i UTC-tid|
|exp| expiration time - tidsstempel for når jwt'en utløper - **MERK:** Tidsstempelet tar utgangspunkt i UTC-tid **MERK:** ID-porten godtar kun maks levetid på jwt'en til 120 sekunder (exp - iat <= 120 )|
|jti| Optional - JWT ID - unik id på jwt'en som settes av klienten. **MERK:** JWT'er kan ikke gjenbrukes. ID-porten håndterer dette ved å sammenligne en hash-verdi av jwt'en mot tidligere brukte jwt'er. Dette impliserer at dersom klienten ønsker å sende mer enn en token-request i sekundet må jti elementet benytttes.|

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

```
URL: http://eid-exttest.difi.no/idporten-oidc-provider/token
```

Følgende header-parametere må brukes på request:

| Parameter  | Verdi |
| --- | --- |
|Http-metode:|POST|
|Content-type:|application/x-www-form-urlencoded|

Følgende attributter må sendes inn i requesten:

| Attributt  | Verdi |
| --- | --- |
|grant_type|urn:ietf:params:oauth:grant-type:jwt-bearer|
|assertion|\<Den genererte JWT'en for token-requesten\>|

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