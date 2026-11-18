# Portefeuille Tool

Een tool in één html-bestand voor het taxeren van vastgoedportefeuilles (Savills). Je sleept er de
Excel-export van de query in (de tabellen `tblQuery`, één regel per object, en `tblVHE`, de verhuurbare
eenheden) of losse rekenmodellen (.xlsm, tabblad `Overview`). De tool controleert de data, toont
overzicht, kaart, objecten en VHE's, en maakt een rapport (PDF via afdrukken in de browser) en een
Excel-bestand. Alles gebeurt in de browser, zonder internet.

## Wat er in deze map staat

```
index.html    de tool, met de handleiding erin (huidige versie: 1.25, 18 november 2026)
CLAUDE.md     dit bestand
```

- Meer is er niet. De oorspronkelijke bron (`src/`, `vendor/`, `build.py`, `tests/`, `handleiding/`,
  `hulpscripts/`) staat **niet** in deze map en ook niet in git. Er is dus geen bouwstap: wijzigingen
  gebeuren rechtstreeks in `index.html`.
- `index.html` is het bestand dat Netlify toont en het enige dat geleverd wordt.
- Een aparte voorbeeldversie of een los handleidingbestand is er niet meer: maak die niet opnieuw. Om
  te oefenen heeft de tool zelf voorbeelddata (knop Voorbeelddata op het datascherm).

## Werkafspraken

- Alles in het Nederlands: de tool, namen in de code, commentaar, versieteksten en je antwoorden.
  Het rapport kan ook in het Engels (`T("nl", "en")`).
- Elke levering is een nieuwe versie: een regel bovenaan de lijst `const VERSIES = [...]` in `index.html`
  (zie Versies), en het versienummer en de datum in de handleiding (zie Handleiding).
- Na elke wijziging de tool in de browser openen en controleren, op 375 en op 1280 px breed, met de
  voorbeelddata, zonder JavaScript-fouten. Pas daarna zeggen dat het af is.
- Bestanden niet hernoemen zonder te vragen.

## Harde regels

- **Eén bestand, zonder internet.** Geen CDN, geen externe lettertypes of scripts (cdn.jsdelivr.net,
  fonts.googleapis.com, unpkg.com en dergelijke). Alles wat nodig is, zit al in `index.html`
  (xlsx-js-style 1.2.0, lettertypes van het rapport, kaartdata, omslagbeelden).
- **Data blijft in de browser.** Nooit data naar een server, geen statistieken of telemetrie. Alleen de kaart
  vraagt iets op bij PDOK (Kadaster): tegels als de gebruiker Topografisch kiest, en adressen (alleen straat,
  huisnummer en plaats) via de schakelaar in het venster ‘Wat wil je zien?’ of de knop op de kaart. Die schakelaar
  staat op verzoek van de eigenaar standaard aan, maar altijd zichtbaar en met uitleg voordat er iets wordt
  verstuurd; nooit adressen opzoeken zonder dat de gebruiker die schakelaar of knop heeft gezien.
- **Geen gevoelige gegevens naar buiten, ook niet naar jou.** Open of lees geen echte klantbestanden
  (Excel-exports, rekenmodellen, rapporten) tenzij de gebruiker daar uitdrukkelijk om vraagt. Test met de
  voorbeelddata. Bij een probleem met echte data: vraag om de kopie van het logboek (knop Kopiëren op het
  datascherm), die bevat geen gevoelige gegevens.
- **Het logboek bevat nooit** bestandsnamen, adressen, bedragen of celwaarden (zie Logboek).
- **Alleen fictieve data in voorbeelden.** Schermafbeeldingen, de handleiding en alles wat gedeeld kan
  worden gebruiken de voorbeelddata van de tool.
- **Rapporten, PDF's en pagina's in Savills-opmaak niet online publiceren**; alleen als bestand.
- Huisstijl: navy `#25273A`, geel `#FFDF00`, cream `#EEE8E3`. De sjablonen volgen de Savills Brand
  Guidelines (januari 2024). Grafiekkleuren zijn gecontroleerd op kleurenblindheid; tekst staat in inkt,
  nooit in de kleur van een reeks.

## Valkuilen bij het lezen

- `index.html` is ruim 5 MB, met lange regels vol data (lettertypes, kaartdata, xlsx-bibliotheek en
  onderaan de handleiding met zijn afbeeldingen). Lees het nooit in zijn geheel; zoek gericht (op
  functienaam of tekst) en lees alleen het stuk eromheen.
- Een losse `</script>` in de code breekt het bestand: schrijf `<\/script>` in een string.
- Windows: lees en schrijf in scripts altijd met `encoding="utf-8"`.

