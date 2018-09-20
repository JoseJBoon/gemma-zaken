---
title: "Ontwerpkeuzes"
description: ""
weight: 30
menu:
  docs:
    parent: "developers"
---

## UUID4 als ID-parameter in endpoints

De ID-parameter, hieronder aangeduid met `{uuid}` wordt gebruikt om via de URL
een enkel object van een bepaald type resource te vinden. Bijvoorbeeld een
`Zaak`:

|URL|Voorbeeld|
|---|---|
| `https://www.example.com/zaken/{uuid}/`|`https://www.example.com/zaken/550e8400-e29b-41d4-a716-446655440000/`|

Een [UUID (versie 4)] is in de praktijk altijd uniek, zonder dat deze centraal
hoeft te worden bijgehouden. Deze keuze laat onverlet de mogelijkheid om op
andere manieren bij een enkel object te komen, zoals op een combinatie van
velden die samen uniek zijn, zoals `bronorganisatie` en `zaakidentificatie`:
`https://www.example.com/zaken/?bronorganisatie=0329&zaakidentificatie=MOR-0000001`

*Achtergrond*
De paden van API endpoints bevatten referenties naar de objecten in de
achterliggende datastore. Deze parameters zouden semantisch kunnen ingevuld
worden, zoals gebruikmaken van `bronorganisatie` en `zaakidentificatie` voor
een zaak. Echter, na analyse blijkt dat dit lastig consequent toe te passen is
door de hele de API heen. Tevens wekt het de indruk dat dat deze parameters
genoeg zijn om een Zaak te vinden maar dat is niet correct. De volledige URL is
nodig voor het opvragen van een enkele Zaak.

Daarom is besloten om gebruik te maken van [UUID (versie 4)] voor deze
parameters. De motivatie is verder dat deze:

* uniciteit garandeert, ook over meerdere systemen heen;
* geen volgordelijkheid/database IDs lekt;
* de DSO API-richtlijnen volgt;
* semantisch bevragen niet onmogelijk maakt.

[UUID (versie 4)]: https://en.wikipedia.org/wiki/Universally_unique_identifier#Version_4_(random)


## ISO-8601 durations voor uitdrukken van duur

