---
title: OAuth2-basert autorisasjon 
pageid: oauth2-based-authorization
layout: default
description: Autorisasjon 
isHome: false
resource: true
hiddenInToc: false
---

## Introduksjon

I forbindelse med en OpenID Connect-autentisering kan ID-portens OpenID Connect provider også autorisere en tjeneste til å opptre på vegne av innbyggeren opp mot et API tilbudt av en 3.dje-part.

## Overordna beskrivelse av scenariet

Dette er den klassise Oauth2-flyten, der innbyggeren samtykker - enten eksplisitt eller implisitt - til at tjenesten kan bruke et API på vegne av seg selv.  I ID-porten-sammenheng vil vanligvis samtykket være implisitt, siden det er autentiseringshandlingen som i seg selv tolkes som det informerte samtykket ("Ved å logge inn i tjenesten godtar du at vi henter opplysninger om deg fra Vegveseent"). For eksplisitte samtykker som skal vare "lenge" ("jeg samtykker til at Banken min kan hente inntektsopplysninger hos Skatteetaten de neste 3 årene") henviser vi til bruk av Samtykkeløsningen i Altinn.


Samtykket, eller autorisasjonen, blir av ID-porten utlevert som et _access_token_. Tjenesten bruker så dette access_tokenet når den skal aksessere APIet.

Hvilket API/ressurs som skal aksesserers, er styrt av _scopes_.  Klienten må vite hvilke(t) scope som hører til den aktuelle API-operasjonen, og må forespørre dette scopet i autorisasjonsforespørselen.


| --- | --- |
| Roller | Beskrivelse
| --- | --- |
| Klient | Sluttbrukertjenesten, som har behov for API-tilgang |
| Autorisasjonsserver | ID-portens OpenID Connect provider |
| Ressursserver | 3.part, som tilbyr et API som sluttbrukertjenesten ønsker å benytte | 

(oppdatere med figurer)

Autentiseringsflyten ihht. OpenID Connect er dokumentert [her](2_authentication_using_code_flow.html). Forskjellen på OpenIDConnect og "plain" Oauth2 er altså at klienten _må_ forspørre 'openid'-scopet for å trigge oppførsel ihht. OpenID Connect-spesifikasjonen.  I begge flytene kan man forspørre så mange andre scopes man ønsker. 


Eksempler på scopes:

| --- | --- |
| scope | --- |
| --- | --- |
| user/kontaktinformasjon.read | Lese kontaktinformasjon fra Kontakt- og Reservasjonsregistert for innlogget bruker |
| 


## Typer access_token og validering av disse

ID-porten kan utstede to typer access_token.  Ressursservere som mottar access_token som del av en API-forespørsel må validere disse før API-operasjonen kan utføres.

|---|---|
|token|beskrivelse|
|---|---|
|by reference| Tokenet inneholder kun en referanse til autorisasjonen internt i ID-porten.  Ressursserveren må sjekke opp mot tokeninfo-endpunktet til ID-porten for å få om tokenet fremdeles er gyldig, hvem brukeren er, hvilke scopes som ble forespurt, og hvilken klient (inkludert dennes org.nr.) det var utstedt til. |
|by value | Tokenet er såkalt self-contained, dvs. det inneholder all informasjon som man ellers kunne sjekke mot tokeninfo-endpunktet.  Slike tokens kan ikke trekkes tilbake, og bør derfor ha kort levetid |

###  Eksempel på bruk av tokeninfo-endepunktet



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



## Utlevering av kontaktopplysninger 

Digital kontaktinformasjon knyttet til innlogget bruker er tilgjengelig på et eget Oauth2-beskyttet endepunkt, og ikke som del av userinfo-endepunktet.  

Man må forespørre ett eller flere av følgende scopes: *user/kontaktinformasjon.read*, *user/varslingsstatus.read*, *user/digitalpost.read* og *user/sertifikat.read*
og vil da motta et access_token som kan benyttes mot Kontakt- og Reservasjonsregisteret sitt endepunkt:
```
URL: https://oidc-ver2.difi.no/kontaktinfo-oauth2-server/rest/v1/person
```

&nbsp;

Følgende header-parametere må brukes på request:

| Parameter  | Verdi |
| --- | --- |
| Http-metode: | GET |
| Accept: | application/jose  (evt. application/json ) |
| Authorization: | Bearer \<utstedt access_token\> |
 
## Eksempel på respons:


Se https://begrep.difi.no/Oppslagstjenesten/Person for definisjon av kodeverket.


```
      {
         "personidentifikator": "23079421936",
         "reservasjon": "NEI",
         "status": "AKTIV",
		 "varslingsstatus" : "KAN_VARSLES",
         "kontaktinformasjon":
         {
            "epostadresse": "23079421936-test@minid.norge.no",
            "epostadresse_oppdatert": "2010-12-16T13:32:05.000+01:00"
         }
      }
```



