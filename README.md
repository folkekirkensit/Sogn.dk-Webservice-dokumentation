# Webservices til kirkelige begivenheder fra de danske sogne og kirkelige enheder

(ElasticSearch version)

<!-- TOC -->
* [Webservices til kirkelige begivenheder fra de danske sogne og kirkelige enheder](#webservices-til-kirkelige-begivenheder-fra-de-danske-sogne-og-kirkelige-enheder)
  * [Generelt om disse services](#generelt-om-disse-services)
    * [Forældede services](#forældede-services)
  * [Begivenheder (typen event)](#begivenheder--typen-event-)
    * [Sammenligning af felter fra gamle services](#sammenligning-af-felter-fra-gamle-services)
    * [Eksempel på forespørgsel og svar<a href="#3-alternativ-til-url-encoding"><sup>3</sup></a>](#eksempel-på-forespørgsel-og-svar-a-href3-alternativ-til-url-encoding-sup-3-sup-a)
  * [Sogne, kirker og lokationer (typen organiser)](#sogne-kirker-og-lokationer--typen-organiser-)
    * [Organiser felter](#organiser-felter)
    * [Sogne, Provstier og Stifte GET eksempler](#sogne-provstier-og-stifte-get-eksempler)
      * [Json body til søgning efter Sogn på `sogneid`](#json-body-til-søgning-efter-sogn-på-sogneid)
      * [Json body til søgning efter Provsti på `provstiid`](#json-body-til-søgning-efter-provsti-på-provstiid)
      * [Json body til søgning efter Stift på `StiftID`](#json-body-til-søgning-efter-stift-på-stiftid)
    * [Kirke GET eksempler](#kirke-get-eksempler)
  * [Menighedsråd (typen mr)](#menighedsråd--typen-mr-)
  * [Præster (typen priest)](#præster--typen-priest-)
    * [Eksempel på forespørgsel efter præster for et givent sogn](#eksempel-på-forespørgsel-efter-præster-for-et-givent-sogn)
  * [Debug og test med Postman](#debug-og-test-med-postman)
  * [Kontakt](#kontakt)
  * [Referencer](#referencer)
    * [1 Gamle feeds](#1-gamle-feeds)
    * [2 Stackoverflow GET request med body](#2-stackoverflow-get-request-med-body)
    * [3 Alternativ til url encoding](#3-alternativ-til-url-encoding)
    * [4 Se ElasticSearch query string Syntax](#4-se-elasticsearch-query-string-syntax)
    * [5 Url encoding](#5-url-encoding)
    * [6 Bemærk query strengen](#6-bemærk-query-strengen)
  * [Ændringslog](#ændringslog)
    * [Version 4.1.0](#version-410)
    * [Version 4.0.3](#version-403)
    * [Version 4.0.2, udgivet 22-11-2021](#version-402-udgivet-22-11-2021)
    * [Version 4.0.1, udgivet 15-3-2021](#version-401-udgivet-15-3-2021)
    * [Version 3.0.3, udgivet 11-6-2019](#version-303-udgivet-11-6-2019)
    * [Version 3.0.2, udgivet 29-5-2017](#version-302-udgivet-29-5-2017)
    * [Version 3.0.1, udgivet 22-5-2017](#version-301-udgivet-22-5-2017)
<!-- TOC -->

## Generelt om disse services

Disse services er baseret på ElasticSearch vers. 5.1. Alle data er således gemt i et elasticsearch indeks, som er
offentligt tilgængeligt for GET requests.  
Data opdateres i nær-realtid fra sogn.dk. Data om begivenheder kommer dels fra sognenes egne indtastninger fra det
web-interface som sogn.dk tilbyder til oprettelse og redigering af begivenheder (sogn.dk/admin), dels fra de eksterne
leverandører som sogne kan have valgt at samarbejde med om deres kalenderløsning, som også synkroniseres med sogn.dk.
Grunddata om sogne, provstier, stifter, kirker, menighedsråd og præster kommer fra Kirkeministeriets
Informationssystem (KIS), som der synkroniseres med hver nat.  
Denne dokumentation beskriver de data der kan hentes og viser nogle eksempler på hvordan data kan trækkes ud.  
Disse services leverer alle data i UTF-8 format.  
Der er skelet noget til gamle feltnavne – under navngivningen af nye felter, men der er altså ikke 100% bagud
kompatibilitet med feltnavne. Se hvordan feltnavne fra de ”gamle” webservices relaterer sig til de nye navne i tabellen
i næste afsnit.  
Betegnelsen begivenhed bruges i dette dokument som fælles betegnelse for gudstjenester og arrangementer, selvom det ikke
er det samme, men har et stort sammenfald af egenskaber.

### Forældede services

Vi har i nogle år stillet data tilgængeligt via Yahoos proxy "sky-tjeneste" YQL. Denne service er nu lukket ned og det
opfordres til at bruge de services beskrevet her.  
De ”gamle” services til at udtrække data om arrangementer og gudstjenester<a href="#1-gamle-feeds"><sup>1</sup></a>
er forældede og forventes udfaset i løbet af 2020. Der er meget bedre svar-tider og søgemuligheder i de heri beskrevne
ElasticSearch indices, end i de gamle services.  
Ligeledes vil udtræk af data om præster, sogne og kirker med fordel kunne trækkes via det beskrevne ElasticSearch
indeks. Mere om det i de næste afsnit.

## Begivenheder (typen event)

Begivenheder, som er offentligt tilgængelige, kan findes via ElasticSearch indekset, sognekalender, fra ElasticSearch på
denne adresse:

<https://es.sogn.dk/>

Der henvises til dokumentationen på ElasticSearch hjemmesiden:

<https://www.elastic.co/guide/en/elasticsearch/reference/5.1/index.html>

- Hvor man skal være opmærksom på at det, pt. er version 5.1 af ElasticSearch vi benytter os af.

Man kan se hvilke ”typer” der er i indekset og hvordan deres mapping er ved at forespørge denne adresse:

<https://es.sogn.dk/sognekalender/_mappings>

Her fremgår det at der er fire typer; event, organiser, mr og priest. Et event har nogenlunde følgende properties:

```
  "address1": {"type": "text"},
  "address2": {"type": "text"},
  "category": {"type": "keyword"},
  "changed": {"type": "date"},
  "churchday": {"type": "text"},
  "city": {"type": "text"},
  "country": {"type": "keyword","null_value": "DK"},
  "created": {"type": "date"},
  "description": {"type": "text"},
  "enddate": {"type": "date","format": "dateOptionalTime||yyyy-MM-dd HH:mm:ss"},
  "enddateTS": {"type": "date"},
  "eventtype": {"type": "integer"},
  "id": {"type": "integer"},
  "img": {"type": "text"},
  "kirkeid": {"type": "integer"},
  "kommunenavn": {"type": "text"},
  "location": {"type": "geo_point","ignore_malformed": true},
  "locationid": {"type": "integer"},
  "locationname": {"type": "text","fields": {"raw": {"type": "keyword"},
  "moreurl": {"type": "text"},
  "noenddate": {"type": "boolean"},
  "organiserid": {"type": "integer"},
  "organisername": {"type": "text"},
  "origin": {"type": "integer"},
  "person": {"type": "text"},
  "provstiid": {"type": "integer"},
  "deleted": {"type": "boolean"},
  "shortdescription": {"type": "text"},
  "sogneid": {"type": "integer"},
  "startdate":{"type":"date","format":"dateOptionalTime||yyyy-MM-dd HH:mm:ss"},
  "startdateTS": {"type": "date"},
  "stiftid": {"type": "integer"},
  "subtitle": {"type": "text"},
  "title": {"type": "text"},
  "zip": {"type": "text"},
  "online": {"type": "boolean"},
  "dawa" : {"type":"text"}
```

De fleste af disse felter er selvforklarende. I næste afsnit sammenlignes de fleste felter med felterne fra de tidligere
services.  
Tidligere skulle man hente data om gudstjenester og arrangementer fra to separate services. Det behøver man ikke
længere. Man kan dog nemt filtrere sin søgning til kun at returnere data om den ene eller anden type af begivenheder.  
For at få den kommende måneds gudstjenester for et givent sogn kan man sende en GET request med nedenstående request
Body til:

`https://es.sogn.dk/sognekalender/event/_search -d '`

```JSON
{
  "query": {
    "bool": {
      "must": [
      ],
      "filter": [
        {
          "term": {
            "deleted": false
          }
        },
        {
          "term": {
            "sogneid": 7000
          }
        },
        {
          "term": {
            "eventtype": 1
          }
        },
        {
          "range": {
            "startdate": {
              "gte": "now",
              "lte": "now+1M"
            }
          }
        }
      ]
    }
  },
  "sort": [
    {
      "startdate": "ASC"
    }
  ],
  "size": 100
}
```

Nogle mener ikke at man kan eller bør sende en body i en GET request<a href="#2-stackoverflow-get-request-med-body"><sup>2</sup></a> (som skal være unik
alene ud fra den URL der requestes – ikke mindst for at kunne cache svar). Andre vil have svært ved at få deres
”framework” til at sende en request body med en GET request. Derfor har ElasticSearch også gjort det muligt at man kan
sende disse requests som POST. MEN da man med POST requests også har adgang til at opdatere og slette indhold, så har vi
som udgangspunkt ikke tilladt offentlig adgang til at kunne POST’e til denne service.  
Kontakt os hvis I har problemer med at få det til at virke. Vi bruger ikke en proxy foran denne service så det burde
virke at sende GET requests med en body – som i eksemplet ovenfor. Eksemplet vil i PHP med cUrl se nogenlunde således
ud:

```PHP
$params = array(
    "query"  => array(
        "bool" => array(
            "filter" => array(
                array(
                    "term" => array(
                        "sogneid" => 7000
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
                            "lte" => "now+1M"
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
    error_log('Searcherr: '.curl_error($ch).$result;
}
curl_close($ch);
return $resultArray;
```

Brug af ElasticSearch via javascript følger samme principper.
Bemærk at man nemt kan medtage en size og en from parameter til at opdele resultater i mindre mængder (se ElasticSearch
dokumentationen).

### Sammenligning af felter fra gamle services

I nedenstående tabel kan man se alle feltnavne for event typen fra ElasticSearch indekset sognekalender. Der er angivet
hvad feltet hed i de to gamle services og en kort beskrivelse af feltets indhold. Mange id'er kommer fra
Kirkeministeriets Informationssystem (KIS), hvilket så er angivet som KIS.

Alle felter kan bruges til at indsnævre en søgning. Afhængigt af typen kan man bruge forskellige typer af søgning og
filtre.
typen event:

| Feltnavn                                                                                                                                                                                                                            |                                                          arrangementer2012.php                                                          |                                                              gudstjenester2012.php                                                              | Forklaring                                                                                                                                                                                                                                                                                                                                                                                                                                               |
|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|:---------------------------------------------------------------------------------------------------------------------------------------:|:-----------------------------------------------------------------------------------------------------------------------------------------------:|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `kirkeid` (integer)                                                                                                                                                                                                                 |                                                                 kirkeId                                                                 |                                                                     kirkeId                                                                     | id som stammer fra KIS. <br>Vil kun være sat hvis en begivenhed foregår i en kirke.                                                                                                                                                                                                                                                                                                                                                                      |
| `sogneid` (integer)                                                                                                                                                                                                                 |                                                                 sogneId                                                                 |                                                                     sogneId                                                                     | id som stammer fra KIS.<br>Burde altid være med for begivenheder for sogne.                                                                                                                                                                                                                                                                                                                                                                              |
| `organiserid`                                                                                                                                                                                                                       |                                                               organiserId                                                               |                                                                   organiserId                                                                   | angiver det organiser id som en given arrangør har. Et sogn har feks et almindeligt sogneid,- men også et organiserid. Det er sognets organiserid som man kan bruge til at lave opslag direkte i den type der hedder organiser. Således vil et request til denne adresse: <br> <https://es.sogn.dk/sognekalender/organiser/39893> giver Gellerup Sogn, med de informationer der er gemt for sognet (stiftsid, provstiid, sogneid, kommunenavn, url, mm). |
| `locationid` (integer)                                                                                                                                                                                                              |                                                               lokationId                                                                |                                                                   lokationId                                                                    | locationid: angiver det organiser id som man kan bruge til at finde yderligere information om hvor en begivenhed foregår. Man kalder altså samme adresse som organiser søgningen ovenfor men nu med locationid.                                                                                                                                                                                                                                          |
| `provstiid` (integer)                                                                                                                                                                                                               |                                                                provstiId                                                                |                                                                    provstiId                                                                    | id som stammer fra KIS. <br> Burde altid være med for begivenheder for sogne.                                                                                                                                                                                                                                                                                                                                                                            |
| `deleted` (bool)                                                                                                                                                                                                                    |                                                            deleted (integer)                                                            |                                                                deleted (integer)                                                                | Hvis denne er true (tidligere sat til 1) er den tilhørende begivenhed blevet slettet eller usynliggjort og bør derfor ikke vises i lister eller kalendre.                                                                                                                                                                                                                                                                                                |
| `changed` (date, unix timestamp - antal millisekunder siden 1/1 1970). <br>Bemærk at denne date vises som antal millisekunder og ikke sekunder.<br>Man kan bruge alle "date" funktioner til at indsnævre en søgning via dette felt. |                                    changed (integer, unix timestamp - antal sekunder siden 1/1 1970)                                    |                                        changed (integer, unix timestamp - antal sekunder siden 1/1 1970)                                        | Dette date felt angiver hvornår en begivenhed sidst er ændret (eller oprettet). <br> Dette kan med fordel bruges til at tjekke om en begivenhed er opdateret/ændret siden man sidst hentede den.                                                                                                                                                                                                                                                         |
| `startdate` (date) <br> Bemærk at datoen vises uden tidszone angivelse, men er i dansk format (inkl. sommertid)                                                                                                                     |                                starttid-tekst <br> Denne havde samme format (feks: 2017-05-18 09:00:00)                                 |                                                       dato <br> Denne havde samme format                                                        | Startdato for hvornår en begivenhed starter. <br> Kan med fordel bruges i et filter i en søgning hvor der ønskes vist begivenheder som starter i en bestemt periode.                                                                                                                                                                                                                                                                                     |
| `startdateTS` (date) <br> Denne dato er ækvivalent med startdate, men vises som antal millisekunder siden 1/1 1970.                                                                                                                 |       starttid <br> Her var det sekunder siden 1/1 1970 (Unix Timestamp). Så vær opmærksom på at det nye felt er i millisekunder.       |           starttid <br> Her var det sekunder siden 1/1 1970 (Unix Timestamp). Så vær opmærksom på at det nye felt er i millisekunder.           | Alternativ representation af startdate, medtaget af hensyn til bagudkompatibilitet.                                                                                                                                                                                                                                                                                                                                                                      |
| `enddate` (date) <br> Bemærk at datoen vises uden tidszone angivelse, men er i dansk format (inkl. sommertid)                                                                                                                       |                                 sluttid-tekst <br> Denne havde samme format (feks: 2017-05-18 10:00:00)                                 |                                     sluttid-tekst <br> Denne havde samme format (feks: 2017-05-18 10:00:00)                                     | Dato og tid for hvornår en begivenhed slutter. Hvis den ikke er angivet under oprettelsen kan den være samme tid som starttid.                                                                                                                                                                                                                                                                                                                           |
| `enddateTS` (date) <br> Denne dato er ækvivalent med sluttid, men vises som antal millisekunder siden 1/1 1970.                                                                                                                     |       sluttid <br> Her var det sekunder siden 1/1 1970 (Unix Timestamp). Så vær opmærksom på at det nye felt er i millisekunder.        |           sluttid <br> Her var det sekunder siden 1/1 1970 (Unix Timestamp). Så vær opmærksom på at det nye felt er i millisekunder.            | Alternativ representation af enddate, medtaget af hensyn til bagudkompatibilitet.                                                                                                                                                                                                                                                                                                                                                                        |
| `stiftid` (integer)                                                                                                                                                                                                                 |       Fandtes ikke i disse services, men man kunne forespørge med denne parameter. Nu kan man sætte et filter på feltet i stedet.       |           Fandtes ikke i disse services, men man kunne forespørge med denne parameter. Nu kan man sætte et filter på feltet i stedet.           | id som stammer fra KIS. <br> Burde altid være med for begivenheder for sogne, hvor det er tilhørende stifts id der vises.                                                                                                                                                                                                                                                                                                                                |
| `location` (geo_point) <br> Her gemmes en lokations GPS koordinater (lat,lng). <br> Kan bruges til geo søgninger.                                                                                                                   |                        latitude og longitude var separate felter i begge feeds. De findes ikke separat længere.                         |                            latitude og longitude var separate felter i begge feeds. De findes ikke separat længere.                             | GPS efter WGS84 systemet. Eks: <br> `"location": "56.155773,10.133944"`, <br> Det er ikke alle lokationer der har koordinater, men kirker og en del andre har.                                                                                                                                                                                                                                                                                           |
| `id` (integer)                                                                                                                                                                                                                      | Var angivet som attribut på det moede element der indeholdt alle felter for et arrangement: `<arrangementer>`<br>`<moede id="1264114">` | Var angivet som attribut på det praediken element der indeholdt alle felter for en gudstjeneste: `<gudstjenester>`<br>`<praediken id="989358">` | Begivenhedens unikke id (er altid sat)                                                                                                                                                                                                                                                                                                                                                                                                                   |
| `title` (string)                                                                                                                                                                                                                    |                                                                  navn                                                                   |                                                                      navn                                                                       | Begivenhedens navn. Vil ofte være kategorien da denne sættes som default hvis titlen ikke er angivet under oprettelsen.                                                                                                                                                                                                                                                                                                                                  |
| `subtitle`                                                                                                                                                                                                                          |                                                               Undertitel                                                                |                                                                   Undertitel                                                                    | Undertitel var meget sjældent brugt. Indtil videre vil subtitle nok også sjældent blive brugt.                                                                                                                                                                                                                                                                                                                                                           |
| `shortdescription` (string)                                                                                                                                                                                                         |                                                                abstract                                                                 |                                                                    abstract                                                                     | En kort "teaser" tekst                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| `description` (text)                                                                                                                                                                                                                |                     text og text_plain. Begge disse gamle felter var uden formattering, og det er det nye felt også                     |                         text og text_plain. Begge disse gamle felter var uden formattering, og det er det nye felt også                         | En længere beskrivelse                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| `person` (string)                                                                                                                                                                                                                   |                                                              Fandtes ikke                                                               |                                                                     person                                                                      | Typisk er det den person (præst) der afholder begivenheden.                                                                                                                                                                                                                                                                                                                                                                                              |
| `eventtype`                                                                                                                                                                                                                         |                                    Fandtes ikke men var så implicit 2 når man kaldte arrangementer.                                     |                                        Fandtes ikke men var så implicit 1 når man kaldte gudstjenester.                                         | 1 angiver gudstjenester, 2 arrangementer                                                                                                                                                                                                                                                                                                                                                                                                                 |
| `origin`                                                                                                                                                                                                                            |                                                              Fandtes ikke                                                               |                                                                  Fandtes ikke                                                                   | Er mest et meta felt som vi bruger til at bestemme hvorfra en begivenhed er kommet ind til ES fra. Kan ignoreres.                                                                                                                                                                                                                                                                                                                                        |
| `moreurl` (string)                                                                                                                                                                                                                  |                                                                  link                                                                   |                                                                      link                                                                       | URL til en længere beskrivelse (alternativt til streaming adressen).                                                                                                                                                                                                                                                                                                                                                                                     |
| `noenddate` (bool)                                                                                                                                                                                                                  |                                                              ingen-sluttid                                                              |                                                                  ingen-sluttid                                                                  | Arrangøren har angivet at der ikke er en sluttid. Såfremt dette felt er sat kan man ikke regne med den værdi der fremgår af felterne for sluttid.                                                                                                                                                                                                                                                                                                        |
| `online` (bool)                                                                                                                                                                                                                     |                                                        Fandtes ikke (nu online)                                                         |                                                            Fandtes ikke (nu online)                                                             | Angiver at begivenheden streames via internettet. Begivenheden kan stadig godt have en fysisk afvikling også. Mere information bør fremgå af feltet description. Ligesom en URL til streamen kan forventes at findes der eller i feltet moreurl.                                                                                                                                                                                                         |
| `dawa` (string)                                                                                                                                                                                                                     |                                                        Fandtes ikke (nu dawaid)                                                         |                                                            Fandtes ikke (nu dawaid)                                                             | Eksperimentel! Man kan ikke regne med at denne værdi er sat. <br> Adgangsadressens unikke id, f.eks. 0a3f5095-45ec-32b8-e044-0003ba298018 som anvendt af DAWA <br> Se <http://origin-dawa-p1.aws.dk/dok/api/adgangsadresse> feltet id.                                                                                                                                                                                                                   |

### Eksempel på forespørgsel og svar<a href="#3-alternativ-til-url-encoding"><sup>3</sup></a>

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
  "took": 3,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "failed": 0
  },
  "hits": {
    "total": 35,
    "max_score": null,
    "hits": [
      {
        "_index": "sognekalender-2017-06-27-15:17",
        "_type": "event",
        "_id": "2132613",
        "_score": null,
        "_source": {
          "id": 2132613,
          "title": "Test With link JS prod",
          "subtitle": "",
          "shortdescription": "",
          "description": "test link is available",
          "person": "Karen Elisabeth Huus",
          "locationname": "Testkirke A",
          "category": "Gudstjeneste",
          "kirkeid": 9000,
          "locationid": 42609,
          "location": "0,0",
          "organiserid": 38224,
          "organisername": "testsogn",
          "provstiid": "300",
          "stiftid": "8",
          "kommunenavn": "Københavns Kommune",
          "sogneid": 7000,
          "eventtype": 1,
          "address1": "Addresse A",
          "address2": "Addresse A",
          "zip": "1010",
          "city": "Lokation A",
          "created": 1614939025000,
          "changed": 1614939025000,
          "deleted": false,
          "noenddate": false,
          "origin": 3,
          "moreurl": "https://medarbeideren.dk/appointment?id=8cbfce55-7ac4-485d-8889#",
          "img": "",
          "startdate": "2021-03-15 11:00:00",
          "startdateTS": 1615802400000,
          "enddate": "2021-03-15 12:00:00",
          "enddateTS": 1615806000000,
          "online": true,
          "dawa": ""
        },
        "sort": [
          1615806000000
        ]
      }
    ]
  }
}
```

Man kan se at der er 35 resultater af søgningen (men kun en returneret grundet size parameteren på 1). Man kan se at
søgningen (selve søgningen internt i ElasticSearch) har taget 3 millisekunder. Dette er vel at mærke ikke tiden det
tager fra man sender et request til man modtager svaret (der er en del andre tider involveret i den proces). Men det er
stadig en væsentlig forbedring over vores sædvanlige PHP / MySQL baserede søgninger.

## Sogne, kirker og lokationer (typen organiser)

Exsempel på søgning efter et sogn ud fra et navn<a href="#4-se-elasticsearch-query-string-syntax"><sup>4</sup></a>

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
`curl -XGET "https://es.sogn.dk/sognekalender/organiser/_search?q=navn:Gellerup"` <a href="#5-url-encoding"><sup>5</sup></a>

Resultatet for begge ovenstående forespørgsler er:

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
    "total": 4,
    "max_score": 7.9810753,
    "hits": [
      {
        "_index": "sognekalender-2017-06-27-15:17",
        "_type": "organiser",
        "_id": "42507",
        "_score": 7.9810753,
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
          "changed": 1556632009000,
          "origin": 1,
          "address1": "Gudrunsvej 88",
          "address2": "",
          "zip": "8220",
          "city": "Brabrand",
          "img": "https://sogn.dk/uploads/DyconSogneadmin/Organiser/uid42507-ImageCropped-631c91.jpg",
          "location": "56.155773,10.133944",
          "kirkeid": "7314"
        }
      },
      {
        "_index": "sognekalender-2017-06-27-15:17",
        "_type": "organiser",
        "_id": "39893",
        "_score": 7.902451,
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
          "origin": 1
        }
      },
      {
        "_index": "sognekalender-2017-06-27-15:17",
        "_type": "organiser",
        "_id": "85173",
        "_score": 7.8435044,
        "_source": {
          "id": 85173,
          "navn": "Gellerup Kirke",
          "sogneid": 9135,
          "provstiid": "817",
          "stiftid": "8",
          "type": 4,
          "origin": 3,
          "organiserid": "39894",
          "kommunenavn": "Aarhus Kommune"
        }
      },
      {
        "_index": "sognekalender-2017-06-27-15:17",
        "_type": "organiser",
        "_id": "86217",
        "_score": 7.8435044,
        "_source": {
          "id": 86217,
          "navn": "Gellerup Kirke",
          "sogneid": 8130,
          "provstiid": "817",
          "stiftid": "8",
          "type": 4,
          "origin": 3,
          "organiserid": "39890",
          "kommunenavn": "Aarhus Kommune"
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

Kan også laves som en direkte query string request<a href="#6-bemærk-query-strengen"><sup>6</sup></a>:

```curl
curl -XGET "https://es.sogn.dk/sognekalender/organiser/_search?source=%7B%0A%20%20%22query%22%3A%20%7B%20%0A%20%20%20%20%22bool%22%3A%20%7B%20%0A%20%20%20%20%20%20%22must%22%3A%20%5B%0A%20%20%20%20%20%20%20%20%7B%20%22match%22%3A%20%7B%20%22navn%22%3A%20%22Gellerup*%22%20%7D%7D%2C%20%20%0A%20%20%20%20%20%20%20%20%7B%20%22match%22%3A%20%7B%20%22type%22%3A%201%20%7D%7D%20%20%0A%20%20%20%20%20%20%5D%0A%20%20%20%20%7D%0A%20%20%7D%0A%7D%0A" 
```

Resultatet er nu at man kun får Gellerup Sogn i svaret.
Hvis man kun vil finde kirker, kan man så i stedet sætte type til 5.  
I nedenstående tabel kan man se hvilke properties / felter der er for typen organiser samt hvad de tilsvarende hedder i
de "gamle" service feed (sogne.php og kirker.php), samt en kort beskrivelse af indholdet.  
Vi kan se at properties fra mappings for organiser typen via:
`https://es.sogn.dk/sognekalender/organiser/_mapping`

```
"address1": {"type": "text"},
"address2": {"type": "text"},
"changed": {"type": "date"},
"city": {"type": "text"},
"country": {"type": "keyword","null_value": "DK"}
"created": {"type": "date"},
"id": {"type": "integer"},
"img": {"type": "text","index": false},
"kirkeid": {"type": "integer"},
"kommunenavn": {"type": "text"},
"location": {"type": "geo_point","ignore_malformed": true},
"navn": {"type": "text","fields": {
    "raw": {"type": "keyword"}
  },
},
"organiserid": {"type": "integer"},
"origin": {"type": "integer"},
"ownurl": {"type": "text"},
"provstiid": {"type": "integer"},
"deleted": {"type": "boolean"},
"sogndkurl": {"type": "text"},
"sogneid": {"type": "integer"},
"stiftid": {"type": "integer"},
"type": {"type": "integer"},
"zip": {"type": "text"}
```

### Organiser felter

| Feltnavn                                                                                                                                                                                                                          |                                        sogne.php                                         |                                        kirker.php                                        | Forklaring                                                                                                                                                                                                                                                                                                       |
|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|:----------------------------------------------------------------------------------------:|:----------------------------------------------------------------------------------------:|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `kirkeid` (integer)                                                                                                                                                                                                               |                                         kirkeId                                          |                                         kirkeId                                          | id som stammer fra KIS. <br> Vil kun være sat for kirker.                                                                                                                                                                                                                                                        |
| `navn` (text)                                                                                                                                                                                                                     |                                        kirkenavn                                         |                                        kirkenavn                                         | Navnet på organiser/lokation (som jo kan være af forskellig type). Se type.                                                                                                                                                                                                                                      |
| `type` (integer)                                                                                                                                                                                                                  |             Fandtes ikke (var implicit 1 for sogne.php og 5 for kirker.php)              |             Fandtes ikke (var implicit 1 for sogne.php og 5 for kirker.php)              | Typen af organiser: <br> 1: Sogn (KIS), <br> 2: Provsti (KIS), <br> 3: Stift (KIS), <br> 4: Oprettede lokationer fra API leverandørers Sted angivelse <br> 5: Kirke (KIS), <br> 6: Brugeroprettet lokation fra sogn.dk/admin                                                                                     |
| `sogneid` (integer)                                                                                                                                                                                                               |                                         sogneId                                          |                                         sogneId                                          | id som stammer fra KIS. <br> Er også sat for lokationer oprettet af sognebrugere, hvor det så er sognets id der står angivet.                                                                                                                                                                                    |
| `provstiid` (integer)                                                                                                                                                                                                             |                                        provstiid                                         |                                        provstiId                                         | id som stammer fra KIS.                                                                                                                                                                                                                                                                                          |
| `stiftid` (integer)                                                                                                                                                                                                               |                                         stiftid                                          |                                         stiftId                                          | id som stammer fra KIS.                                                                                                                                                                                                                                                                                          |
| `Id`                                                                                                                                                                                                                              |                                       Fandtes ikke                                       |                                        locationid                                        | Organiser id. Er så et unikt id på tværs af de forskellige organiser typer.                                                                                                                                                                                                                                      |
| `changed` (date, unix timestamp - antal millisekunder siden 1/1 1970). Bemærk at denne date vises som antal millisekunder og ikke sekunder. <br> Man kan bruge alle "date" funktioner til at indsnævre en søgning via dette felt. |                                       Fandtes ikke                                       |                                       Fandtes ikke                                       | Dette date felt angiver hvornår en organiser sidst er ændret (eller oprettet).                                                                                                                                                                                                                                   |
| `location` (geo_point) <br> Her gemmes en lokations GPS koordinater (lat,lng). <br> Kan bruges til geo søgninger.                                                                                                                 | latitude og longitude var separate felter i begge feeds. De findes ikke separat længere. | latitude og longitude var separate felter i begge feeds. De findes ikke separat længere. | GPS efter WGS84 systemet. Eks: <br> "location": "56.155773,10.133944", <br> Det er ikke alle lokationer der har koordinater, men kirker og en del andre har. Lokationer af typen 4 (oprettet via API) har næsten aldrig koordinater. Ej heller sogne, provstier og stifte (typerne 1,2 og 3).                    |
| `origin`                                                                                                                                                                                                                          |                                       Fandtes ikke                                       |                                       Fandtes ikke                                       | Er mest et meta felt som vi bruger til at bestemme hvorfra en organiser er kommet ind til ES fra. Kan ignoreres.                                                                                                                                                                                                 |
| `sogndkurl`                                                                                                                                                                                                                       |                                        sogndkurl                                         |                                        sogndkurl                                         | Er primært relevant for sogne (organisere af typen 1). Angiver den URL som et sogn har på sogn.dk (feks: sogn.dk/gellerup)                                                                                                                                                                                       |
| `ownurl`                                                                                                                                                                                                                          |                                          ownurl                                          |                                          ownurl                                          | Angiver en eventuel egen URL (hjemmeside). Dette er også primært relevant/sat for sogne.                                                                                                                                                                                                                         |
| `dawa` (string)                                                                                                                                                                                                                   |                                   Fandtes/findes ikke                                    |                                   Fandtes/findes ikke                                    | Eksperimentel! Man kan ikke regne med at denne værdi er sat. <br> Adgangsadressens unikke id, f.eks. 0a3f5095-45ec-32b8-e044-0003ba298018 som anvendt af DAWA <br> Se <https://dawadocs.dataforsyningen.dk/dok/api/adgangsadresse> feltet id. <br> Vil nok kun være sat / relevant for lokationer (typen 4,5,6). |

### Sogne, Provstier og Stifte GET eksempler

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

Liste over stifter findes her da de ændres sjældent. 

| Id  |      Stitfs navn       |          Url           |
|:----|:----------------------:|:----------------------:|
| 1   |    Københavns Stift    | www.kobenhavnsstift.dk |
| 2   |    Helsingør Stift     | www.helsingoerstift.dk |
| 3   |     Roskilde Stift     |  www.roskildestift.dk  |
| 4   | Lolland-Falsters Stift |     www.lfstift.dk     |
| 5   |      Fyens Stift       |   www.fyensstift.dk    |
| 6   |     Aalborg Stift      |  www.aalborgstift.dk   |
| 7   |      Viborg Stift      |   www.viborgstift.dk   |
| 8   |      Århus Stift       |   www.aarhusstift.dk   |
| 9   |       Ribe Stift       |    www.ribestift.dk    |
| 10  |    Haderslev Stift     | www.haderslevstift.dk  |

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

Man vil bemærke at man udover landets 10 stifter også får ”Døvemenigheder”, da de også figurerer som et administrativt
stift i KIS.

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

## Menighedsråd (typen mr)

Mr typen viser udvalgt data om alle landets menighedsråd (små 2000). Ved at kalde _mapping:
`https://es.sogn.dk/sognekalender/mr/_mapping`

\- ses de felter der er tilgængelige.

En søgning uden filtre eller parametre vil give et resultat på alle landets menighedsråd (dog begrænset til de første 10
da det er standard når size parameteret udelades).  
`https://es.sogn.dk/sognekalender/mr/_search`  
resultat (her vises blot det første):

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
    "total": 1703,
    "max_score": 1,
    "hits": [
      {
        "_index": "sognekalender-2017-06-27-15:17",
        "_type": "mr",
        "_id": "8357",
        "_score": 1,
        "_source": {
          "id": 8357,
          "type": 11,
          "stiftid": 6,
          "mrkode": 8357,
          "provstiid": 603,
          "organiserids": "39015",
          "sogneids": "8357",
          "sognenames": "Store Ajstrup Sogn",
          "sogndkurls": "https://sogn.dk/storeajstrup/menighedsraad/",
          "navn": "Store Ajstrup Sogns Menighedsråd",
          "email": "8357@SOGN.DK",
          "kommunenavn": "Aalborg Kommune",
          "country": "DK",
          "deleted": false,
          "changed": 1497882625000,
          "origin": 1
        }
      }
    ]
  }
}
```

Igen er det vigtigt altid at huske at medtage et `deleted:false` filter, da man ellers får slettede menighedsråd med.

## Præster (typen priest)

Man kan søge efter landets ansatte præster ved at søge i typen priest. De forskellige felter kan ses ved at kalde
mapping:

`https://es.sogn.dk/sognekalender/priest/_mapping`

Resultat:

```json
{
  "sognekalender-2017-06-27-15:17": {
    "mappings": {
      "priest": {
        "properties": {
          "changed": {
            "type": "date"
          },
          "created": {
            "type": "date"
          },
          "deleted": {
            "type": "boolean"
          },
          "id": {
            "type": "integer"
          },
          "mrkodetilknytninger": {
            "type": "text",
            "analyzer": "default",
            "search_analyzer": "default_search"
          },
          "navn": {
            "type": "text",
            "fields": {
              "raw": {
                "type": "keyword"
              }
            },
            "analyzer": "default",
            "search_analyzer": "default_search"
          },
          "organisertilknytninger": {
            "type": "text",
            "analyzer": "default",
            "search_analyzer": "default_search"
          },
          "origin": {
            "type": "integer"
          },
          "provstier": {
            "type": "text",
            "analyzer": "default",
            "search_analyzer": "default_search"
          },
          "sogne": {
            "type": "text",
            "analyzer": "default",
            "search_analyzer": "default_search"
          },
          "stifte": {
            "type": "text",
            "analyzer": "default",
            "search_analyzer": "default_search"
          },
          "stillingskoder": {
            "type": "text",
            "analyzer": "default",
            "search_analyzer": "default_search"
          }
        }
      }
    }
  }
}
```

Et svar til `_search` (uden parametre kan give et resultat som nedenfor):

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
    "total": 2197,
    "max_score": 1,
    "hits": [
      {
        "_index": "sognekalender-2017-06-27-15:17",
        "_type": "priest",
        "_id": "314",
        "_score": 1,
        "_source": {
          "id": 314,
          "navn": "Henriette Leslie Idestrup",
          "sogne": "7780",
          "provstier": "501",
          "stifte": "5",
          "mrkodetilknytninger": "7780",
          "stillingskoder": "9254",
          "organisertilknytninger": "38754",
          "deleted": false,
          "origin": 1,
          "created": 1498569446000,
          "changed": 1498569446000
        }
      },
      {
        "_index": "sognekalender-2017-06-27-15:17",
        "_type": "priest",
        "_id": "356",
        "_score": 1,
        "_source": {
          "id": 356,
          "navn": "Lisbeth Munk Madsen",
          "sogne": "7143",
          "provstier": "210",
          "stifte": "2",
          "mrkodetilknytninger": "7143",
          "stillingskoder": "9255",
          "organisertilknytninger": "38271",
          "deleted": false,
          "origin": 1,
          "created": 1498569446000,
          "changed": 1498569446000
        }
      },
      {
        "_index": "sognekalender-2017-06-27-15:17",
        "_type": "priest",
        "_id": "387",
        "_score": 1,
        "_source": {
          "id": 387,
          "navn": "Christina Radsted H Feddersen",
          "sogne": "7200 7201",
          "provstier": "313",
          "stifte": "3",
          "mrkodetilknytninger": "7200 7201",
          "stillingskoder": "9255",
          "organisertilknytninger": "38636 38637",
          "deleted": false,
          "origin": 1,
          "created": 1498569446000,
          "changed": 1498569446000
        }
      }
    ]
  }
}
```

Bemærk specielt den sidste præst med "id": 387, som har to ”sogne”, to ”mrkodetilknytninger” og to
”organisertilknytninger”. Det vil sige, at denne præst har flere tilknytninger (præst for to sogne). En præst kan have
mange tilknytninger, så denne liste kunne bestå af væsentligt flere sogne, mm.  
Bemærk at værdierne er adskilt af mellemrum.  
For de leverandører der indleverer kalender data på vegne af et sogn til sogn.dk,- er det specielt interessant at forstå
hvordan præstens unikke id for en sognerelation opbygges.  
Generelt består en præsts id (som det feks bruges som attributten id for feltet praest i DTD’en for indlevering af
begivenheder <https://sogn.dk/api/dtd/api.dtd>) af:

`<id>` + `<sognenr>` + `<stillingskode>` (hvor + blot er en konkatenering)

Så I tilfælde af flere sognetilknytninger, skal man bruge det relevante sogns nummer. For præsten med id 387 vil denne
have to præste id’er som kunne bruges til at angive denne præst som præst for en begivenhed for hhv sogn med nr 7200 
og 7201. Nemlig 38772009255 og 38772019255.  
Såfremt der er flere stillingskoder vil de relevante koder for præster være en af de følgende: 
(rangeret med hyppigst forekommende øverst)

9255 Sognepræst (kirkebogsfører)  
9254 Sognepræst  
9023 Sognepræst og korshærspræst  
9264 Provst (kirkebogsfører)  
9655 Hospitalpræst

### Eksempel på forespørgsel efter præster for et givent sogn

Man kan enten lave en søgning på hele indekset (ved udeladelse af /priest/):

`https://es.sogn.dk/sognekalender/_search`

Eller en direkte søgning hvor det kun er typen priest der søges i:

`https://es.sogn.dk/sognekalender/priest/_search`

Hvis man søger på hele indekset vil det typisk være nødvendigt at medtage et type filter for at filtrere resultater fra
andre typer fra:

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
        },
        {
          "type": {
            "value": "priest"
          }
        }
      ]
    }
  }
}
```

Resultatet af ovenstående forespørgsel vil være en liste med præster for sognet med nr. 7201:

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
    "total": 3,
    "max_score": 0,
    "hits": [
      {
        "_index": "sognekalender-2017-06-27-15:17",
        "_type": "priest",
        "_id": "387",
        "_score": 0,
        "_source": {
          "id": 387,
          "navn": "Christina Radsted H Feddersen",
          "sogne": "7200 7201",
          "provstier": "313",
          "stifte": "3",
          "mrkodetilknytninger": "7200 7201",
          "stillingskoder": "9255",
          "organisertilknytninger": "38636 38637",
          "deleted": false,
          "origin": 1,
          "created": 1498569446000,
          "changed": 1498569446000
        }
      },
      {
        "_index": "sognekalender-2017-06-27-15:17",
        "_type": "priest",
        "_id": "423",
        "_score": 0,
        "_source": {
          "id": 423,
          "navn": "Kirsten Senbergs",
          "sogne": "7200 7201",
          "provstier": "313",
          "stifte": "3",
          "mrkodetilknytninger": "7200 7201",
          "stillingskoder": "9023",
          "organisertilknytninger": "38636 38637",
          "deleted": false,
          "origin": 1,
          "created": 1498569446000,
          "changed": 1498569446000
        }
      },
      {
        "_index": "sognekalender-2017-06-27-15:17",
        "_type": "priest",
        "_id": "235382",
        "_score": 0,
        "_source": {
          "id": 235382,
          "navn": "Marlene Evensen",
          "sogne": "7200 7201 7411 7410",
          "provstier": "313 208",
          "stifte": "3 2",
          "mrkodetilknytninger": "7200 7201 7411 7410",
          "stillingskoder": "9023",
          "organisertilknytninger": "38636 38637 38223 38222",
          "deleted": false,
          "origin": 1,
          "created": 1498569446000,
          "changed": 1546308001000
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

Folkekirkens IT  
E-post: it-kontoret@km.dk  
Telefon: 70 20 25 35  
Telefontid: mandag-torsdag kl. 8.30-16.00 - fredag kl. 9.00-14.00  
Hjemmeside: https://folkekirkensit.dk

## Referencer

### 1 Gamle feeds

<https://sogn.dk/xmlfeeds/arrangementer2012.php>,  
<https://sogn.dk/xmlfeeds/arrangementer.php>,  
<https://sogn.dk/xmlfeeds/gudstjenester2012.php> og  
<https://sogn.dk/xmlfeeds/gudstjenester.php>

### 2 Stackoverflow GET request med body

<https://stackoverflow.com/questions/36939748/elasticsearch-get-request-with-request-body>

### 3 Alternativ til url encoding

Se alternativ mulighed for at url_encode hele requestet (i stedet for at sende en request body):
<https://stackoverflow.com/questions/14339696/elasticsearch-post-with-json-search-body-vs-get-with-json-in-url>

### 4 Se ElasticSearch query string Syntax 

Se <https://www.elastic.co/guide/en/elasticsearch/reference/5.1/query-dsl-query-string-query.html#query-string-syntax>

### 5 Url encoding

Bemærk at query strengen i parameteren q skal url encodes (hvis ikke din browser gør det for dig)

### 6 Bemærk query strengen

Bemærk at guery strengen er url encodet direkte ud fra request body (men med source som parameter nu)

## Ændringslog

### Version 4.1.0

- Tilføjet eksempler på udtræk af sogne, provstier, stifter og kirker.
- Formaterings ændringer.

### Version 4.0.3

- Tilrettet tekst og tilføjet manglende postman billeder.

### Version 4.0.2, udgivet 22-11-2021

- Publiceret til GitHub

### Version 4.0.1, udgivet 15-3-2021

- Tilføjede beskrivelse for de nyligt tilføjede felter for dawa og online for events og dawa for organisers.
- Tilføjede beskrivelse af moreurl og noenddate for events.
- Opdaterede eksempel for event resultat af søgning.
- Små rettelser til tekst og grammatik.

### Version 3.0.3, udgivet 11-6-2019

- Tilføjede beskrivelse til hentning af data om Præster og Menighedsråd. Og enkelte rettelser til de gamle beskrivelser
  for hentning af data om begivenheder, sogne, provstier og lokationer.
- Tilføjede afsnit om brugen af Postman til at teste og debugge forespørgsler med.

### Version 3.0.2, udgivet 29-5-2017

- Opdaterede URL til es.sogn.dk (var sat til den ret lange URL som AWS sætter under oprettelsen af en ES instans).

### Version 3.0.1, udgivet 22-5-2017

- Nyt API baseret på ElasticSearch. Ny dokumentation i første version. Det forventes at der kommer flere typer til og
  flere eksempler på hentning af data.
