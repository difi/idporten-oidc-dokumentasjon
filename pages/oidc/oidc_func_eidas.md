---
title: eIDAS
description: eIDAS-søtte over OIDC
summary: "Klienter kan også motta europeiske brukere ihht eIDAS-reguleringen fra ID-portens OIDC-provider. "
permalink: oidc_func_eidas.html

layout: page
sidebar: oidc
---

# Oversikt eIDAS-støtte over OIDC:

ID-porten tilbyr to typer eidas-støtte over OIDC:

* **Enkel**: Her sees eIDAS på lik linje med en norsk eID, slik at tjenesten kun vil motta innlogginger der eIDAS-brukeren er blitt entydig gjenkjent i Folkeregisteret med F/D-nummer. De fleste tjenester vil ønske denne oppførselen
* **Avansert**:  Tjenesten kan selv styre hvilken eIDAS-oppførsel de vil ha, ved å sende ulike parametre som del av autentiseringsforespørselen


Alternativene er oppsummert i følgende tabell:

|Alternativ ||	Parametre tilstades i autentiseringsforespørsel	||||Respons||
|-|-|-|-|-|-|-|-|-|-|
|Flyt	|Krever gjenkjenning (1)|login_hint = eidas:true|scope = edias (utlevere eidas-kjerneattributter)| claims{identitymatch} (aksepterte tilleggsgjenkjenningsalgoritmer) | claims {eidas_*} (sektor-spesifikke-attributter)|eidas_identitymatch (Benytta gjenkjenningsalgoritme)|acr Sikkerhetsnivå|
|-|-|-|-|-|-|-|
|Enkel|JA|ja|nei|nei|nei|tom  (2)|	Level3 Level4|
|Avansert	|JA|	ja|	ja	|nei|	mulig|	tom (2)	|Level3 Level4 |
|||||BEST_EFFORT| mulig | UNAMBIGUOUS BEST_EFFORT|Level3 Level4 |
|Avansert	|NEI	|ja	|ja	|NOT_FOUND	|mulig| UNAMBIGUOUS NOT_FOUND ERROR |Level3 Level4|
|||||NOT_FOUND BEST_EFFORT|	mulig|UNAMBIGUOUS BEST_EFFORT NOT_FOUND ERROR|Level3 Level4|

(1) Ved "krever gjenkjenning", vil ID-porten vise en feilside dersom bruker ikke ble gjenkjent ihht forespurt gjenkjenningsstyrke

(2) UNAMBIGUOUS vil bli brukt, men IdentityMatch returneres ikke dersom ikke den er forspurt.



# Tilgjengelig eIDAS-funksjonalitet i autentiseringsforespørsel

Se http://openid.net/specs/openid-connect-core-1_0.html#AuthRequest for korrekt syntax og valideringsregler.

## 1: eIDAS-støtte

Alle som vil motta eidas-pålogging sender inn "{eidas:true}" som "login_hint".

Standard gjenkjenningsalgoritme basert på entydig, identifikator-basert gjenkjenning ('UNAMBIGUOUS')  vil bli forsøkt. Dersom ingen folkeregisterperson ble gjenkjent, vil innloggingsflyten da stoppe med at ID-porten OIDC-provider viser en feilmelding ("This service require a norwegian D-number, but none could be found" (Denne oppførselen kalles "kreve gjenkjenning").

eIDAS sikkerhetstnivå "high" mappes til "Level4" og nivå "substantial" mappes til "Level3" i acr-claimet i id_token.

Det er ingen sentral tilgongsstyring i OIDC provider på kven som skal ha tilgong, alle som vil ha får tilgong til eidas.

Ein gong i fremtida (2020?) vil ID-porten aktivere enkel eidas-støtte for alle OIDC-tenester.

## 2: Utlevere eidas kjerneattributter
Ved å sende 'eidas' som et scope i autentiseringsforespørsel, vil eidas kjerneattributter (Minimum Data Set) verte utlevert i id_token:

* 4 obligatoriske eidas attributter (PersonIdentifier, fornavn, etternavn, fødselsdato)
* 5 valfrie eidas attributter (om desse eksisterer) (todo)

Denne funksjonaliteten medfører implisitt aktivering av "avansert" eidas-oppførsel, der mao: resterende funksjonalitet i dette kan avsnittet kan også då benyttast.

## 3: Forespørre tilleggsgjenkjenningsalgoritmer  (herunder "kreve gjenkjenning")
Klienter kan forespørre ekstra gjenkjenningsalgoritmer, som vil bli forsøkt i tillegg til standard-oppførselen med entydig identifikator-basert gjenkjenning ('UNAMBIGUOUS').

Dette gjøres ved å bruke standard OIDC-funksjonalitet for å forespørre claims i id_token, se http://openid.net/specs/openid-connect-core-1_0.html#ClaimsParameter .  Klienten må inkludere en array over ønska identitymatch-verdier, slik:

```
claims=
{
  "id_token":
     {
      "identitymatch": { "values": ["besteffort", "notfound"] }
     }
}

```


Følgende verdier er per idag mulig å sende inn i forespørsel

* BEST_EFFORT: Dersom unambiguous ikke gir treff, vil gjenkjenning bli forsøkt basert på navn+fødselsdato.
* NOT_FOUND: Deaktivere "kreve gjenkjenning", mao: vil motta eIDAS-brukere som ikke er gjenkjent
* NOT_FOUND BEST_EFFORT: En kombinasjon av de to foregående.

Den algoritmen som ligger til grunn for norsk personidentifikator i reponsen vil bli utlevert i id_token som et claim `eidas_identitymatch`. Verdien "ERROR" kan også utleveres, her har det skjedd en feil i gjenkjenningsprosedyren og/eller integrasjonen mellom ID-porten og Folkeregisteret, og klienten kan ikke tolke fravær av norsk folkeregisteridentifikator som at eidas-brukeren ikke har F/D-nummer.

Dersom verdien "NOT_FOUND" er tilstede i array'en over forespurte gjenkjenningsalgoritmer, medfører dette at standardoppførselen "kreve gjenkjenning" blir deaktivert, og tjenesten vil også kunne motta ikkje-gjenkjente eIDAS-brukere. Claimet "pid" vil da ikke være tilstede i id_token. Ikke-gjenkjente eIDAS-brukeres sikkerhetsnivå mappes fremdeles til norske nivåer. 'sub'-claimet vil være en pairwise verdi basert på eidas-PersonIdentifier, som medfører at dersom samme eidas-brukere senere blir gjenkjent, vil 'sub' endre seg.

##  4: Forespørre sektor-spesifikke attributter
eIDAS-infrasturen kan transparent overføre attributter som ikke er del av eIDAS-spesifikasjonen, såkalte sektor-spesifikke attributter.  En klient forespør slike attributter ved å prefixe dem med "eidas_" og be om dem som i claims i autentiseringsforespørselen, se forrige avsnitt. Dette er kun mulig dersom scope=eidas også er forespurt.

ID-porten vil da be om disse attributtene fra den utenlandske IDP'en. Ikke alle land/IDPer kan fremskaffe attributtene, men de som ID-porten eventuelt mottar, vil bli utlevert i ID-token.

## Eksempel
```
https://oidc.difi.no/authorize?             
   ...              
   &scopes=openid eidas             
   &login_hint={eidas:true}             
   &claims={"idtoken":{"eidas_sector_att_1":null, "eidas_sector_att_2":null, "identitymatch":{ "values": ["besteffort", "notfound"] }}}      
    ...                                          
```