## Hoe de code in elkaar zit

`index.html` is ooit gebouwd uit losse onderdelen; die volgorde zit er nog in. De lijst `BRONREGELS`
(zoek op `BRONREGELS =`) noemt per onderdeel de regel waar het begint:

| Onderdeel | Wat |
|---|---|
| kop, `10-basis.css`, `11-tool.css`, `20-body.html` | kop, opmaak, de schermen |
| `30-kern.js` | helpers (`$`, `$$`, `esc`, `meervoud`), opslag (`S`, `laadS`, `bewaar`), `naar`, `melding`, `bevestig`, bezig-venster, `VERSIES` |
| `30a-logboek.js`, `46-logboek.js` | logboek: regels schrijven; het paneel op het datascherm |
| `31a-kolomnamen.js`, `31-velden.js` | kolomnamen (nieuw en oud), de velden (`VELD`), `standaardS()` |
| `32-data.js` | inlezen, herkennen, koppelen en controleren: `OBJ`, `VHE`, `DATA`; de voorbeelddata |
| `33-reken.js` | opmaken (`fmt`), rekenen, `selectie()` |
| `34-grafiek.js` | SVG-grafieken, voor het scherm én het rapport |
| `35-geo.js` | kaartdata (gegenereerd; niet met de hand wijzigen) |
| `36-locatie.js`, `37-kaart.js` | ligging van objecten; de kaart (`kaartStatisch`) |
| `38-analyse.js` | waardebrug en gevoeligheid |
| `39-rekenmodellen.js` | rekenmodellen (.xlsm) lezen |
| `40`–`45` | de schermen: data, overzicht, objecten, VHE's, objectvenster, kaart |
| `47-snel-rapport.js` | Snel rapport op het datascherm |
| `48-sjablonen.js`, `49-staand.js` | rapportsjablonen; de staande sjablonen |
| `50-rapport.js`, `51-rapport-grafieken.js` | rapportstudio en het rapport zelf |
| `60-export.js` | Excel-export |
| `90-start.js` | opstarten |

- Gewone JavaScript, geen framework en geen modules: alles staat in één `<script>` met `"use strict"`,
  functies en constanten zijn globaal.
- `S`: instellingen en keuzes, bewaard in localStorage (`OPSLAG + "_v1"`). Een nieuwe instelling krijgt
  een standaardwaarde in `standaardS()`; na een wijziging `bewaar()`. Ruwe data staat in IndexedDB,
  het logboek in sessionStorage.
- `OBJ`: de objecten. `o.v` zijn de huidige waarden per veld (`o.v.MV_Total`), `o.p` die van de vorige
  taxatie, `o.loc` de ligging. `VHE`: de verhuurbare eenheden.
- `selectie()`: de objecten na filters en uitgezette objecten. Overzicht, kaart, lijsten, rapport en
  export gaan allemaal hiervan uit.
- Totalen: `aggr(objs, veld)` (opgeteld of gewogen, zie `tot` in `VELDEN`). Het verschil in een totaal
  (like-for-like) kiest de gebruiker bij ‘Verschil’ naast Backtesting (`S.kolommen.verschil`): `totaal`
  (`aggrDelta`, het verschil van het totaal of gemiddelde) of `object` (`gemDelta`, het gemiddelde van de
  verschillen per object). Toon een verschil in een totaal altijd via `aggrVerschil` (met `verschilAantal`),
  ook in rapport en Excel, en zeg in de tekst welke manier het is (`verschilPerObject()`). De waardebrug
  rekent altijd met totalen. Uitleg bij een totaal op het scherm: `totaalTip(objs, veld)` als `data-tip`.
- Schermen: `naar("data" | "overzicht" | "kaart" | "objecten" | "vhe" | "rapport")`.
- HTML met template strings; data altijd door `esc()`. Getallen via `fmt(waarde, soort, taal)` met soort
  `euro`, `pct`, `m2`, `aantal`, `factor` of `datum`.
- Meldingen: `melding(tekst, "goed" | "mis")`; een vraag: `await bevestig(titel, uitleg, knop)`.
- Weergave (`S.weergave`, knop `#knopWeergave` in de kop, `pasWeergaveToe()` zet `data-modus` en `data-thema` op `<html>`):
  - Modus `eenvoudig` (alleen bekijken; standaard voor wie de tool voor het eerst opent) of `uitgebreid` (alles). Wat bij
    één modus hoort, krijgt de klasse `alleen-uitgebreid` of `alleen-eenvoudig`; in de code vraag je `eenvoudig()`.
    Een nieuwe knop voor rapport, instellingen of koppelen: geef hem `alleen-uitgebreid`.
  - Thema `licht`, `donker` of `auto`. De donkere kleuren staan in één blok aan het eind van de opmaak (zoek op
    `weergave: modus en donker thema`): eerst de variabelen, daarna de regels voor vaste kleuren. Een nieuwe vaste kleur
    in de opmaak of in een SVG op het scherm: zet daar de donkere tegenhanger bij en bekijk het scherm in donker.
    Grafieken en kaart worden daar met `[fill="#…"]` omgezet; het rapport en de handleiding blijven altijd licht.
