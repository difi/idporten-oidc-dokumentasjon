---
title: eIDAS
description: eIDAS-søtte over OIDC
summary: "Klienter kan også motta europeiske brukere ihht eIDAS-reguleringen fra ID-portens OIDC-provider. "
sidebar: oidc
permalink: oidc_func_eidas.html 
redirect_to: https://difi.github.io/felleslosninger/oidc_func_eidas.html
---
Flyttet til https://difi.github.io/felleslosninger/oidc_func_eidas.html| ERROR | Det oppstod en feil ved forsøket på å koble utenlandsk eID mot norsk personidentifikator (Dette kan skje feks ved feil i kommuniksajonen mellom ID-porten og Det Sentrale Folkeregisteret|
|NOT_FOUND | Ingen treff ved forsøk på kobling av utenlandsk eID mot norsk personidentifikator i Det Sentrale Folkeregister)|
|SELF_DECLARED | Norsk personidentifikator er basert på opplysninger som brukeren selv har oppgitt (Brukeren har for eksempel koblet sin Facebook-konto med ID-porten|



Ved bruk av tilleggsgjenkjenningsalgoritmer vil  tjenesteeier kunne motta to identifikatorer på brukerne:


  | acr | eidas-personidentifier | pid |  Beskrivelse |
  | --- | ---- | --- | --- | --- |
  | eidas | xx/NO/yyyyy… | <tomt> |  Personen har autentisert seg med europeisk eID. Norsk D-nummer ble ikke funnet.|
  | eidas | xx/NO/yyyyy… | norsk folkeregisteridentifikator | Personen har autentisert seg med europeisk eID og har norsk D/F-nummer. |
  | [En av disse](https://begrep.difi.no/ID-porten/SAMLAssertionV1)| <tomt> | norsk folkeregisteridentifikator | Personen har autentisert seg med vanlig norsk eID. |



Merk av ved bruk av tilleggsgjenkjenningsalgoritmer, vil fravær av verdi i feltet *pid* ikke  garantere at personen ikke har fått tildelt d/f-nummer.

## Utenlandske testbrukere

Det er dessverre ikke mange land som tilbyr dedikerte testbrukere ennå.  Vi anbefaler tjenesteeiere å velge *Sverige* som innloggingsland, og deretter velge "Test ID-tjänst",  her vil man få en nedtrekksliste med tilgjengelige testbrukere.  

Den først i lista, Mohamed Al Samed, er hardkoda i den norske eIDAS Noden til å bli  entydig gjenkjent med norsk D-nummer 59125502061.  ID-token for en autentisering på eidas-nivå 'substantial' vil se slik ut:

```
 "amr" : [ "eIDAS" ],
 "pid" : "59125502061",
 "eidas-identitymatch" : "UNAMBIGUOUS",
 "eidas-personidentifier" : "SE/NO/199008199391",
 "eidas-firstname" : "Mohamed",
 "eidas-familyname" : "Al Samed",
 "eidas-dateofbirth" : "1990-08-19",
 "acr" : "Level3",
 ```

## Integrerte land i produksjonsmiljøet

Per Januar 2019 er Estland koblet på i produksjonsmiljøet.    Vi forventer å ha Tyskland, Italia, Portugal, Spania og Luxemburg i løpet av 2019, etterhvert som de blir formelt *notifisert* og fagfellevurdert av EU-kommisjonen.  For en oppdatert status, se EU-kommisjonen sin side: [https://ec.europa.eu/cefdigital/wiki/display/EIDCOMMUNITY/Overview+of+pre-notified+and+notified+eID+schemes+under+eIDAS](https://ec.europa.eu/cefdigital/wiki/display/EIDCOMMUNITY/Overview+of+pre-notified+and+notified+eID+schemes+under+eIDAS)

I testmiljøet har vi for tiden 19 land integrert.
