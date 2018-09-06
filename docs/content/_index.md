---
title: "Introductie"
description: ""
weight: 10
---

GEMMA Zaken is opgedeeld in verschillende componenten.

Alle componenten zijn noodzakelijk om zaakgericht werken mogelijk te maken. Echter, niet alle componenten hoeven aanwezig te zijn in elk software pakket. Sterker nog, het moet in te stellen zijn waar elk component "leeft". Zo kan een gemeente een software pakket afnemen dat zowel een ZRC, ZTC en DRC bevat maar als ze overstappen op een ander DRC moet dit eenvoudig te configureren zijn in het software pakket. Het kan dus ook zo zijn dat het ZRC, ZTC en DRC door verschillende leveranciers wordt geleverd, of dat er gebruik wordt gemaakt van een SaaS-oplossing die bijvoorbeeld een ZTC component aanbiedt.

## Zaakregistratiecomponent (ZRC)

Component voor opslag en ontsluiting van zaakgegevens - dit vormt de spil
van zaakgericht werken. Zie de [ZRC documentatie]({{< relref "zrc.md" >}}).

## Zaaktypecatalogus (ZTC)

Component voor opslag en ontsluiting van zaaktypegegevens - hier worden de
mogelijke zaaktypen geconfigureerd, wat een weerslag heeft op afgeleide
gegevens in andere componenten. Zie de
[ZTC documentatie]({{< relref "ztc.md" >}}).

## Documentregistratiecomponent (DRC)

Component voor opslag en ontsluiting van informatieobjecten en daarbij
behorende metadata. Informatieobjecten zijn documenten/bijlagen met
meta-informatie die bij zaken kunnen horen. Deze component voorziet in het
ontsluiten hiervan. Zie de [DRC documentatie]({{< relref "drc.md" >}}).

## Besluitregistratiecomponent (BRC)

Uitkomsten van zaken worden in besluiten vastgelegd. Daarnaast is er het
concept van niet-zaakgerelateerde besluiten (out of scope). In deze component
worden besluiten geregistreerd, die aan zaken kunnen gerelateerd worden.
Zie de [BRC documentatie]({{< relref "brc.md" >}}).