Voor het uitdrukken van duur wordt gebruikt gemaakt van [ISO-8601 durations](https://en.wikipedia.org/wiki/ISO_8601#Durations). Dit sluit aan bij ISO-8601 weergave van timestamps doorheen de API.


## Omgang met polymorfe resources

In het RGBZ zijn er voor sommige relaties meerdere types van gerelateerde
resources mogelijk. Concrete voorbeelden hiervan zijn:

* betrokkenen bij een zaak - dit kunnen `Natuurlijke Personen`, `Medewerkers`,
  `Organisatorische Eenheden` en meer zijn. Elk van deze betrokkenen heeft
  verschillende attributen, en een aantal komen (bijna) overal voor.

* objecten gerelateerd aan een zaak via ZAAKOBJECT. Hier zijn erg veel types
  mogelijk, bijvoorbeeld: `Huishouden`, `OpenbareRuimte`, `Wegdeel` etc.

In essentie is dit een vorm van polymorfisme.

Door het uitgangspunt van Common Ground om data-bij-de-bron-opslaan te hanteren,
werken we in de ZDS 2.0 API specificaties met _linked data_, wat betekent
dat de relaties een referentie-url geven naar de gerelateerde resource. Een
client/consumer weet op voorhand niet naar welke vorm van gerelateerde resource
er verwezen wordt.

Daarom is ervoor gekozen om op de relatie bij te houden om welk type resource
het gaat. Een concreet voorbeeld van een response is dan:

```http
GET /zrc/api/v1/rollen/fb1f6871-6dad-4f9b-abf8-0c46797b084a
Content-Type: application/json

{
    "url": "https://example.com/zrc/api/v1/rollen/fb1f6871-6dad-4f9b-abf8-0c46797b084a",
    "zaak": "https://example.com/zrc/api/v1/zaken/6232ba6a-beee-4357-b054-6bde6d1ded6c",
    "betrokkene": "https://brp.utrecht.nl/api/v1/np/ab44ed92-ff55-4c4c-87aa-c538d58e887d",
    "betrokkene_type": "Natuurlijk persoon",
}
```

Het gaat hier dan om het `betrokkene_type` veld.

De URLs in dit voorbeeld zijn uiteraard fictief.


## Naamgeving van de API velden binnen een resource

* Prefixes die slaan op de eigen resource, worden niet gebruikt. Voorbeeld:
  Attribuutsoort [informatieobjecttype.informatieobjecttype-omschrijving](https://www.gemmaonline.nl/index.php/Imztc_2.1/doc/attribuutsoort/informatieobjecttype.informatieobjecttype-omschrijving)
  in RGBZ2 krijgt de naam "omschrijving" in de API, aangezien het een veld is
  binnen de resource `informatieobjecttype`. Dit komt overeen met de in RGBZ
  genoemde XML-tag.

  ```javascript
  {
      "url": "https://example.com/drc/api/v1/informatieobjecttypen/8534ba6a-bcde-4387-b054-6bde6d1ded8f",
      "omschrijving": "Document"
      // ...
  }
  ```
* Relaties worden **niet** aangeduid met hun relatie omschrijving (`isVan`,
  `heeft`, `kent`, etc.). Dit heeft mede te maken met het feit dat er geen
  mnemonic attributen zijn (zoals in StUF-ZKN) in de REST API, waardoor het
  niet direct duidelijk is naar wat voor type resource verwezen wordt.
* Indien verwezen wordt naar een andere resource (relatie), wordt de volledige
  naam van de resource gebruikt als veldnaam. Voorbeeld: De resource `zaak`
  heeft een veld `zaaktype` en niet `type`. Ter vergelijking, in StUF-ZKN is
  dit: `zaak-isVan-<zaak type data>` waar `isVan` verwijst naar het `zaaktype`.

  ```javascript
  {
      "url": "https://example.com/zrc/api/v1/zaken/6232ba6a-beee-4357-b054-6bde6d1ded6c",
      "zaaktype": "https://example.com/ztc/api/v1/zaaktypen/6232ba6a-beee-4357-b054-6bde6d1ded6c"
      // ...
  }
  ```


## Gerelateerde objecten zonder eigen resource / Groepsattributen

Indien een (hoofd)object een gerelateerd object (of lijst van objecten) heeft,
dat **geen** eigen resource URL nodig heeft, dan wordt het (child)object inline
(binnen het hoofdobject) ontsloten. Deze child objecten zijn binnen RGBZ
ook wel gedefinieerd als *groepsattributen*. Typische voorbeelden zijn
(zaak-)kenmerken of (zaak-)eigenschappen waarin `Zaak` het hoofdobject betreft,
en `Kenmerk` en `Eigenschap` de child objecten.

Voorbeeld:

```javascript
{
    "url": "https://example.com/zrc/api/v1/zaken/6232ba6a-beee-4357-b054-6bde6d1ded6c",
    "zaaktype": "https://example.com/ztc/api/v1/zaaktypen/6232ba6a-beee-4357-b054-6bde6d1ded6c"
    // ...
    "kenmerken": [
        {
            "kenmerk": "test",
            "bron": "http://www.example.com/"
        }
    ]
}
```


## Input validatie

Componenten die APIs aanbieden (ZRC, DRC, ZTC) moet aan volledige
inputvalidatie doen. Dit betekent dat er _meer_ validatie nodig is dan enkel
garanderen dat een veld aan het url-formaat voldoet, en dat er communicatie
tussen systemen is in het geval van gedistribueerde componenten.

Een concreet voorbeeld hiervan is het zetten van een STATUS op een ZAAK:

1. Het aanmaken van een status gebeurt door een nieuwe status te POSTen, met
   onder andere de URL van het `statusType`.
2. Het ZRC MOET deze `statusType` url opvragen uit het ZTC, en controleren dat
   het zaaktype van dit statustype overeenkomt met het zaaktype van de
   betreffende zaak.

De foutberichten voor deze validatie zullen opgesteld worden en opgenomen in
de API spec.

Noot: voor suites kan de implementatie hiervan uiteraard afwijken - indien
alles in 1 enkele database leeft, en de suite ontsluit 3 APIs, dan kan prima
de validatie intern gebeuren op een geoptimaliseerde manier. Belangrijk is dat
de fout-responses (zie ook #130) correct teruggegeven worden.

Noot 2: authenticatie en authorisatie worden in een later stadium uitgewerkt.
Het is bekend dat hier goed over nagedacht moet worden.


## Besluitenregistratie (BRC)

Er komt een aparte besluitenregistratie (naast ZRC, ZTC en DRC) om besluiten
in vast te leggen. De motivatie hiervoor is dat besluiten een bestaansrecht
hebben onafhankelijk van zaken: niet elk besluit ontstaat gedurende de
uitvoering van een zaak (denk aan raadsbesluiten).

Hierbij lopen we wat vooruit, maar volgen we wel data-bij-de-bron principes uit
Common Ground.
BESLUITTYPE laten we vooralsnog in de ZTC. Nader bepaald moet gaan worden hoe we hiermee omgaan, zoals onderbrengen in de BRC, aparte type-componenten cq. api's e.d.

Dit betekent dat er voor BESLUIT een informatiemodel moet komen, ontrokken aan het
RGBZ. BESLUIT krijgt een optionele relatie naar ZAAK.

De verwachting is dat er later vergelijkbare designkeuzes gemaakt worden voor
andere resources, zoals bijvoorbeeld klantcontacten.


## Response bij input validatie

De DSO schrijft een structuur voor waar een fout-response aan moet voldoen
in het geval van validatiefouten:

```json
{
    "type": "...",
    "title": "...",
    "status": 400,
    "detail": "...",
    "instance": "...",
    "invalid-params": [{
        "type": "...",
        "name": "<veldnaam>",
        "reason": "Maximale lengte overschreden.",
    }]
}
```

Op het [forum](https://forum.pdok.nl/t/formaat-foutafhandeling-input-validatie-api-50/1848)
is nagevraagd hoe er moet omgegaan worden met meerdere fouten op hetzelfde veld,
met de conclusie dat dit beter gespecifieerd moet worden.

In ZDS 2.0 kiezen we ervoor om elke fout op een veld als apart object op te
nemen binnen de `"invalid-params"` sleutel. Dit laat clients toe om elke
individuele fout te renderen zoals zij wensen.

Tevens voegen we op hoofdniveau van fouten en binnen een object in
`"invalid-params"` een sleutel `"code"` toe. Dit veld bevat een waarde die door
machines kan begrepen worden, bijvoorbeeld `"invalid"` of
`"max_length_exceeded"`. De `"type"` sleutel is namelijk optioneel en bedoeld
voor developers om meer informatie over het fouttype te lezen, en niet voor
automatische verwerking.

Een voorbeeld response is dan:

```json
{
    "type": "URI: https://ref.tst.vng.cloud/ref/fouten/ValidationError/",
    "title": "Ongeldige gegevens",
    "status": 400,
    "code": "invalid",
    "detail": "Er deden zich validatiefouten voor",
    "instance": "urn:uuid:2d3f8adb-470d-4bf4-8c43-4b2ebeef7504",
    "invalid-params": [
        {
            "name": "identificatie",
            "code": "max-length",
            "reason": "Maximale lengte overschreden.",
        },
            {
            "name": "identificatie",
            "code": "special-characters",
            "reason": "De identificatie mag geen speciale karakters bevatten.",
        }
    ]
}
```

Merk op dat de concrete codes hier fictief zijn en puur ter illustratie zijn.


## Many-to-many relaties verspreid over API's

Deze beslissing komt voort uit
[issue #166](https://github.com/VNG-Realisatie/gemma-zaken/issues/166).

Tussen twee componenten A en B (met component A de component waar in het IM de
relatie naar component B loopt), wordt de relatie in beide componenten
bewaard. Eventuele metagegevens (extra informatie op de relatie) komt in
component A te liggen, en component B bevat _enkel_ de relatieinformatie zelf.

Dit is een technische oplossing zodat beide componenten de relaties kunnen
opvragen zonder alle (mogelijks honderden) componenten af te moeten lopen om
te vragen of ze een relatie hebben.

Een consumer maakt 1x een relatie aan, tussen component A en B, met
metagegevens over de relatie in component A. Component A is vervolgens
verantwoordelijk om de relatie in component B aan te maken.

We stemmen de `objectTypes` af zodat die mappen op de namen van resources om
generieke synchronisatieimplementaties mogelijk te maken.

Component B moet dan aan de volgende spelregels voldoen:

- de naam van de resource is `{resourceB}{resourceA}`, in component B
- de resource wordt genest ontsloten binnen `ResourceB`
- de resource accepteert een URL-veld met de naam `resourceA`

Deze spelregels en interactie worden actief getest in de
[integratietests](https://github.com/vng-Realisatie/gemma-zaken-test-integratie)
om compliancy af te kunnen dwingen.

Een concreet voorbeeld hiervan is een `INFORMATIEOBJECT` in het DRC en een
`ZAAK` in het ZRC:

1. De consumer maakt in het DRC een relatie (met polymorfe relatieinformatie):

    ```http
    POST https://drc.nl/api/v1/objectinformatieobjecten

    {
        "informatieobject": "https://drc.nl/api/v1/enkelvoudigeinformatieobjecten/1234",
        "object": "https://zrc.nl/api/v1/zaken/456789",
        "objectType": "zaak",
        "titel": "",
        "beschrijving": "",
        "registratiedatum": "2018-09-19T17:57:08+0200"
    }
    ```

2. Het DRC doet vervolgens een request naar het ZRC (op basis van de URL van `object`):

    ```http
    POST https://zrc.nl/api/v1/zaken/456789/zaakinformatieobjecten

    {
       "informatieobject": "https://drc.nl/api/v1/enkelvoudigeinformatieobjecten/1234",
    }
    ```