- Lang werk gaat in stappen met het bezig-venster (`bezigStart`, `bezigStap`, `bezigKlaar`). De tool moet
  vlot blijven bij 1.000+ objecten en honderden rekenmodellen.
- Data laden (`leesBestanden`): nieuwe bestanden komen bij wat er al staat (`voegBij`: regels onder elkaar,
  kolommen op naam of veld; hetzelfde dossiernummer, of bij VHE's hetzelfde bestand, wordt vervangen door de
  nieuwe regels). Alleen ‘Ander bestand, alleen voor …’ (optie `alleen`) vervangt een deel; voorbeelddata gaat
  altijd weg. Rekenmodellen komen bij rekenmodellen; een query en rekenmodellen gaan niet samen (`kanErbij`).
- Naar het overzicht (`gaNaarOverzicht`): de eerste keer per portefeuille het venster ‘Wat wil je zien?’
  (`openOverzichtKeuze`, `S.ovKeuzeGedaan`; de keuze staat in `S.overzicht.kaarten`, zie `OV_KEUZES` en
  `ovKaartAan`). Staat `S.kaart.auto` aan, dan zoekt `zoekAdressenMetBalk()` eerst de adressen op, met het
  bezig-venster als voortgangsbalk. Een nieuw onderdeel in het overzicht: ook een regel in `OV_KAARTEN`,
  `OV_KEUZES` en `ovKaartKan`.
- Query: de tool zoekt de Excel-tabellen `tblQuery` en `tblVHE` en kent nieuwe en oude kolomnamen;
  kolommen met `_Prev` en `_Delta` horen bij hun veld.
- Rekenmodellen: alleen het tabblad Overview wordt uitgepakt: regel 1 de kolomnamen, regel 2 de waarden;
  de VHE's in de kolommen A–D, kop op regel 34, eenheden op 35–209.

## Adressen op de kaart

- `plaatsObject(o)` kiest de plek: zelf gekozen, coördinaten, adres (PDOK, alleen op verzoek), postcode, plaats, VHE's.
- Adressen in de query zijn vaak opsommingen (‘Dorpsstraat 13A-01, 13A-02’, ‘Kade 5 - 57 odd’, twee straten).
  `adresDelen()` haalt er per straat het eerste huisnummer uit; `pdokZoekAdres()` neemt alleen een antwoord aan in
  dezelfde straat (`straatGelijk`) en in of bij dezelfde plaats (`pdokPlaatsPast`), anders de straat zelf (bron
  `straat`). Zoek nooit met de hele adrestekst: dan geeft PDOK een andere straat terug.
- Verandert de manier van zoeken, verhoog dan `ADRES_VERSIE`: wat eerder niet of twijfelachtig is gevonden, wordt dan
  opnieuw opgezocht (`adresTeZoeken`).
- Testen tegen PDOK alleen met verzonnen adressen op openbare straten of de voorbeelddata, nooit met adressen van een klant.

## Het rapport

- `bouwRapport(opties)` maakt een compleet html-document met een eigen script. Dat verdeelt de inhoud
  over bladen (`.blad`), zet getallen in rondjes in het midden (`centreerGetallen`), zet
  `window.__klaar = true` en opent het afdrukvenster (niet met `voorbeeld: true`: dat is het voorbeeld
  rechts in de rapportstudio, in het frame `#pdfFrame`).
- Dat script staat in een template literal: schrijf `\\s` voor `\s` in een reguliere expressie,
  `<\/script>`, en geen `${` tenzij je het bedoelt. Het is geschreven in ES5-stijl (`var`, `function`).
- Sjablonen: klassiek, modern, geel, navy, dashboard en research, en voor staand bijlage, factsheet,
  magazine en compact. Een sjabloon is opmaak; inhoud, rekenwerk en bladverdeling zijn gedeeld.
- De bladverdeling rekent met gemeten maten (`SJ_MATEN`). Verander je de hoogte van rijen of koppen,
  controleer dan het aantal bladen in alle sjablonen.
- **Getallen in rondjes altijd als SVG**: rondje en `<text class="rnr" data-m="cx cy">` samen in één
  `<svg>`, nooit een html-achtergrond met tekst erin (anders staat het getal scheef in Chrome). In de tool
  rekent `getalPlek()`, in het rapport `centreerGetallen()`.
