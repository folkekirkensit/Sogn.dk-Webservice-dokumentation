# Webservices til kirkelige begivenheder og andre data fra de danske sogne og kirkelige enheder

Søgning med OpenSearch på Sogn.dk data

<!-- TOC -->
- [Webservices til kirkelige begivenheder og andre data fra de danske sogne og kirkelige enheder](#webservices-til-kirkelige-begivenheder-og-andre-data-fra-de-danske-sogne-og-kirkelige-enheder)
  - [Generelt om disse services](#generelt-om-disse-services)
  - [Indhold tilgængeligt i dette åbne API](#indhold-tilgængeligt-i-dette-åbne-api)
  - [Forskelle fra det gamle ElasticSearch til det nye OpenSearch](#forskelle-fra-det-gamle-elasticsearch-til-det-nye-opensearch)
  - [Typer af data der findes til udtræk:](#typer-af-data-der-findes-til-udtræk)
    - [Begivenheder (`event`)](#begivenheder-event)
      - [Begivenheds felter](#begivenheds-felter)
    - [Enheder (`organiser`)](#enheder-organiser)
      - [Organiser felter](#organiser-felter)
    - [Menighedsråd (`mr`)](#menighedsråd-mr)
      - [Menighedsråd felter](#menighedsråd-felter)
    - [Præster (`priest`)](#præster-priest)
      - [Præst felter](#præst-felter)
  - [Liste over stifter med tilhørende stift ID](#liste-over-stifter-med-tilhørende-stift-id)
  - [Kogebog](#kogebog)
    - [Udtræk af begivenheder med eksempler](#udtræk-af-begivenheder-med-eksempler)
      - [Gudstjenester i et bestemt Sogn, 3 måneder frem.](#gudstjenester-i-et-bestemt-sogn-3-måneder-frem)
      - [Begivenheder i bestemt sogn](#begivenheder-i-bestemt-sogn)
    - [Udtræk af Sogne, kirker og enheder (`organiser`)](#udtræk-af-sogne-kirker-og-enheder-organiser)
      - [Json body til søgning efter Sogn på `sogneid`](#json-body-til-søgning-efter-sogn-på-sogneid)
      - [Json body til søgning efter Provsti på `provstiid`](#json-body-til-søgning-efter-provsti-på-provstiid)
      - [Json body til søgning efter Stift på `StiftID`](#json-body-til-søgning-efter-stift-på-stiftid)
    - [Kirke GET eksempler](#kirke-get-eksempler)
    - [Udtræk af Menighedsråd (`mr`)](#udtræk-af-menighedsråd-mr)
    - [Udtræk af Præster (`priest`)](#udtræk-af-præster-priest)
      - [Eksempel på forespørgsel efter præster for et givent sogn](#eksempel-på-forespørgsel-efter-præster-for-et-givent-sogn)
  - [Debug og test med Postman](#debug-og-test-med-postman)
  - [Kontakt](#kontakt)
<!-- TOC -->

## Generelt om disse services

Disse services er baseret på OpenSearch vers. 2.11.0 Alle data er således gemt i 4 indices, som er
offentligt tilgængeligt for GET requests (men ikke andre typer).  
Data opdateres i nær-realtid fra sogn.dk. Data om begivenheder kommer dels fra sognenes egne indtastninger fra det
administrations interface som sogn.dk tilbyder til oprettelse og redigering af begivenheder (sogn.dk/admin), dels fra de eksterne leverandører som sogne kan have valgt at samarbejde med om deres kalenderløsning, som også synkroniseres med sogn.dk.  
Grunddata om sogne, provstier, stifter, kirker, menighedsråd og præster kommer fra Kirkeministeriets
Informationssystem (KIS), som der synkroniseres med hver nat.  

Denne dokumentation beskriver de data der kan hentes og viser nogle eksempler på hvordan data kan trækkes ud.  
Disse services leverer alle data i UTF-8 format.  
Betegnelsen "begivenhed" bruges i dette dokument som fælles betegnelse for gudstjenester og arrangementer, selvom det ikke er det samme, men har et stort sammenfald af egenskaber.


## Indhold tilgængeligt i dette åbne API

Begivenheder, som er offentligt tilgængelige, kan findes via OpenSearch indicerne `event` `organiser` `mr` og `priest`, fra OpenSearch på denne adresse:

<https://webservice.sogn.dk/>

Du kan læse dokumentationen til OpenSearch på deres hjemmeside:

<https://opensearch.org/docs/2.11/about/>

## Forskelle fra det gamle ElasticSearch til det nye OpenSearch

Generelt er OpenSearch lavet på baggrund af ElasticSearch version 7. Dvs at man kan bruge langt de fleste af de søgeforespørgsler man brugte på det gamle elasticsearch indeks på det nye opensearch indeks.
Væsentlige forskelle er at OpenSearch ikke understøtter typer på samme måde som ElasticSearch har gjort det. Når vi i elasticsearch har lavet en søgning efter events via 

<http://es.sogn.dk/sognekalender/event/_search> 

så skal vi med det nye OpenSearch indeks bruge 

<https://webservice.sogn.dk/event/_search>

I OpenSearch har alle records typen _doc, så dette er default og ikke længere en (nødvendig) del af den søgeurl med bruger.

Man skal også være opmærksom på at response svaret er en smule anderledes ift elasticsearch. Primært at total antal hits på en request nu ikke er tilgængelig via:

result["hits"]["total"], da dette nu er et objekt og findes via result["hits"]["total"]["value"] 
- med mindre man medsender GET parametren: rest_total_hits_as_int (se https://opensearch.org/docs/latest/api-reference/search/)

```json
{"took":0,"timed_out":false,"_shards":{"total":5,"successful":5,"skipped":0,"failed":0},"hits":{"total":{"value":10000,"relation":"gte"},
```

Man skal også være opmærksom på at søgning direkte med GET parametre (uden en request body) kræver at man medsender det content-type svar som man gerne vil have tilbage:
https://opensearch.org/docs/latest/api-reference/common-parameters/#request-body-in-query-string

Altså skal man bruge source som den primære parameter til DSL query og så medtage GET paramtren &source_content_type=application/json


## Typer af data der findes til udtræk:

### Begivenheder (`event`)

https://webservice.sogn.dk/event/

#### Begivenheds felter

|           Feltnavn | Felttype       | Forklaring    |
| :----------------- | -------------- | ------------- |
|               `id` | integer        | Unikt ID for begivenheden. |
|            `title` | text           | Titlen på begivenheden. |
|         `subtitle` | text           | Undertitlen på begivenheden. (eksperimentel)|
|      `description` | text           | En detaljeret beskrivelse af begivenheden. |
| `shortdescription` | text           | En kort beskrivelse af begivenheden. |
|           `person` | text           | Navnet på prædikanten/personen, der er tilknyttet begivenheden. |
|         `category` | keyword        | Kategorien af begivenheden (f.eks. "Gudstjeneste"). |
|     `locationname` | text           | Navnet på stedet, hvor begivenheden finder sted. |
|          `kirkeid` | integer        | Unikt ID for kirker. (id til organiser indekset) |
|       `locationid` | integer        | Unikt ID for lokationen. (id til organiser indekset) |
|         `location` | geo_point      | Geografisk lokation af begivenheden som et geo_point. |
|      `organiserid` | integer        | Unikt ID for arrangøren af begivenheden (id til organiser indekset) |
|    `organisername` | integer        | Navnet på enheden |
|          `sogneid` | integer        | 4 cifret Myndighedskode (fra KIS) |
|        `eventtype` | integer        | Type af begivenhed, se begivenheds typer |
|         `address1` | text           | Adresse linje 1 |
|         `address2` | text           | Adresse linje 2 |
|              `zip` | text           | Postnummer |
|             `city` | text           | Bynavn |
|        `churchday` | text           | Dagens navn i kirkeåret |
|          `created` | datetime       | Tidspunkt hvor begivenheden er oprettet i systemet |
|          `changed` | datetime       | Tidspunkt for seneste ændring af begivenheden |
|          `deleted` | boolean        | Om begivenheden er blevet slettet |
|        `noenddate` | boolean        | Om begivenheden er sat til at skjule sluttidspunktet (uden sluttid) |
|           `origin` | integer        | Oprettelses oprindelse (intern angivelse). |
|          `moreurl` | text           | Link til at læse mere |
|           `online` | boolean        | Om begivenheden afholdes online, eventuelt som supplement |
|             `dawa` | text           | Unikt Dawa ID for adgangsadressen (Endnu ikke i drift) Eksperimentel! Man kan ikke regne med at denne værdi er sat. <br> Adgangsadressens unikke id, f.eks. 0a3f5095-45ec-32b8-e044-0003ba298018 som anvendt af DAWA <br> Se <https://dawadocs.dataforsyningen.dk/dok/api/adgangsadresse> feltet id. <br> Vil nok kun være sat / relevant for lokationer (typen 4,5,6). |
|      `kommunenavn` | text           | Kommune navn |
|        `provstiid` | integer        | Unik provsti ID (fra KIS) |
|          `stiftid` | integer        | Unik stift ID (fra KIS), se listen over stifter  |
|        `startdate` | date           | Startdato og -tid for begivenheden. Format: yyyy-MM-dd HH:mm:ss  i lokaltid (CET/CEST) |
|      `startdateTS` | datetime       | Startdato og -tid for begivenheden. unix timestamp i millisekunder |
|          `enddate` | date           | Slutdato og -tid for begivenheden. Format: yyyy-MM-dd HH:mm:ss  i lokaltid (CET/CEST) |
|        `enddateTS` | datetime       | Slutdato og -tid for begivenheden. unix timestamp i millisekunder |
|          `country` | text           | Landekode, normalt blot DK |
|              `img` | text           | URL til et billede for begivenheden (endnu ikke understøttet) |
|    `opencommunity` | boolean        | Om begivenheden er angivet som et åbent fællesskab (eksperimentel) |

### Enheder (`organiser`)
https://webservice.sogn.dk/organiser/

Sogne, kirker, lokationer og arrangører

#### Organiser felter

|  Feltnavn          | Felttype  | Forklaring|
|:------------------ | --------- | ------------------------------------------ |
| `id`               | interger  | Organiser id. Et unikt id på tværs af de forskellige organiser typer. |
| `type`             | integer   | Typen af organiser: <br> 1: Sogn (KIS), <br> 2: Provsti (KIS), <br> 3: Stift (KIS), <br> 4: Oprettede lokationer fra API leverandørers Sted angivelse <br> 5: Kirke (KIS), <br> 6: Brugeroprettet lokation fra sogn.dk/admin, <br> 7: Brugeroprettede personer |
| `stiftid`          | integer   | Stiftets myndighedskode (fra KIS) |
| `provstiid`        | integer   | Provstiets myndighedskode (fra KIS) |
| `organiserids`     | integer   | De Organiser ID'er som et evt sogn/menighedsråd er tilknyttet (kommasepareret) |
| `sogneid`          | integer   | Sognets myndighedskode (fra KIS). |
| `navn`             | text      | Navnet på organiser/lokation (som kan være af forskellig type). Se type. |
| `kommunenavn`      | text      | Kommunenavn som denne organiser er placeret i |
| `sogndkurl`        | text      | URL som et sogn har på sogn.dk (feks: sogn.dk/gellerup) Er primært relevant for sogne (typen 1). |
| `country`          | text      | Landekode, normalt blot DK |
| `ownurl`           | text      | URL for egen hjemmeside. Dette er primært relevant/sat for sogne (type = 1). |
| `deleted`          | boolean   | Om enheden er blevet slettet |
| `changed`          | datetime  | Unix timestamp i millisekunder. Man kan bruge alle "date" funktioner til at indsnævre en søgning via dette felt. Feltet angiver hvornår en organiser sidst er ændret. |
| `created`          | datetime  | Hvornår denne organiser er oprettet i systemet |
| `aliases`          | text      | Meta felt. Alias for denne organiser (til alternativ stavning ifm søgning). |
| `navnesuggest`     | search_as_you_type | Meta felt. Til søgning (https://opensearch.org/docs/latest/field-types/supported-field-types/search-as-you-type/) |
| `origin`           | integer   | Meta felt som vi bruger til at bestemme hvorfra en organiser er kommet ind til OpenSearch fra. Kan ignoreres. |
| `address1`         | text      | Adresse linje 1 |
| `address2`         | text      | Adresse linje 2 |
| `zip`              | text      | Postnummer |
| `city`             | text      | Bynavn |
| `img`              | text      | Link til billede for enhed, somregel kun sat på kirker |
| `location`         | geo_point | Geo koordinatorne på enheden <br> Det er ikke alle lokationer der har koordinater, men kirker og en del andre har. Lokationer af typen 4 (oprettet via API) har næsten aldrig koordinater. Ej heller sogne, provstier og stifte (typerne 1,2 og 3). |
| `kirkeid`          | integer   | id som stammer fra KIS. Vil kun være sat for kirker. |
| `sogneorganiserid` | text      | Tilknyttet sogn, findes på kirker og brugeroprettede lokationer |
| `dawa`             | text      | Unikt Dawa ID for adgangsadressen (Eksperimentel). Man kan ikke regne med at denne værdi er sat. <br> Adgangsadressens unikke id, f.eks. 0a3f5095-45ec-32b8-e044-0003ba298018 som anvendt af DAWA <br> Se <https://dawadocs.dataforsyningen.dk/dok/api/adgangsadresse> feltet id. <br> Vil nok kun være sat / relevant for lokationer (typen 4,5,6). |

### Menighedsråd (`mr`)
https://webservice.sogn.dk/mr/

Menighedsråd og oplysninger derom

#### Menighedsråd felter

|  Feltnavn      | Felttype | Forklaring |
| :------------- | -------- | ------------------------------- |
| `navn`         | text     | Navn på menighedsrådet |
| `kommunenavn`  | text     | Kommunenavn som menighedsrådet er placeret i |
| `changed`      | datetime | Sidst ændret i unix datetime i millisekunder |
| `mrkode`       | integer  | Myndighedskoden for menighedsrådet |
| `id`           | integer  | ID for menighedsrådet, som regel samme som mrkode |
| `organiserids` | integer  | Organiser ID (kommasepareret hvis der flere) som dette menighedsråd er tilknyttet |
| `navnesuggest` | search_as_you_type | Menighedsråds navn og kommunenavn sat sammen, til søgning |
| `type`         | integer  | Type. 11 for menighedsråd, 8 for Samarbejder (juridiske enheder der har et samarbejde med et menighedsråd - fra KIS) |
| `email`        | text     | Emailadresse på menighedsrådet |
| `sognenames`   | text     | Sognenavn(e) på tilhørende sogne |
| `sogneids`     | integer  | Sognekode(r) tilknyttet dette Menighedsråd |
| `sogndkurls`   | text     | Sogne-webadresser for de sogne som dette menighedsråd omfatter på sogn.dk |
| `deleted`      | bool     | Om det er slettet. |
| `origin`       | integer  | oprindelses id. Internt. |


### Præster (`priest`)
https://webservice.sogn.dk/priest/

#### Præst felter

| Feltnavn                 | Felttype | Forklaring|
| :----------------------- | -------- | ----------------------- |
| `id`                     | integer  | id som stammer fra KIS. |
| `navn`                   | text     | Navn på præsten |
| `navnesuggest`           | search_as_you_type  | Navn til søgning. |
| `sogne`                  | text     | Sognekoder (separaret af mellemrum), som denne præst arbejder i  |
| `provstier`              | datetime | Provstikoder fra KIS (separaret af mellemrum), som denne præst arbejder i |
| `stifte`                 | integer  | Stift(e) fra KIS (separaret af mellemrum), som denne præst arbejder i |
| `mrkodetilknytninger`    | integer  | Menighedsråd (MR koder fra KIS)(separaret af mellemrum) som denne medarbejder arbejder for. |
| `stillingskoder`         | integer  | Stillingskode(r) (separaret af mellemrum) som denne præst er ansat under (stammer fra KIS). |
| `organisertilknytninger` | integer  | id('er) på de organisers (menighedsråd) som denne præst arbejder for. |
| `deleted`                | bool     | Om denne præst er slettet (aktiv eller ej). |
| `origin`                 | integer  | Oprindelses. |
| `created`                | date     | Oprettelsestidspunkt for denne præst i dette system (ikke KIS) |
| `changed`                | date     | Sidst ændret i unix datetime i millisekunder  |


## Liste over stifter med tilhørende stift ID
I tilfælde der skal søges på begivenheder eller enheder der findes i bestemte stifter er herunder en liste.
Denne liste ændres sjældent. 

| Id   |      Stitfs navn       |          Url           |
| :--- | :--------------------: | :--------------------: |
| 1    |    Københavns Stift    | www.kobenhavnsstift.dk |
| 2    |    Helsingør Stift     | www.helsingoerstift.dk |
| 3    |     Roskilde Stift     |  www.roskildestift.dk  |
| 4    | Lolland-Falsters Stift |     www.lfstift.dk     |
| 5    |      Fyens Stift       |   www.fyensstift.dk    |
| 6    |     Aalborg Stift      |  www.aalborgstift.dk   |
| 7    |      Viborg Stift      |   www.viborgstift.dk   |
| 8    |      Århus Stift       |   www.aarhusstift.dk   |
| 9    |       Ribe Stift       |    www.ribestift.dk    |
| 10   |    Haderslev Stift     | www.haderslevstift.dk  |

Den kan også findes ved at filtrere på organiser type = 3: 

Med Lucene Query Syntax: (https://lucene.apache.org/core/2_9_4/queryparsersyntax.html)
https://webservice.sogn.dk/organiser/_search?q=type:3&size=20&pretty 

Eller med DSL Query syntax: (https://opensearch.org/docs/latest/query-dsl/)
```http
GET https://webservice.sogn.dk/organiser/_search?source={%22query%22:{%22bool%22:{%22filter%22:[{%22match%22:{%22type%22:3}},{%22match%22:{%22deleted%22:false}}]}},%22sort%22:[{%22stiftid%22:{%22order%22:%22asc%22}}],%22size%22:20}&pretty&source_content_type=application/json
```
#### Man vil bemærke at man udover landets 10 stifter også får ”Døvemenigheder”, da de også figurerer som et administrativt stift i KIS.

Man har lidt flere muligheder for filtrering og søgning med DSL query syntax, men det er nok også lidt mere komplekst at sætte korrekt op.
Læs mere om brug af DSL query via almindelig GET forespørgsler:
https://opensearch.org/docs/latest/api-reference/common-parameters/#request-body-in-query-string
Og her om søgning generelt i OpenSearch:
https://opensearch.org/docs/latest/api-reference/search/
Hvis man har mulighed for at sende rå data med i sit GET request i det programmeringssprog du bruger så er det lidt nemmere at strikke sammen (det kan man med PHP cUrl og med Postman til tests). 
Vi har efterfølgende brugt denne request body form, da den er mere læsevenlig. Men den request body kan man altså blot sætte ind som GET parameter (som beskrevet i linket ovenfor og her: https://stackoverflow.com/questions/48358344/convert-elasticsearch-kibana-query-string-format-to-uri-search-format).

## Kogebog

### Udtræk af begivenheder med eksempler

#### Gudstjenester i et bestemt Sogn, 3 måneder frem. ####
```http
GET https://webservice.sogn.dk/event/_search
```
```JSON
{
  "query": {
    "bool": {
      "filter": [
        {
          "term": {
            "deleted": {
              "value": false // Fravælg slettede begivenheder  
            }
          }
        },
        {
          "term": {
            "sogneid": {
              "value": 7913 // Find i Erritsø sogn via myndighedskode
            }
          }
        },
        {
          "term": {
            "eventtype": {
              "value": 1 // Der er af typen gudstjenester
            }
          }
        },
        {
          "range": {
            "startdate": {
              "gte": "now", // Findes i perioden fra dags dato
              "lte": "now+3M" // Til 3 måneder frem
            }
          }
        }
      ]
    }
  },
  "sort": [
    {
      "startdate": {
        "order": "asc" // Sortere stigende
      }
    }
  ],
  "size": 2 // Retunere de første 2 resultater
}
```

**Svar**
```json
{
    "took": 32,
    "timed_out": false,
    "_shards": {
        "total": 5,
        "successful": 5,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": {
            "value": 7,
            "relation": "eq"
        },
        "max_score": null,
        "hits": [
            {
                "_index": "event-2024-02-28-1425",
                "_id": "2717329",
                "_score": null,
                "_source": {
                    "id": 2717329,
                    "title": "Palmesøndag Familiegudstjeneste",
                    "subtitle": "",
                    "shortdescription": "",
                    "description": "Kom med til gudstjeneste og få hele påskefortællingen fortalt med fortælling, leg og aktiviteter. Gudstjenesten er for hele familien, og samtidigt er det afslutning for minikonfirmanderne. Efter gudstjenesten spiser vi sammen i Sognehuset, og minikonfirmander får overrakt diplomer. Tag gerne hele familien med til festlig familiegudstjeneste og frokost den dag. Maden koster 25 kr., gratis for børn.",
                    "person": "Tina Iversen",
                    "locationname": "Erritsø Kirke",
                    "category": "Familiegudstjeneste",
                    "kirkeid": 7198,
                    "locationid": 42394,
                    "location": "55.547798,9.700200",
                    "organiserid": 40177,
                    "organisername": "Erritsø Sogn",
                    "sogneid": 7913,
                    "eventtype": 1,
                    "address1": "Erritsø Bygade 1",
                    "address2": "Erritsø",
                    "zip": "7000",
                    "city": "Fredericia",
                    "churchday": "Palmesøndag",
                    "created": 1700138021000,
                    "changed": 1703682372000,
                    "deleted": false,
                    "noenddate": false,
                    "origin": 11,
                    "moreurl": "",
                    "img": "",
                    "online": false,
                    "dawa": "",
                    "kommunenavn": "Fredericia Kommune",
                    "provstiid": 1004,
                    "stiftid": 10,
                    "startdate": "2024-03-24 10:00:00",
                    "startdateTS": 1711270800000,
                    "enddate": "2024-03-24 11:00:00",
                    "enddateTS": 1711274400000
                },
                "sort": [
                    1711274400000
                ]
            },
            {
                "_index": "event-2024-02-28-1425",
                "_id": "2717341",
                "_score": null,
                "_source": {
                    "id": 2717341,
                    "title": "Skærtorsdag Gudstjeneste",
                    "subtitle": "",
                    "shortdescription": "",
                    "description": "",
                    "person": "Peter Nikolaj Frøkjær-Jensen",
                    "locationname": "Erritsø Kirke",
                    "category": "Gudstjeneste",
                    "kirkeid": 7198,
                    "locationid": 42394,
                    "location": "55.547798,9.700200",
                    "organiserid": 40177,
                    "organisername": "Erritsø Sogn",
                    "sogneid": 7913,
                    "eventtype": 1,
                    "address1": "Erritsø Bygade 1",
                    "address2": "Erritsø",
                    "zip": "7000",
                    "city": "Fredericia",
                    "churchday": "Skærtorsdag",
                    "created": 1700138212000,
                    "changed": 1700139404000,
                    "deleted": false,
                    "noenddate": false,
                    "origin": 11,
                    "moreurl": "",
                    "img": "",
                    "online": false,
                    "dawa": "",
                    "kommunenavn": "Fredericia Kommune",
                    "provstiid": 1004,
                    "stiftid": 10,
                    "startdate": "2024-03-28 17:00:00",
                    "startdateTS": 1711641600000,
                    "enddate": "2024-03-28 18:00:00",
                    "enddateTS": 1711645200000
                },
                "sort": [
                    1711645200000
                ]
            }
        ]
    }
}
```


Eksemplet vil i PHP med curl se nogenlunde således ud:

```PHP
$params = array(
    "query"  => array(
        "bool" => array(
            "filter" => array(
                array(
    "term" => array(
        "sogneid" => 7913
    )
                ),
                array(
    "term" => array(
        "eventtype" => 1
    )
                ),
                array(
    "term" => array(
        "deleted" => false
    )
                ),
                array(
    "range" => array(
        "startdate" => array(
   "gte" => "now",
   "lte" => "now+3M"
        )
    )
                )
            )
        )
    ),
    "sort" => array(
        "startdate" => "asc"
    ),
    "size" => 100
);
$jsonParams = json_encode($params);

$ch = curl_init();
curl_setopt($ch, CURLOPT_URL, "https://webservice.sogn.dk/event/_search");
curl_setopt($ch, CURLOPT_CUSTOMREQUEST, 'GET');
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
curl_setopt($ch, CURLOPT_HTTPHEADER, array('Content-Type: application/json','Content-Length: ' . strlen($jsonParams)));
curl_setopt($ch, CURLOPT_POSTFIELDS, $jsonParams);
$result = curl_exec($ch);
$resultArray = array();
if (curl_errno($ch) === 0) {
    $resultArray = json_decode($result,true);
} else {
    error_log('Searcher: '.curl_error($ch).$result);
}
curl_close($ch);
return $resultArray;
```
Brug af OpenSearch via javascript følger samme principper.  
Bemærk at man nemt kan medtage en size og en from parameter til at opdele resultater i mindre mængder (se OpenSearch dokumentationen).


#### Begivenheder i bestemt sogn
[Se alternativ til URL encoding](https://stackoverflow.com/questions/14339696/elasticsearch-post-with-json-search-body-vs-get-with-json-in-url)  

```curl
curl -XGET "https://webservice.sogn.dk/event/_search" -d '{
  "query": {
    "bool": {
      "filter": [
        { "term":  { "deleted": false }},
        { "term":  { "sogneid": 7000 }},
        { "term":  { "eventtype": 1}},
        { "range": { "startdate": { "gte": "now" }}}
      ]
    }
  },
  "sort": [
    { "startdate": "ASC"}
  ],
  "size": 1
}'
```

Resultatet af ovenstående forespørgsel kan se således ud:

```json
{
    "took": 35,
    "timed_out": false,
    "_shards": {
        "total": 5,
        "successful": 5,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": {
            "value": 10000,
            "relation": "gte"
        },
        "max_score": null,
        "hits": [
            {
                "_index": "event-2024-02-28-1425",
                "_id": "2500559",
                "_score": null,
                "_source": {
    "id": 2500559,
    "title": "Gudstjeneste",
    "subtitle": "",
    "shortdescription": "",
    "description": "Gudstjeneste i Fole Kirke",
    "person": "Inger Petersen",
    "locationname": "Fole Kirke",
    "category": "Gudstjeneste",
    "kirkeid": 7111,
    "locationid": 42313,
    "location": "55.315300,8.981200",
    "organiserid": 40099,
    "organisername": "Fole Sogn",
    "sogneid": 8952,
    "eventtype": 1,
    "address1": "Folevej 33",
    "address2": "Fole",
    "zip": "6510",
    "city": "Gram",
    "churchday": "Nytårsdag",
    "created": 1668942902000,
    "changed": 1668942920000,
    "deleted": false,
    "noenddate": false,
    "origin": 11,
    "moreurl": "",
    "img": "",
    "online": true,
    "dawa": "",
    "kommunenavn": "Haderslev Kommune",
    "provstiid": 1001,
    "stiftid": 10,
    "startdate": "2023-01-01 00:00:00",
    "startdateTS": 1672527600000,
    "enddate": "2023-01-01 01:00:00",
    "enddateTS": 1672531200000
                },
                "sort": [
    1672531200000
                ]
            }
        ]
    }
}
```


### Udtræk af Sogne, kirker og enheder (`organiser`)

Exsempel på søgning efter et sogn ud fra et navn.  
[Læs her om Opensearch Query String Syntax](https://opensearch.org/docs/latest/query-dsl/full-text/query-string)

```curl
curl -XGET "https://webservice.sogn.dk/organiser/_search" -d '{
  "query": {
    "bool": {
      "must": [
        { "match": { "navn": "Gellerup" }}
      ]
    }
  }
}'
```

Kan også laves som en direkte query string request (med Lucene Query):
https://webservice.sogn.dk/organiser/_search?q=navn:Gellerup&pretty

Resultatet for begge ovenstående forespørgsler er:

```json
{
    "took": 1,
    "timed_out": false,
    "_shards": {
        "total": 5,
        "successful": 5,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": {
            "value": 2,
            "relation": "eq"
        },
        "max_score": 6.677451,
        "hits": [
            {
                "_index": "organiser-2024-01-17-1325",
                "_id": "39893",
                "_score": 6.677451,
                "_source": {
                    "id": 39893,
                    "type": 1,
                    "stiftid": 8,
                    "provstiid": 817,
                    "organiserid": 39893,
                    "sogneid": 9097,
                    "navn": "Gellerup Sogn",
                    "kommunenavn": "Aarhus Kommune",
                    "sogndkurl": "https://sogn.dk/gellerup",
                    "country": "DK",
                    "ownurl": "www.gellerupkirke.dk",
                    "deleted": false,
                    "changed": 1556632009000,
                    "created": 1387396021000,
                    "aliases": "fisk",
                    "navnesuggest": "Gellerup Sogn fisk",
                    "origin": 1
                }
            },
            {
                "_index": "organiser-2024-01-17-1325",
                "_id": "42507",
                "_score": 6.600841,
                "_source": {
                    "id": 42507,
                    "type": 5,
                    "stiftid": 8,
                    "provstiid": 817,
                    "organiserid": 42507,
                    "sogneid": 9097,
                    "navn": "Gellerup Kirke",
                    "kommunenavn": "Aarhus Kommune",
                    "sogndkurl": "https://sogn.dk/gellerup",
                    "country": "DK",
                    "ownurl": "",
                    "deleted": false,
                    "changed": 1631837101000,
                    "created": 1387400461000,
                    "aliases": "",
                    "navnesuggest": "Gellerup Kirke",
                    "origin": 1,
                    "address1": "Gudrunsvej 88",
                    "address2": "",
                    "zip": "8220",
                    "city": "Brabrand",
                    "img": "https://sogn.dk/uploads/DyconSogneadmin/Organiser/uid42507-ImageCropped-631c91.jpg",
                    "location": "56.155773,10.133944",
                    "kirkeid": 7314,
                    "sogneorganiserid": 39893
                }
            }
        ]
    }
}
```

Man får altså 2 hits. Et hit for sognet (type: 1), et hit for kirken i Gellerup (type: 5).  
Hvis man således kun ønsker at søge efter sogne kan man tilføje type: 1 til søgningen:

```curl
curl -XGET "https://webservice.sogn.dk/organiser/_search" -d '{
  "query": {
    "bool": {
      "must": [
        { "match": { "navn": "Gellerup" }},  
        { "match": { "type": 1 }}  
      ]
    }
  }
}'
```

Kan også laves som en direkte query string request med cUrl (eller direkte i browseren):  
```curl 
curl -XGET "https://webservice.sogn.dk/organiser/_search?source=%7B%0A%20%20%22query%22%3A%20%7B%20%0A%20%20%20%20%22bool%22%3A%20%7B%20%0A%20%20%20%20%20%20%22must%22%3A%20%5B%0A%20%20%20%20%20%20%20%20%7B%20%22match%22%3A%20%7B%20%22navn%22%3A%20%22Gellerup*%22%20%7D%7D%2C%20%20%0A%20%20%20%20%20%20%20%20%7B%20%22match%22%3A%20%7B%20%22type%22%3A%201%20%7D%7D%20%20%0A%20%20%20%20%20%20%5D%0A%20%20%20%20%7D%0A%20%20%7D%0A%7D%0A&pretty&source_content_type=application/json" 
```
*Bemærk at guery strengen er url encodet direkte ud fra request body (men med source som parameter nu)*

Eller med Lucene syntax:
```http 
GET https://webservice.sogn.dk/organiser/_search?q=navn:Gellerup%20AND%20type:1&pretty
```
*Bemærk at query strengen i parameteren `q` skal url encodes (hvis ikke din browser gør det for dig). Her bliver mellemrum konverteret til %20*

Resultatet er nu at man kun får Gellerup Sogn i svaret.
Hvis man kun vil finde kirker, kan man så i stedet sætte type til 5.  

#### Json body til søgning efter Sogn på `sogneid`

```json
{
  "query": {
    "bool": {
      "filter": [
        {
          "term": {
            "sogneid": 7044
          }
        },
        {
          "term": {
            "type": 1
          }
        }
      ]
    }
  },
  "size": 1
}
```

#### Json body til søgning efter Provsti på `provstiid`

```json
{
  "query": {
    "bool": {
      "filter": [
        {
          "term": {
            "provstiid": 115
          }
        },
        {
          "term": {
            "type": 2
          }
        }
      ]
    }
  },
  "size": 1
}
```

#### Json body til søgning efter Stift på `StiftID`

```json
{
  "query": {
    "bool": {
      "filter": [
        {
          "term": {
            "stiftid": 1
          }
        },
        {
          "term": {
            "type": 3
          }
        }
      ]
    }
  },
  "size": 1
}
```

Resultat på Stift GET

```json
{
    "took": 1,
    "timed_out": false,
    "_shards": {
        "total": 5,
        "successful": 5,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": {
            "value": 1,
            "relation": "eq"
        },
        "max_score": 0.0,
        "hits": [
            {
                "_index": "organiser-2024-01-17-1325",
                "_id": "54165",
                "_score": 0.0,
                "_source": {
                    "id": 54165,
                    "type": 3,
                    "stiftid": 1,
                    "provstiid": 0,
                    "organiserid": 54165,
                    "sogneid": 0,
                    "navn": "Københavns Stift",
                    "kommunenavn": "",
                    "sogndkurl": "",
                    "country": "DK",
                    "ownurl": "www.kobenhavnsstift.dk",
                    "deleted": false,
                    "changed": 1556632009000,
                    "created": 1401787646000,
                    "aliases": "",
                    "navnesuggest": "Københavns Stift",
                    "origin": 1
                }
            }
        ]
    }
}
```


Ønsker man at begrænse resultatet til alle provstier eller stifte kan man således sætte et filter på typen (med 2 for
provstier og 3 for stifte).

### Kirke GET eksempler

```http request
GET https://webservice.sogn.dk/organiser/_search
```
Json Body
```json
{
  "query": {
    "bool": {
      "filter": [
        {
          "term": { "type": 5 }
        },
        {
          "term": { "deleted": false }
        },
        {
          "term": { "zip": "8000" }
        }
      ]
    }
  },
  "sort": [
    {
      "sogneid": "desc"
    }
  ],
  "size": 2
}
```
Svar
```json
{
    "took": 1,
    "timed_out": false,
    "_shards": {
        "total": 5,
        "successful": 5,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": {
            "value": 9,
            "relation": "eq"
        },
        "max_score": null,
        "hits": [
            {
                "_index": "organiser-2024-01-17-1325",
                "_id": "41847",
                "_score": null,
                "_source": {
                    "id": 41847,
                    "type": 5,
                    "stiftid": 8,
                    "provstiid": 801,
                    "organiserid": 41847,
                    "sogneid": 8132,
                    "navn": "Sankt Lukas Kirke",
                    "kommunenavn": "Aarhus Kommune",
                    "sogndkurl": "https://sogn.dk/sanktlukas-aarhus",
                    "country": "DK",
                    "ownurl": "",
                    "deleted": false,
                    "changed": 1631837101000,
                    "created": 1387400461000,
                    "aliases": "",
                    "navnesuggest": "Sankt Lukas Kirke",
                    "origin": 1,
                    "address1": "Skt.Lukas Kirkeplads 1",
                    "address2": "",
                    "zip": "8000",
                    "city": "Aarhus C",
                    "img": "https://sogn.dk/uploads/DyconSogneadmin/Organiser/8132_6603_1383140270.jpg",
                    "location": "56.144753,10.193872",
                    "kirkeid": 6603,
                    "sogneorganiserid": 39575
                },
                "sort": [
                    8132
                ]
            },
            {
                "_index": "organiser-2024-01-17-1325",
                "_id": "41843",
                "_score": null,
                "_source": {
                    "id": 41843,
                    "type": 5,
                    "stiftid": 8,
                    "provstiid": 801,
                    "organiserid": 41843,
                    "sogneid": 8120,
                    "navn": "Sankt Johannes Kirke",
                    "kommunenavn": "Aarhus Kommune",
                    "sogndkurl": "https://sogn.dk/sanktjohannes-aarhus",
                    "country": "DK",
                    "ownurl": "",
                    "deleted": false,
                    "changed": 1631837101000,
                    "created": 1387400461000,
                    "aliases": "",
                    "navnesuggest": "Sankt Johannes Kirke",
                    "origin": 1,
                    "address1": "Peter Sabroes Gade 20",
                    "address2": "",
                    "zip": "8000",
                    "city": "Aarhus C",
                    "img": "https://sogn.dk/uploads/DyconSogneadmin/Organiser/8120_6599_1393320149.jpg",
                    "location": "56.169144,10.210092",
                    "kirkeid": 6599,
                    "sogneorganiserid": 39573
                },
                "sort": [
                    8120
                ]
            }
        ]
    }
}
```

### Udtræk af Menighedsråd (`mr`)

`mr` typen viser udvalgt data om alle landets menighedsråd (små 2000). Du kan se hvilke felter der findes ved at kalde _mapping:
`https://webservice.sogn.dk/mr/_mapping`

En søgning uden filtre eller parametre vil give et resultat på alle landets menighedsråd (dog begrænset til de første 10
da det er default når size parameteret udelades).  
`https://webservice.sogn.dk/mr/_search`  

Resultat herunder vises blot det første

```json
{
  "took": 0,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 1644,
      "relation": "eq"
    },
    "max_score": 1.0,
    "hits": [
      {
        "_index": "mr-2024-01-17-1325",
        "_id": "7039",
        "_score": 1.0,
        "_source": {
          "navn": "Sankt Pauls Sogns Menighedsråd",
          "kommunenavn": "Københavns Kommune",
          "changed": 1682041235000,
          "mrkode": 7039,
          "id": 7039,
          "organiserids": "38162",
          "navnesuggest": "Sankt Pauls Sogns Menighedsråd Københavns Kommune",
          "type": 11,
          "email": "7039@SOGN.DK",
          "sognenames": "Sankt Pauls Sogn",
          "sogneids": "7039",
          "sogndkurls": "https://sogn.dk/sanktpaul-koebenhavn/menighedsraad/",
          "country": "DK",
          "deleted": false,
          "origin": 1
        }
      }
    ]
  }
}
```

Igen er det vigtigt altid at huske at medtage et `deleted:false` filter, da man ellers får slettede menighedsråd med.

### Udtræk af Præster (`priest`)

Man kan søge efter landets ansatte præster ved at søge i typen priest. De forskellige felter kan ses ved at kalde
mapping:

`https://webservice.sogn.dk/priest/_mapping`


Et svar til `_search` (uden parametre kan give et resultat som nedenfor):

```json
{
  "took": 0,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 2222,
      "relation": "eq"
    },
    "max_score": 1.0,
    "hits": [
      {
        "_index": "priest-2024-01-17-1325",
        "_id": "35",
        "_score": 1.0,
        "_source": {
          "id": 35,
          "navn": "Henriette Petersen",
          "navnesuggest": "Henriette Petersen",
          "sogne": "7697 7698",
          "provstier": "508",
          "stifte": "5",
          "mrkodetilknytninger": "7697 7698",
          "stillingskoder": "9255",
          "organisertilknytninger": "38882 38883",
          "deleted": false,
          "origin": 1,
          "created": 1705494301000,
          "changed": 1705494301000
        }
      },
      {
        "_index": "priest-2024-01-17-1325",
        "_id": "17",
        "_score": 1.0,
        "_source": {
          "id": 17,
          "navn": "Curt Petersen",
          "navnesuggest": "Curt Petersen",
          "sogne": "7220 7236 7237",
          "provstier": "304",
          "stifte": "3",
          "mrkodetilknytninger": "7107",
          "stillingskoder": "9255",
          "organisertilknytninger": "38078",
          "deleted": false,
          "origin": 1,
          "created": 1705494301000,
          "changed": 1705494301000
        }
      },
      {
        "_index": "priest-2024-01-17-1325",
        "_id": "357",
        "_score": 1.0,
        "_source": {
          "id": 357,
          "navn": "Peter Peddersen",
          "navnesuggest": "Peter Peddersen",
          "sogne": "7107",
          "provstier": "106",
          "stifte": "1",
          "mrkodetilknytninger": "7220 7236 7237",
          "stillingskoder": "9255",
          "organisertilknytninger": "38372 38383 38384",
          "deleted": false,
          "origin": 1,
          "created": 1705494301000,
          "changed": 1705494301000
        }
      }
    ]
  }
}
```

Bemærk specielt den sidste præst med "id": 357, som har tre ”sogne”, to ”mrkodetilknytninger” og tre
”organisertilknytninger”. Det vil sige, at denne præst har flere tilknytninger (præst for tre sogne).  
En præst kan have mange tilknytninger, så denne liste kunne bestå af væsentligt flere sogne, mm.  
Bemærk at værdierne er adskilt af mellemrum.  
For de leverandører der indleverer kalender data på vegne af et sogn til sogn.dk,- er det specielt interessant at forstå
hvordan præstens unikke id for en sognerelation opbygges.  
Generelt består en præsts id (som det feks bruges som attributten id for feltet praest i DTD’en for indlevering af
begivenheder <https://sogn.dk/api/dtd/api.dtd>) af:

`<id>` + `<sognenr>` + `<stillingskode>` (hvor + blot er en konkatenering)

Så I tilfælde af flere sognetilknytninger, skal man bruge det relevante sogns nummer. For præsten med id 357 vil denne
have tre præste id’er som kunne bruges til at angive denne præst som præst for en begivenhed for hhv. sogn med nr 7220 
og 7236. Nemlig 35772209255 og 35772369255.  
Såfremt der er flere stillingskoder vil de relevante koder for præster være en af de følgende: 
(rangeret med hyppigst forekommende øverst)

9255 Sognepræst (kirkebogsfører)  
9254 Sognepræst  
9023 Sognepræst og korshærspræst  
9264 Provst (kirkebogsfører)  
9655 Hospitalpræst

#### Eksempel på forespørgsel efter præster for et givent sogn

`https://webservice.sogn.dk/priest/_search`


```json
{
  "query": {
    "bool": {
      "must": [],
      "filter": [
        {
          "term": {
            "deleted": false
          }
        },
        {
          "term": {
            "sogne": "7201"
          }
        }
      ]
    }
  }
```

Resultatet af ovenstående forespørgsel vil være en liste med præster for sognet med nr. 7201:

```json
{
    "took": 726,
    "timed_out": false,
    "_shards": {
        "total": 5,
        "successful": 5,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": {
            "value": 3,
            "relation": "eq"
        },
        "max_score": 0.0,
        "hits": [
            {
                "_index": "priest-2023-12-20-1636",
                "_id": "387",
                "_score": 0.0,
                "_source": {
                    "id": 387,
                    "navn": "Jim Petersen",
                    "navnesuggest": "Christina Petersen",
                    "sogne": "7200 7201",
                    "provstier": "313",
                    "stifte": "3",
                    "mrkodetilknytninger": "7200 7201",
                    "stillingskoder": "9255",
                    "organisertilknytninger": "38636 38637",
                    "deleted": false,
                    "origin": 1,
                    "created": 1703090194000,
                    "changed": 1703090194000
                }
            },
            {
                "_index": "priest-2023-12-20-1636",
                "_id": "270457",
                "_score": 0.0,
                "_source": {
                    "id": 270457,
                    "navn": "Jane Petersen",
                    "navnesuggest": "Maria Petersen",
                    "sogne": "7200 7201",
                    "provstier": "313",
                    "stifte": "3",
                    "mrkodetilknytninger": "7200 7201",
                    "stillingskoder": "9023",
                    "organisertilknytninger": "38636 38637",
                    "deleted": false,
                    "origin": 1,
                    "created": 1703090194000,
                    "changed": 1703090194000
                }
            },
            {
                "_index": "priest-2023-12-20-1636",
                "_id": "423",
                "_score": 0.0,
                "_source": {
                    "id": 423,
                    "navn": "Joe Petersen",
                    "navnesuggest": "Joe Petersen",
                    "sogne": "7200 7201",
                    "provstier": "313",
                    "stifte": "3",
                    "mrkodetilknytninger": "7200 7201",
                    "stillingskoder": "9023",
                    "organisertilknytninger": "38636 38637",
                    "deleted": false,
                    "origin": 1,
                    "created": 1703090194000,
                    "changed": 1703090194000
                }
            }
        ]
    }
}
```

Man kunne lave en lignende forespørgsel hvor man henter en liste med alle præster for et givent provsti eller et helt
stift, ved angivelse af hhv ”provstier” eller ”stifte” som ekstra filter parameter.

## Debug og test med Postman

Man kan selvfølgelig teste adgang og forespørgsler via programmering og brugen af eksempelvis cUrl i PHP. En lidt
simplere måde at teste på er ved at bruge et værktøj som eksemplevis Postman:

<https://www.postman.com/downloads/>

Her er et eksempel på en forespørgsel foretaget med Postman, for at finde alle præster i et givent sogn:
![image](https://user-images.githubusercontent.com/26836939/143202455-c0926821-c4a8-4743-a7c8-e94c456e3a7f.png)

Bemærk at det – på trods af at være et GET request – godt kan lade sig gøre at medsende en ”raw” body (det har man ikke
kunne i tidligere versioner af Postman, men det kan man godt i nyere versioner).
Resultatet findes under Headers efter at man har Sendt forespørgslen:
![image](https://user-images.githubusercontent.com/26836939/143202577-b984f034-1535-47fc-9117-c6e4c1ad508b.png)

På denne måde kan man nemt finde frem til passende søge-kriterier at medsende til ElasticSearch.
Der er umiddelbart ingen restriktioner på hvor mange requests eller hvor meget data man kan hente hvor ofte,- men vi
opfordrer til at man ikke overbelaster vores services unødigt,- da vi så kan blive nødt til at indføre sådanne
restriktioner.

## Kontakt

Folkekirkens It  
By-, Land- og Kirkeministeriet  
E-post: it-kontoret@km.dk  
Telefon: 70 20 25 35  
Telefontid: mandag-torsdag kl. 8.30-16.00 - fredag kl. 9.00-14.00  
https://folkekirkensit.dk
