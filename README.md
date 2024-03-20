# Webservices til kirkelige begivenheder og andre data fra de danske sogne og kirkelige enheder

Søgning med OpenSearch på Sogn.dk data

<!-- TOC -->
- [Webservices til kirkelige begivenheder og andre data fra de danske sogne og kirkelige enheder](#webservices-til-kirkelige-begivenheder-og-andre-data-fra-de-danske-sogne-og-kirkelige-enheder)
  - [Generelt om disse services](#generelt-om-disse-services)
  - [Indhold tilgængeligt i dette åbne API](#indhold-tilgængeligt-i-dette-åbne-api)
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

Disse services er baseret på OpenSearch vers. 2.11.0 Alle data er således gemt i 4 indekse, som er
offentligt tilgængeligt for GET requests.  
Data opdateres i nær-realtid fra sogn.dk. Data om begivenheder kommer dels fra sognenes egne indtastninger fra det
administrations interface som sogn.dk tilbyder til oprettelse og redigering af begivenheder (sogn.dk/admin), dels fra de eksterne leverandører som sogne kan have valgt at samarbejde med om deres kalenderløsning, som også synkroniseres med sogn.dk.  
Grunddata om sogne, provstier, stifter, kirker, menighedsråd og præster kommer fra Kirkeministeriets
Informationssystem (KIS), som der synkroniseres med hver nat.  

Denne dokumentation beskriver de data der kan hentes og viser nogle eksempler på hvordan data kan trækkes ud.  
Disse services leverer alle data i UTF-8 format.  
Betegnelsen "begivenhed" bruges i dette dokument som fælles betegnelse for gudstjenester og arrangementer, selvom det ikke er det samme, men har et stort sammenfald af egenskaber.


## Indhold tilgængeligt i dette åbne API

Begivenheder, som er offentligt tilgængelige, kan findes via OpenSearch indeksene `event` `organiser` `mr` og `priest`, fra OpenSearch på denne adresse:

<https://webservice.sogn.dk/>

Du kan læse dokumentationen til OpenSearch på deres hjemmeside:

<https://opensearch.org/docs/2.11/about/>

## Typer af data der findes til udtræk:

### Begivenheder (`event`)

https://webservice.sogn.dk/event/

#### Begivenheds felter

|        Feltnavn | Felttype       | Forklaring    |
| :-------------- | -------------- | ------------- |
|            `id` | integer        | Unikt ID for begivenheden. |
|         `title` | text           | Titlen på begivenheden. |
|   `description` | text           | En detaljeret beskrivelse af begivenheden. |
|        `person` | text           | Navnet på prædikanten/personen, der er tilknyttet begivenheden. |
|      `category` | keyword        | Kategorien af begivenheden (f.eks. "Gudstjeneste"). |
|  `locationname` | text           | Navnet på stedet, hvor begivenheden finder sted. |
|       `kirkeid` | integer        | Unikt ID for kirker |
|    `locationid` | integer        | Unikt ID for lokationen |
|      `location` | geo_point      | Geografisk lokation af begivenheden som et geo_point. |
|   `organiserid` | integer        | Unikt ID for enheder |
| `organisername` | integer        | Navnet på enheden |
|       `sogneid` | integer        | 4 cifret Myndighedskode |
|     `eventtype` | integer        | Type af begivenhed, se begivenheds typer |
|      `address1` | text           | Adresse linje 1 |
|      `address2` | text           | Adresse linje 2 |
|           `zip` | text           | Postnummer |
|          `city` | text           | Bynavn |
|     `churchday` | text           | Søndagens navn |
|       `created` | datetime       | Begivenhed oprettet i systemet |
|       `changed` | datetime       | Begivenheden sidst ændret |
|       `deleted` | boolean        | Om begivenheden er blevet slettet |
|     `noenddate` | boolean        | Om begivenheden er sat til at skjule sluttidspunktet |
|        `origin` | integer        | Oprettelses oprindelse |
|       `moreurl` | text           | Link til at læse mere |
|        `online` | boolean        | Om begivenheden afholdes online, eventuelt som supplement |
|          `dawa` | text           | Unikt Dawa ID for adgangsadressen (Endnu ikke i drift) Eksperimentel! Man kan ikke regne med at denne værdi er sat. <br> Adgangsadressens unikke id, f.eks. 0a3f5095-45ec-32b8-e044-0003ba298018 som anvendt af DAWA <br> Se <https://dawadocs.dataforsyningen.dk/dok/api/adgangsadresse> feltet id. <br> Vil nok kun være sat / relevant for lokationer (typen 4,5,6). |
|   `kommunenavn` | text           | Provsti navn |
|     `provstiid` | integer        | Unik provsti ID |
|       `stiftid` | integer        | Unik stift ID, se listen over stifter  |
|     `startdate` | date           | Startdato og -tid for begivenheden. Format: yyyy-MM-dd HH:mm:ss |
|   `startdateTS` | datetime       | Startdato og -tid for begivenheden |
|       `enddate` | date           | Slutdato og -tid for begivenheden. Format: yyyy-MM-dd HH:mm:ss  |
|     `enddateTS` | datetime       | Slutdato og -tid for begivenheden. |


### Enheder (`organiser`)
https://webservice.sogn.dk/organiser/

Sogne, kirker og lokationer

#### Organiser felter

|  Feltnavn          | Felttype  | Forklaring|
|:------------------ | --------- | ------------------------------------------ |
| `Id`               | interger  | Organiser id. Er så et unikt id på tværs af de forskellige organiser typer. |
| `type`             | integer   | Typen af organiser: <br> 1: Sogn (KIS), <br> 2: Provsti (KIS), <br> 3: Stift (KIS), <br> 4: Oprettede lokationer fra API leverandørers Sted angivelse <br> 5: Kirke (KIS), <br> 6: Brugeroprettet lokation fra sogn.dk/admin |
| `stiftid`          | integer   | Stiftets myndighedskode |
| `provstiid`        | integer   | Provstiets myndighedskode |
| `organiserids`     | integer   | Organiser ID menighedsrådet er tilknyttet |
| `sogneid`          | integer   | Sognets myndighedskode. Er også sat for lokationer oprettet af sognebrugere, hvor det så er sognets id der står angivet. |
| `navn`             | text      | Navnet på organiser/lokation (som jo kan være af forskellig type). Se type. |
| `kommunenavn`      | text      | Kommunenavn som menighedsrådet er placeret i |
| `sogndkurl`        | text      | URL som et sogn har på sogn.dk (feks: sogn.dk/gellerup) Er primært relevant for sogne (typen 1). |
| `country`          | text      | Landekode, normalt blot DK |
| `ownurl`           | text      | URL for egen hjemmeside. Dette er primært relevant/sat for sogne. |
| `deleted`          | boolean   | Om begivenheden er blevet slettet |
| `changed`          | datetime  | Unix timestampt, Bemærk at denne date vises som antal millisekunder og ikke sekunder. Man kan bruge alle "date" funktioner til at indsnævre en søgning via dette felt. | Dette date felt angiver hvornår en organiser sidst er ændret (eller oprettet). |
| `created`          | datetime  | Begivenhed oprettet i systemet |
| `aliases`          | text      | Meta felt |
| `navnesuggest`     | text      | Meta felt |
| `origin`           | integer   | Meta felt som vi bruger til at bestemme hvorfra en organiser er kommet ind til OpenSearch fra. Kan ignoreres. |
| `address1`         | text      | Adresse linje 1 |
| `address2`         | text      | Adresse linje 2 |
| `zip`              | text      | Postnummer |
| `city`             | text      | Bynavn |
| `img`              | text      | Link til billede for enhed, somregel kun sat på kirker |
| `location`         | geo_point | Geo koordinatorne på enheden <br> Det er ikke alle lokationer der har koordinater, men kirker og en del andre har. Lokationer af typen 4 (oprettet via API) har næsten aldrig koordinater. Ej heller sogne, provstier og stifte (typerne 1,2 og 3). |
| `kirkeid`          | integer   | id som stammer fra KIS. Vil kun være sat for kirker. |
| `sogneorganiserid` | text      | Tilknyttet sogn, findes på kirker og brugeroprettede lokationer |

### Menighedsråd (`mr`)
https://webservice.sogn.dk/mr/

Menighedsråd og oplysninger derom

#### Menighedsråd felter

|  Feltnavn      | Felttype | Forklaring |
| :------------- | -------- | ------------------------------- |
| `navn`         | text     | Navn på menighedsrådet |
| `kommunenavn`  | text     | Kommunenavn som menighedsrådet er placeret i |
| `changed`      | datetime | Sidst ændret i unix datetime |
| `mrkode`       | integer  | Myndighedskoden for menighedsrådet |
| `id`           | integer  | ID for menighedsrådet, som regel samme som mrkode |
| `organiserids` | integer  | Organiser ID menighedsrådet er tilknyttet |
| `navnesuggest` | text     | Menighedsråds navn og kommunenavn sat sammen, til søgning |
| `type`         | integer  | Type af Kirke |
| `email`        | text     | Emailadresse på menighedsrådet |
| `sognenames`   | text     | Tilhørende sogns navn |
| `sogneids`     | integer  | Myndighedskoden for sognet |
| `sogndkurls`   | text     | Link til menighedsrådet på Sogn.dk |
| `country`      | text     | id som stammer fra KIS. |
| `deleted`      | bool     | id som stammer fra KIS. |
| `origin`       | integer  | id som stammer fra KIS. |


### Præster (`priest`)
https://webservice.sogn.dk/priest/

#### Præst felter

| Feltnavn                 | Felttype | Forklaring|
| :----------------------- | -------- | ----------------------- |
| `id`                     | integer  | id som stammer fra KIS. |
| `navn`                   | text     | Navn på menighedsrådet |
| `navnesuggest`           | integer  | id som stammer fra KIS. |
| `sogne`                  | text     | Kommunenavn som menighedsrådet er placeret i |
| `provstier`              | datetime | Sidst ændret i unix datetime |
| `stifte`                 | integer  | Myndighedskoden for menighedsrådet |
| `mrkodetilknytninger`    | integer  | id som stammer fra KIS. |
| `stillingskoder`         | integer  | id som stammer fra KIS. |
| `organisertilknytninger` | integer  | id som stammer fra KIS. |
| `deleted`                | bool     | id som stammer fra KIS. |
| `origin`                 | integer  | id som stammer fra KIS. |
| `created`                | integer  | id som stammer fra KIS. |
| `changed`                | integer  | id som stammer fra KIS. |


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
          "bool": {
            "should": [
              {
                "term": {
  "deleted": {
    "value": false // Fravælg slettede begivenheder  
  }
                }
              }
            ]
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
    "took": 50,
    "timed_out": false,
    "_shards": {
        "total": 5,
        "successful": 5,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": {
            "value": 8,
            "relation": "eq"
        },
        "max_score": null,
        "hits": [
            {
                "_index": "event-2024-02-28-1425",
                "_id": "2588909",
                "_score": null,
                "_source": {
    "id": 2588909,
    "title": "Gudstjeneste v/Test Testersen",
    "subtitle": "",
    "shortdescription": "",
    "description": "",
    "person": "Test Testersen",
    "locationname": "Gellerup Kirke",
    "category": "Højmesse",
    "kirkeid": 7314,
    "locationid": 42507,
    "location": "56.155773,10.133944",
    "organiserid": 39893,
    "organisername": "Gellerup Sogn",
    "sogneid": 9097,
    "eventtype": 1,
    "address1": "Gudrunsvej 88",
    "address2": "",
    "zip": "8220",
    "city": "Brabrand",
    "churchday": "3. s. i advent",
    "created": 1680605609000,
    "changed": 1702022718000,
    "deleted": false,
    "noenddate": false,
    "origin": 11,
    "moreurl": "",
    "img": "",
    "online": false,
    "dawa": "",
    "kommunenavn": "Aarhus Kommune",
    "provstiid": 817,
    "stiftid": 8,
    "startdate": "2023-12-17 10:00:00",
    "startdateTS": 1702803600000,
    "enddate": "2023-12-17 11:30:00",
    "enddateTS": 1702809000000
                },
                "sort": [
    1702807200000
                ]
            },
            {
                "_index": "event-2024-02-28-1425",
                "_id": "2596922",
                "_score": null,
                "_source": {
    "id": 2596922,
    "title": "Gudstjeneste",
    "subtitle": "",
    "shortdescription": "",
    "description": "",
    "person": "Test Testersen",
    "locationname": "Gellerup Kirke",
    "category": "Gudstjeneste",
    "kirkeid": 7314,
    "locationid": 42507,
    "location": "56.155773,10.133944",
    "organiserid": 39893,
    "organisername": "Gellerup Sogn",
    "sogneid": 9097,
    "eventtype": 1,
    "address1": "Gudrunsvej 88",
    "address2": "",
    "zip": "8220",
    "city": "Brabrand",
    "churchday": "",
    "created": 1682069430000,
    "changed": 1702894196000,
    "deleted": false,
    "noenddate": false,
    "origin": 11,
    "moreurl": "",
    "img": "",
    "online": false,
    "dawa": "",
    "kommunenavn": "Aarhus Kommune",
    "provstiid": 817,
    "stiftid": 8,
    "startdate": "2023-12-22 10:30:00",
    "startdateTS": 1703237400000,
    "enddate": "2023-12-22 11:30:00",
    "enddateTS": 1703241000000
                },
                "sort": [
    1703241000000
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
curl_setopt($ch, CURLOPT_URL, "https://es.sogn.dk/sognekalender/event/_search");
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

*Bemærk: eksemplerne kan indeholde linjeskift, som ikke bør være i forespørgslen.*

```curl
curl -XGET "https://es.sogn.dk/sognekalender/event/_search" -d '{
  "query": {
    "bool": {
      "must": [],
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
curl -XGET "https://es.sogn.dk/sognekalender/organiser/_search" -d '{
  "query": {
    "bool": {
      "must": [
        { "match": { "navn": "Gellerup" }}  
      ]
    }
  }
}'
```

Kan også laves som en direkte query string request:
`curl -XGET "https://webservice.sogn.dk/organiser/_search?q=navn:Gellerup"`  
*Bemærk at query strengen i parameteren `q` skal url encodes (hvis ikke din browser gør det for dig)*

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
    "max_score": 6.675736,
    "hits": [
      {
        "_index": "organiser-2024-01-17-1325",
        "_id": "39893",
        "_score": 6.675736,
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
          "aliases": "",
          "navnesuggest": "Gellerup Sogn fisk",
          "origin": 1
        }
      },
      {
        "_index": "organiser-2024-01-17-1325",
        "_id": "42507",
        "_score": 6.598057,
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

Man får altså 4 hits. Et hit for sognet (type: 1), et hit for kirken i Gellerup (type: 5) og to hits for Gellerup
Kirke (type: 4).  
Dette skyldes at der for typen organiser findes seks forskellige typer angivet fra integer feltet type (ikke at
forveksle med de tidligere omtalte ElasticSearch typer):  
1: Sogn (KIS),  
2: Provsti (KIS),  
3: Stift (KIS),  
4: Oprettede lokationer fra API leverandørers Sted angivelse  
5: Kirke (KIS),  
6: Brugeroprettede lokationer fra sogn.dk/admin  
Hvis man således kun ønsker at søge efter sogne kan man tilføje type: 1 til søgningen:

```curl
curl -XGET "https://es.sogn.dk/sognekalender/organiser/_search" -d '{
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

Kan også laves som en direkte query string request:  
```curl
curl -XGET "https://es.sogn.dk/sognekalender/organiser/_search?source=%7B%0A%20%20%22query%22%3A%20%7B%20%0A%20%20%20%20%22bool%22%3A%20%7B%20%0A%20%20%20%20%20%20%22must%22%3A%20%5B%0A%20%20%20%20%20%20%20%20%7B%20%22match%22%3A%20%7B%20%22navn%22%3A%20%22Gellerup*%22%20%7D%7D%2C%20%20%0A%20%20%20%20%20%20%20%20%7B%20%22match%22%3A%20%7B%20%22type%22%3A%201%20%7D%7D%20%20%0A%20%20%20%20%20%20%5D%0A%20%20%20%20%7D%0A%20%20%7D%0A%7D%0A" 
```
*Bemærk at guery strengen er url encodet direkte ud fra request body (men med source som parameter nu)*

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
  "took": 2,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "failed": 0
  },
  "hits": {
    "total": 1,
    "max_score": 0.0,
    "hits": [
      {
        "_index": "sognekalender-2022-06-24-01:55",
        "_type": "organiser",
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
          "origin": 1
        }
      }
    ]
  }
}
```


Ønsker man at begrænse resultatet til alle provstier eller stifte kan man således sætte et filter på typen (med 2 for
provstier og 3 for stifte).

Med denne forespørgsel fås således landets stifter:

```curl
curl -XGET "https://es.sogn.dk/sognekalender/organiser/_search" -d '{"query":{
 "bool":{
   "must":[],
   "filter":[
      {"term":{"deleted": false}},
      {"type":{"value":"organiser"}},
      {"term":{"type":3}}
   ]
 }
}}'
```

Man vil bemærke at man udover landets 10 stifter også får ”Døvemenigheder”, da de også figurerer som et administrativt stift i KIS.

### Kirke GET eksempler

```http request
GET https://es.sogn.dk/sognekalender/organiser/_search
```
Json Body
```json
{
  "query": {
    "bool": {
      "must": [],
      "filter": [
        {
          "term": {
            "type": 5
          }
        },
        {
          "term": {
            "deleted": false
          }
        },
        {
          "term": {
            "zip": "8000"
          }
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
    "failed": 0
  },
  "hits": {
    "total": 9,
    "max_score": null,
    "hits": [
      {
        "_index": "sognekalender-2022-06-24-01:55",
        "_type": "organiser",
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
          "origin": 1,
          "address1": "Skt.Lukas Kirkeplads 1",
          "address2": "",
          "zip": "8000",
          "city": "Aarhus C",
          "img": "https://sogn.dk/uploads/DyconSogneadmin/Organiser/8132_6603_1383140270.jpg",
          "location": "56.144753,10.193872",
          "kirkeid": 6603
        },
        "sort": [
          8132
        ]
      },
      {
        "_index": "sognekalender-2022-06-24-01:55",
        "_type": "organiser",
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
          "origin": 1,
          "address1": "Peter Sabroes Gade 20",
          "address2": "",
          "zip": "8000",
          "city": "Aarhus C",
          "img": "https://sogn.dk/uploads/DyconSogneadmin/Organiser/8120_6599_1393320149.jpg",
          "location": "56.169144,10.210092",
          "kirkeid": 6599
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

`mr` typen viser udvalgt data om alle landets menighedsråd (små 2000). Ved at kalde _mapping:
`https://webservice.sogn.dk/mr/_mapping`

En søgning uden filtre eller parametre vil give et resultat på alle landets menighedsråd (dog begrænset til de første 10
da det er standard når size parameteret udelades).  
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