- Afdrukken: Chrome of Edge, ‘Opslaan als PDF’, marges op ‘Geen’.

## Logboek

- Regels: `logInfo`, `logOk`, `logLet` en `logFout(categorie, tekst, { sub, b, kern })`.
- Schrijf de tekst met de tag `LT` en de vervangers: `anB(bestandsnaam)` geeft B1.xlsx, `anM(map)` M1,
  `anO(object)` O1, `anN(naam)` alleen gewone kolom- of tabbladnamen, `anW(waarde)` de soort (getal,
  tekst, leeg, #VALUE!) en `anV(tekst)` tekst van de tool zelf. Voorbeeld:
  ``logOk("data", LT`${anB(f.name)}: ${n} kolommen`)``.
- `anVeilig` is het laatste vangnet, geen vervanging voor het bovenstaande.
- Fouten in de code komen er vanzelf in, met de plek als onderdeel en regel (`39-rekenmodellen.js:285`),
  berekend met `BRONREGELS`. Voeg je in `index.html` regels toe of haal je ze weg, dan verschuiven die
  plekken: werk dan de getallen in `BRONREGELS` na het gewijzigde onderdeel bij.

## Handleiding

- De handleiding zit in `index.html`, helemaal onderaan, na het hoofdscript: een blok
  `<script type="text/plain" id="handleidingBron">` met het complete html-document van de handleiding
  als tekst. In die tekst staat `<\/script` voor `</script` en `<\!--` voor `<!--`, anders breekt het
  bestand.
- `openHandleiding(hoofdstuk)` (bij "over deze tool" in `30-kern.js`) zet die tekst pas bij het openen in
  een `<iframe srcdoc>` in een venster over het hele scherm. Zonder hoofdstuk opent hij bij het scherm
  waar je bent (`HL_HOOFDSTUK`; op het datascherm `laden` of, met data, `controle`). De knop is
  `#knopHandleiding` (?) in de kop; ‘Over deze tool’ heeft ook een knop.
- Links `#…` binnen de handleiding vangt `openHandleiding` zelf af: in een srcdoc-kader zou de link
  anders `index.html` in het kader laden. Esc sluit eerst een vergrote afbeelding, daarna het venster.
- De hoofdstukken (id's van de `<section>`s): start, laden, controle, overzicht, kaart, objecten, vhe,
  rapport, excel, rekenen, herkennen, problemen, toetsen.
- Tekst aanpassen: zoek de zin in het blok en wijzig hem daar. Afbeeldingen zijn WebP als `data:`-adres
  (maximaal 1300 px breed, kwaliteit 62). Een nieuwe schermafbeelding: met de voorbeelddata.
- Het versienummer staat twee keer in de handleiding: `<span class="versie">versie …</span>` in de kop en
  `handleiding bij versie … · datum` in de voet.

## Versies

Een nieuwe versie bovenaan `const VERSIES = [...]` in `index.html` (als datum de dag van de levering):

```js
{"nr":"1.22","datum":"2026-11-10","nieuw":["Onderdeel: wat er voor de gebruiker verandert, in gewone zinnen"]},
```

De tool laat de wijzigingen zien onder ‘Over deze tool’.

## Testen

- Er is geen testscript meer in deze map. Controleer met de browser (bijvoorbeeld Playwright voor Python:
  `pip install playwright` en `python -m playwright install chromium`).
- Open `index.html` als `file://`, klik op `#knopVoorbeeld` (of `await laadVoorbeeld()`) en wacht op
  `OBJ.length === 20`. Blokkeer het netwerk met `ctx.route(re.compile(r"^https?://"), ...)`.
- Handig in de pagina: `naar("kaart")`, `S.pagina.sjabloon = "geel"`, `S.pagina.taal = "en"`,
  `bouwRapport({ voorbeeld: true })` en dat met `page.set_content(html)` openen en wachten op
  `window.__klaar === true`; `maakExcel()` geeft `{ data, naam }`; `logKopieTekst(LOGBOEK.regels)` is de
  kopie van het logboek (controleer dat er geen bestandsnamen of bedragen in staan).
- Bekijk zichtbare wijzigingen echt: schermafbeeldingen op 375 en 1280 px breed, ook van het rapport, en
  in beide talen. Ook in het donkere thema en in de eenvoudige modus
  (`S.weergave.thema = "donker"; S.weergave.modus = "eenvoudig"; pasWeergaveToe(); toonScherm(UI.stap)`).
  Een nieuwe browser begint in de eenvoudige modus: zet in een test eerst `S.weergave.modus = "uitgebreid"`.
