# Formatspezifikation: Modelfile für das „Interaktive Beziehungsmodellierungs-Tool"

Dieses Dokument beschreibt vollständig das JSON-Format, das eine Modelfile-Datei für das Tool
„Interaktives Beziehungsmodellierungs-Tool" (objekt-beziehungsmodell.html) haben muss, damit sie
über den Button **„Öffnen" → „📂 Datei vom PC wählen"** bzw. **„⬆ Datei öffnen"** fehlerfrei geladen
werden kann. Es richtet sich an eine KI (oder einen Menschen), die eine neue Modelfile von Grund
auf erzeugen soll, ohne das Tool selbst zu kennen.

---

## 1. Grundprinzip

Ein Modelfile beschreibt ein **Objekt-Beziehungsmodell**: eine Menge von **Objekten (Nodes)**, die
über **Beziehungen (Edges)** miteinander verbunden sind. Jedes Objekt gehört zu genau einer
**Klasse** (z. B. „Person", „Stadt", „Unternehmen") — die Klasse bestimmt Icon, Farbe und die
sinnvollen Attribut-Felder des Objekts. Alle gültigen Klassen sind in Abschnitt 4 aufgelistet.

**Wichtigste Regel:** Nur die in Abschnitt 4 gelisteten Klassen-Keys werden vom Tool korrekt
dargestellt (mit Icon, Farbe und Formularfeldern). Verwendet man einen Klassen-Key, der dort nicht
existiert, setzt das Tool die Klasse beim Laden automatisch und stillschweigend auf `neutral`
zurück — die Objektdaten bleiben zwar erhalten, aber Icon/Farbe/Feldbezeichnungen gehen verloren.

---

## 2. Oberste Ebene der Datei

```json
{
  "format": "objekt-beziehungsmodell-v1",
  "name": "Name des Modells",
  "nodes": [ /* Array von Node-Objekten, siehe Abschnitt 3.1 */ ],
  "edges": [ /* Array von Edge-Objekten, siehe Abschnitt 3.2 */ ],
  "exportedAt": 0
}
```

| Feld | Typ | Pflicht | Beschreibung |
|---|---|---|---|
| `format` | String | ja | Immer exakt `"objekt-beziehungsmodell-v1"` |
| `name` | String | ja | Anzeigename des Modells (erscheint oben links im Tool) |
| `nodes` | Array | ja | Liste aller Objekte (siehe 3.1). Darf leer sein, aber sollte für ein sinnvolles Modell mind. 2 Einträge haben |
| `edges` | Array | ja | Liste aller Beziehungen (siehe 3.2). Darf leer sein |
| `exportedAt` | Zahl | nein | Zeitstempel, wird vom Tool nicht zur Validierung genutzt; `0` oder aktueller Unix-Timestamp ist ausreichend |

---

## 3. Objekt-Strukturen

### 3.1 Node (Objekt)

```json
{
  "id": "obj_a1b2c3d",
  "class": "person",
  "name": "Anzeigename des Objekts",
  "params": {
    "feldkey1": { "de": "Wert auf Deutsch", "en": "Value in English" },
    "feldkey2": { "de": "Wert 2", "en": "Value 2" }
  },
  "group": "A",
  "pos2d": { "x": 0, "y": 0 },
  "pos3d": { "x": 0, "y": 0, "z": 0 },
  "pinned2d": false,
  "pinned3d": false,
  "lat": "48.2082",
  "lon": "16.3738",
  "images": [ "data:image/jpeg;base64,/9j/4AAQSkZJRg..." ],
  "createdAt": "2026-09-18T15:09"
}
```

| Feld | Typ | Pflicht | Beschreibung |
|---|---|---|---|
| `id` | String | **ja, muss eindeutig sein** | Frei wählbare eindeutige ID. Empfohlenes Muster: `obj_` + 7 zufällige alphanumerische Zeichen, z. B. `obj_x7f2k9a`. Zwei Nodes dürfen niemals dieselbe `id` haben |
| `class` | String | ja | Muss einer der Klassen-Keys aus Abschnitt 4 sein (sonst Reset auf `neutral`, siehe oben). `neutral` selbst ist ebenfalls ein gültiger, "leerer" Klassenwert |
| `name` | String | ja | Anzeigename/Titel des Objekts, erscheint als Label am Knoten |
| `params` | Objekt | ja (kann `{}` sein) | Attributwerte des Objekts. Jeder Schlüssel muss einem `key` aus den `fields` der gewählten Klasse entsprechen (siehe Abschnitt 4). Nicht benötigte Felder können weggelassen werden – leere/irrelevante Felder nicht mit Platzhaltern füllen, einfach weglassen |
| `group` | String | ja | Frei wählbarer interner Gruppierungs-Buchstabe für Fusions-/Kopplungsfunktionen des Tools. Wenn keine Fusion geplant ist, immer `"A"` verwenden |
| `pos2d` | `{x,y}` | ja | Position im 2D-Diagramm. Zahlen, beliebige Einheit (Pixel-artig). Siehe Hinweise zu Layout in Abschnitt 5 |
| `pos3d` | `{x,y,z}` | ja | Position im 3D-Diagramm. Kann identisch zu `pos2d` mit `z:0` gesetzt werden |
| `pinned2d` / `pinned3d` | Boolean | ja | Ob die Position beim automatischen Neuanordnen fixiert bleibt. Für neu erzeugte Dateien immer `false` |
| `lat` / `lon` | String | nein | Breiten-/Längengrad als Dezimalgrad-Zahl in String-Form, z. B. `"48.2082"`. Nur sinnvoll bei real verorteten Objekten. Aktiviert im Tool die Anzeige als Stecknadel auf einer Karte (2D) bzw. einem Globus (3D) im „GEO"-Modus. Fehlt eines der beiden Felder oder ist der Wert ungültig, gilt das Objekt als nicht georeferenziert |
| `images` | Array von Strings | nein | Liste vollständiger Bild-Daten-URIs (`"data:image/jpeg;base64,..."`), die im Inspektor als Galerie angezeigt werden. Kann fehlen oder ein leeres Array sein. Bilder sollten vor dem Einbetten auf eine vernünftige Kantenlänge (z. B. ≤ 900 px) verkleinert werden, um die Datei nicht unnötig aufzublähen. Nur von Browsern direkt darstellbare Formate verwenden (JPEG, PNG, WebP, GIF) — **kein HEIC** und keine anderen Formate, die `<img>` nicht nativ anzeigen kann; das Tool selbst weist solche Formate beim Hochladen zurück, eine von Hand erzeugte Datei sollte dieselbe Regel einhalten |
| `createdAt` | String | nein | Erstellungszeitpunkt im Format `"YYYY-MM-DDTHH:mm"` (identisch zum HTML-Feldtyp `datetime-local`), z. B. `"2026-09-18T15:09"`. Wird vom Tool beim Erzeugen eines Objekts automatisch gesetzt, kann von einer KI aber auch selbst vorbelegt werden |

**WICHTIG zu `params`:** Jeder Feldwert ist **zweisprachig** und MUSS die Form
`{ "de": "...", "en": "..." }` haben — niemals ein reiner String. Wenn keine Übersetzung
vorliegt, denselben Text für `de` und `en` eintragen. Ein Feld einfach als
`"feldkey": "Text"` zu setzen (ohne `de`/`en`-Objekt) führt dazu, dass das Tool das Feld nicht
korrekt anzeigt.

**Universelles Feld `bemerkung`:** Zusätzlich zu den klassenspezifischen Feldern aus Abschnitt 4
zeigt das Tool bei **jeder** Klasse (außer `neutral`) automatisch ein weiteres Feld namens
„Bemerkung" (`bemerkung`) an — unabhängig davon, ob es in der Feldliste der Klasse aufgeführt ist.
Es dient für Kommentare oder Informationen, die sich keinem der vordefinierten Felder zuordnen
lassen, und wird beim **Fusionieren** zweier Modelle automatisch genutzt, um bei gleichnamigen
Objekten derselben Klasse widersprüchliche Werte zu protokollieren (siehe Abschnitt 9). Eine KI,
die ein neues Modelfile erzeugt, kann dieses Feld optional befüllen, muss es aber nicht angeben —
fehlt es, wird es beim Laden automatisch als leer initialisiert. Anders als `bemerkung` (in
`params`, zweisprachig) liegen `lat`, `lon`, `images` und `createdAt` direkt auf der Node-Ebene
und sind **nicht** zweisprachig — es handelt sich um reine Werte (Koordinate, Bilddaten,
Zeitstempel), keine übersetzbaren Texte.

### 3.2 Edge (Beziehung)

```json
{
  "id": "edge_p9q8r7s",
  "from": "obj_a1b2c3d",
  "to": "obj_e4f5g6h",
  "rolle": { "de": "ist Mitarbeiter von", "en": "is an employee of" },
  "kontakt": { "de": "Zusatzinfo/Kommentar zur Beziehung", "en": "Additional note on the relationship" },
  "flow": "forward",
  "showArrow": true
}
```

| Feld | Typ | Pflicht | Beschreibung |
|---|---|---|---|
| `id` | String | **ja, muss eindeutig sein** | Analog zu Node-`id`, empfohlenes Muster `edge_` + 7 Zeichen |
| `from` | String | ja | Muss der `id` eines existierenden Node in `nodes` entsprechen |
| `to` | String | ja | Muss der `id` eines existierenden Node in `nodes` entsprechen (darf nicht gleich `from` sein) |
| `rolle` | `{de,en}` | ja | Beschreibt die Art der Beziehung ("Rolle"), z. B. „ist Mitarbeiter von", „gehört zu", „liegt in". Frei wählbarer Text, keine feste Werteliste |
| `kontakt` | `{de,en}` | ja (kann leere Strings enthalten) | Freitextfeld für Zusatzinformationen/Begründung/Kommentar zur Beziehung. Kann `{ "de": "", "en": "" }` sein, wenn nichts zu ergänzen ist |
| `flow` | String | nein | Wirkrichtung der Beziehung: `"forward"` = wirkt von `from` auf `to` (Standard, wenn das Feld fehlt), `"backward"` = wirkt von `to` auf `from`. Das Tool zeigt die Richtung als Pfeil auf der Linie und baut den Modell Report danach auf |
| `showArrow` | Boolean | nein | `false` blendet den Pfeil aus; die Beziehung gilt dann als richtungslos (`flow` wird ignoriert). Fehlt das Feld, wird der Pfeil angezeigt |

**Zur Wirkrichtung:** `rolle` wird immer in Leserichtung `from` → `to` formuliert, unabhängig von
`flow`. Soll eine Beziehung entgegen dieser Leserichtung wirken, genügt `"flow": "backward"` — die
Rolle muss nicht umformuliert werden. Das Tool setzt beide Felder nur, wenn sie vom Standard
abweichen; ältere Dateien ohne diese Felder bleiben gültig (alle Beziehungen wirken dann von
`from` nach `to`).

**WICHTIG:** Jede Edge muss auf zwei tatsächlich existierende Node-`id`-Werte verweisen. Eine
Edge mit einer `from`- oder `to`-ID, die zu keinem Node passt, wird vom Tool nicht korrekt
dargestellt (führt zu Darstellungsfehlern/verwaisten Kanten).

---

## 4. Vollständige Liste der gültigen Klassen

Jede Klasse gehört zu genau einer Gruppe und hat eine feste Liste von `fields` (Feld-Keys), die
für `params` verwendet werden können. Es müssen nicht alle Felder befüllt werden. Zusätzlich zeigt
das Tool bei jeder Klasse (außer `neutral`) automatisch die universellen Felder `bemerkung`
(in `params`) sowie `lat`, `lon`, `images` und `createdAt` (auf Node-Ebene) an, siehe Abschnitt 3.1.

| Klassen-Key | Bezeichnung | Gruppe | Feld-Keys (params) |
|---|---|---|---|
| `behaelter` | Behälter | Apparate | `volumen`, `werkstoff`, `betriebsdruck`, `betriebstemperatur`, `medium` |
| `filter` | Filter | Apparate | `filtertyp`, `maschenweite`, `differenzdruck`, `wechselintervall`, `medium` |
| `pumpe` | Pumpe | Apparate | `foerdermedium`, `foerderleistung`, `foerderhoehe`, `antriebsart`, `werkstoff` |
| `waermetauscher` | Wärmetauscher | Apparate | `bauart`, `waermeleistung`, `mediumprimaer`, `mediumsekundaer`, `flaeche` |
| `anstellung` | Anstellungsverhältnis | Arbeit | `arbeitgeber`, `beschaeftigungsart`, `beginn`, `befristung`, `kontakt` |
| `ausbildungsberuf` | Ausbildungsberuf | Ausbildung | `bezeichnung`, `dauer`, `ausbildungsbetrieb`, `abschluss`, `kontakt` |
| `lehrgang` | Lehrgang / Kurs | Ausbildung | `bezeichnung`, `dauer`, `anbieter`, `abschluss`, `kontakt` |
| `autohersteller` | Autohersteller (Konzern) | Automobilindustrie | `land`, `hauptsitz`, `gruendungsjahr`, `kontakt`, `status` |
| `automarke` | Automarke | Automobilindustrie | `land`, `hauptsitz`, `gruendungsjahr`, `kontakt`, `status` |
| `automodell` | Automodell | Automobilindustrie | `antriebsart`, `segment`, `marktstart`, `kontakt` |
| `bedienungsanleitung` | Bedienungsanleitung | Dokumente | `dokumentnummer`, `version`, `hersteller`, `sprache`, `ablageort` |
| `pid` | P&ID | Dokumente | `dokumentnummer`, `version`, `ersteller`, `freigabedatum`, `ablageort` |
| `stromlaufplan` | Stromlaufplan | Dokumente | `dokumentnummer`, `version`, `ersteller`, `freigabedatum`, `ablageort` |
| `aktoren` | Aktoren | EMR-Ausrüstung | `funktion`, `stellbereich`, `ansteuersignal`, `stellzeit`, `einbauort` |
| `elektrischerantrieb` | Elektrischer Antrieb | EMR-Ausrüstung | `leistung`, `spannung`, `drehzahl`, `schutzart`, `hersteller` |
| `sensoren` | Sensoren | EMR-Ausrüstung | `messgroesse`, `messbereich`, `ausgangssignal`, `genauigkeit`, `einbauort` |
| `niederlassung` | Niederlassung | Firmen | `mutterunternehmen`, `adresse`, `funktion`, `leitung`, `kontakt` |
| `unternehmen` | Unternehmen | Firmen | `branche`, `rechtsform`, `adresse`, `geschaeftsfuehrung`, `kontakt` |
| `angelgeschaeft` | Angelgeschäft | Fischerei & Organisation | `adresse`, `oeffnungszeiten`, `telefon`, `sortiment`, `website` |
| `angelhotspot` | Angelhotspot | Fischerei & Organisation | `gewaesser`, `zielfische`, `methode`, `besteZeit`, `zugang` |
| `angellizenz` | Angellizenz / Fischerkarte | Fischerei & Organisation | `art`, `gueltigkeit`, `gewaesser`, `preis`, `inhaber` |
| `angelrevier` | Angelrevier / Gewässer | Fischerei & Organisation | `gewaessertyp`, `bewirtschafter`, `fischbestand`, `erlaubt`, `strecke` |
| `angelverein` | Angelverein / Fischereiverband | Fischerei & Organisation | `adresse`, `obmann`, `mitglieder`, `reviere`, `website` |
| `ausgabestelle` | Ausgabestelle (Lizenzen) | Fischerei & Organisation | `adresse`, `oeffnungszeiten`, `telefon`, `angebot`, `online` |
| `bestimmung` | Fischereibestimmung | Fischerei & Organisation | `art`, `fischart`, `wert`, `gebiet`, `quelle` |
| `fliegenrolle` | Fliegenrolle | Fliegenfischen | `klasse`, `bremse`, `fassung`, `gewicht`, `hersteller` |
| `fliegenrute` | Fliegenrute | Fliegenfischen | `klasse`, `laenge`, `aktion`, `teile`, `hersteller` |
| `fliegenschnur` | Fliegenschnur | Fliegenfischen | `klasse`, `profil`, `sinkrate`, `laenge`, `farbe` |
| `kunstfliege` | Kunstfliege | Fliegenfischen | `typ`, `muster`, `hakengroesse`, `zielfisch`, `saison` |
| `vorfach` | Vorfach / Tippet | Fliegenfischen | `staerke`, `laenge`, `material`, `tragkraft`, `verwendung` |
| `watausruestung` | Watausrüstung | Fliegenfischen | `art`, `groesse`, `material`, `hersteller`, `zustand` |
| `kamera` | Kamera | Fotografie | `sensorformat`, `hersteller`, `bajonett`, `marktstart`, `kontakt` |
| `objektiv` | Objektiv | Fotografie | `brennweite`, `lichtstaerke`, `gewicht`, `sensorformat`, `bildstabilisator` |
| `objektivtyp` | Objektivtyp | Fotografie | `beschreibung`, `brennweitenbereich`, `einsatzgebiet`, `kontakt` |
| `gewerbeimmobilie` | Gewerbeimmobilie | Gebäude | `nutzung`, `adresse`, `mietflaeche`, `baujahr`, `verwaltung` |
| `oeffentlichesgebaeude` | Öffentliches Gebäude | Gebäude | `nutzung`, `adresse`, `baujahr`, `flaeche`, `verantwortlich` |
| `abteilung` | Abteilung / Geschäftsgruppe | Gemeindeverwaltung | `kuerzel`, `zustaendigkeitsbereich`, `adresse`, `telefon`, `email` |
| `ansprechpartner` | Ansprechpartner/in (Verwaltung) | Gemeindeverwaltung | `funktion`, `titel`, `telefon`, `email`, `erreichbarkeit` |
| `kontinent` | Kontinent | Geographie | `flaeche`, `einwohnerzahl`, `anzahllaender`, `kontakt` |
| `land` | Land | Geographie | `hauptstadt`, `einwohnerzahl`, `flaeche`, `staatsform`, `kontakt` |
| `region` | Region | Geographie | `land`, `flaeche`, `einwohnerzahl`, `verwaltungssitz`, `kontakt` |
| `stadt` | Stadt | Geographie | `einwohnerzahl`, `flaeche`, `region`, `buergermeister`, `kontakt` |
| `arzt` | Arzt / Ärztin | Gesundheitsdienst | `fachrichtung`, `praxisadresse`, `kontakt`, `sprechzeiten`, `zulassungsnummer` |
| `krankenhaus` | Krankenhaus | Gesundheitsdienst | `traeger`, `adresse`, `fachabteilungen`, `bettenzahl`, `kontakt` |
| `elektrogeraet` | Elektrogerät | Haushalt | `geraetetyp`, `hersteller`, `baujahr`, `energieeffizienzklasse`, `zustand` |
| `haushaltstextil` | Haushaltstextil | Haushalt | `material`, `masse`, `farbe`, `pflegehinweis`, `zustand` |
| `kuechengeraet` | Küchengerät/-utensil | Haushalt | `typ`, `material`, `hersteller`, `leistung`, `zustand` |
| `moebel` | Möbel | Haushalt | `material`, `masse`, `hersteller`, `zustand`, `standort` |
| `bissanzeiger` | Bissanzeiger / Pose | Hegenenfischen | `art`, `tragkraft`, `farbe`, `montage`, `besonderheiten` |
| `echolot` | Echolot | Hegenenfischen | `hersteller`, `frequenz`, `tiefe`, `gps`, `stromversorgung` |
| `hegenenblei` | Hegenenblei / Endblei | Hegenenfischen | `gewicht`, `form`, `tiefe`, `material`, `besonderheiten` |
| `hegenenrute` | Hegenenrute | Hegenenfischen | `laenge`, `spitze`, `wurfgewicht`, `einsatz`, `hersteller` |
| `hegenensystem` | Hegene (Nymphen-System) | Hegenenfischen | `anzahl`, `hakengroesse`, `nymphenfarbe`, `vorfach`, `zielfisch` |
| `romanfigur` | Figur (Belletristik) | Literatur | `rolle`, `beruf`, `beschreibung`, `kontakt` |
| `werk` | Literarisches Werk | Literatur | `autor`, `erscheinungsjahr`, `gattung`, `quelle`, `kontakt` |
| `allianz` | Luftfahrtallianz | Luftfahrt | `gruendungsjahr`, `zentrale`, `mitgliederzahl`, `kontakt`, `status` |
| `flottentyp` | Flottentyp (Flugzeuge) | Luftfahrt | `hersteller`, `anzahl`, `baujahrvon`, `baujahrbis`, `kontakt` |
| `fluglinie` | Fluglinie | Luftfahrt | `land`, `drehkreuz`, `iatacode`, `kontakt`, `status` |
| `nachname` | Nachname / Familienname | Namenskunde | `bedeutung`, `sprachursprung`, `ersteerwaehnung`, `typ`, `kontakt` |
| `vorname` | Vorname | Namenskunde | `bedeutung`, `sprachursprung`, `geschlecht`, `haeufigkeit`, `kontakt` |
| `aussichtspunkt` | Aussichtspunkt | Natur & Landschaft | `hoehe`, `blick`, `zugang`, `besteZeit`, `besonderheiten` |
| `bucht` | Bucht | Natur & Landschaft | `zugang`, `ankerplatz`, `beschaffenheit`, `schnorcheln`, `besonderheiten` |
| `gipfel` | Berg / Gipfel | Natur & Landschaft | `hoehe`, `gebirge`, `aufstieg`, `schwierigkeit`, `besonderheiten` |
| `hoehle` | Höhle | Natur & Landschaft | `laenge`, `fuehrung`, `temperatur`, `oeffnungszeiten`, `eintritt` |
| `insel` | Insel | Natur & Landschaft | `flaeche`, `einwohnerzahl`, `erreichbarkeit`, `hauptort`, `besonderheiten` |
| `nationalpark` | Nationalpark / Schutzgebiet | Natur & Landschaft | `flaeche`, `schutzstatus`, `eintritt`, `besucherzentrum`, `besonderheiten` |
| `see` | See / Gewässer | Natur & Landschaft | `flaeche`, `tiefe`, `baden`, `wassertemperatur`, `besonderheiten` |
| `strand` | Strand | Natur & Landschaft | `beschaffenheit`, `wasserqualitaet`, `infrastruktur`, `badeaufsicht`, `besonderheiten` |
| `wanderweg` | Wanderweg / Route | Natur & Landschaft | `laenge`, `hoehenmeter`, `dauer`, `schwierigkeit`, `markierung` |
| `wasserfall` | Wasserfall / Schlucht | Natur & Landschaft | `hoehe`, `gewaesser`, `zugang`, `eintritt`, `besteZeit` |
| `buehne` | Bühne | Ort | `zugehoerigeanlage`, `hoehenlage`, `traglast`, `zugang`, `nutzung` |
| `dach` | Dach | Ort | `gebaeude`, `nutzung`, `flaeche`, `zugang`, `tragfaehigkeit` |
| `gebaeude` | Gebäude | Ort | `adresse`, `baujahr`, `nutzung`, `flaeche`, `verantwortlich` |
| `gegenstand` | Gegenstand | Ort | `material`, `farbe`, `anzahl`, `kontakt` |
| `keller` | Keller | Ort | `gebaeude`, `nutzung`, `flaeche`, `zugang`, `belueftung` |
| `raum` | Raum | Ort | `gebaeude`, `nutzung`, `flaeche`, `kontakt` |
| `stockwerk` | Stockwerk | Ort | `gebaeude`, `ebene`, `nutzung`, `flaeche`, `zugang` |
| `betriebsingenieur` | Betriebsingenieur | Personen | `fachgebiet`, `zustaendigeanlage`, `kontakt`, `erreichbarkeit`, `qualifikation` |
| `betriebsleiter` | Betriebsleiter | Personen | `zustaendigkeitsbereich`, `erreichbarkeit`, `kontakt`, `vertretung`, `qualifikation` |
| `schichtmitarbeiter` | Schichtmitarbeiter | Personen | `schichtgruppe`, `qualifikation`, `kontakt`, `erreichbarkeit`, `zustaendigkeitsbereich` |
| `baum` | Baum | Pflanzen | `baumart`, `standort`, `pflanzjahr`, `hoehe`, `besonderheiten` |
| `nutzpflanze` | Nutzpflanze | Pflanzen | `pflanzenart`, `standort`, `erntezeit`, `verwendungszweck`, `besonderheiten` |
| `zierpflanze` | Zierpflanze | Pflanzen | `pflanzenart`, `standort`, `pflegehinweise`, `bluetezeit`, `besonderheiten` |
| `amtstraeger` | Amtsträger/in | Politik | `amt`, `zustaendigkeitsbereich`, `partei`, `kontakt`, `amtszeit` |
| `behoerde` | Behörde / Amt | Politik | `zustaendigkeitsbereich`, `adresse`, `leitung`, `kontakt`, `oeffnungszeiten` |
| `partei` | Partei | Politik | `gruendungsjahr`, `ausrichtung`, `vorsitz`, `mitgliederzahl`, `kontakt` |
| `kindergarten` | Kindergarten | Schulen | `traeger`, `adresse`, `leitung`, `plaetze`, `kontakt` |
| `schule` | Schule | Schulen | `schulform`, `adresse`, `schulleitung`, `schuelerzahl`, `kontakt` |
| `altstadt` | Altstadt / Platz | Sehenswürdigkeiten & Kultur | `epoche`, `highlights`, `markttage`, `fussgaengerzone`, `besonderheiten` |
| `ausgrabung` | Ausgrabung / Archäol. Stätte | Sehenswürdigkeiten & Kultur | `epoche`, `entdeckt`, `eintritt`, `oeffnungszeiten`, `besonderheiten` |
| `bruecke` | Brücke | Sehenswürdigkeiten & Kultur | `bauart`, `baujahr`, `laenge`, `nutzung`, `besonderheiten` |
| `burg` | Burg / Ruine | Sehenswürdigkeiten & Kultur | `epoche`, `zustand`, `zugang`, `eintritt`, `besonderheiten` |
| `denkmal` | Denkmal / Monument | Sehenswürdigkeiten & Kultur | `anlass`, `errichtet`, `kuenstler`, `standort`, `besonderheiten` |
| `kirche` | Kirche / Kapelle | Sehenswürdigkeiten & Kultur | `konfession`, `epoche`, `baustil`, `oeffnungszeiten`, `besonderheiten` |
| `kloster` | Kloster / Stift | Sehenswürdigkeiten & Kultur | `orden`, `gruendung`, `besichtigung`, `oeffnungszeiten`, `besonderheiten` |
| `leuchtturm` | Leuchtturm | Sehenswürdigkeiten & Kultur | `baujahr`, `hoehe`, `besteigbar`, `oeffnungszeiten`, `besonderheiten` |
| `museum` | Museum / Galerie | Sehenswürdigkeiten & Kultur | `thema`, `adresse`, `eintritt`, `oeffnungszeiten`, `website` |
| `schloss` | Schloss / Palast | Sehenswürdigkeiten & Kultur | `epoche`, `baustil`, `eintritt`, `oeffnungszeiten`, `website` |
| `angelschnur` | Angelschnur (Haupt-) | Spinnfischen / Blinkern | `typ`, `durchmesser`, `tragkraft`, `laenge`, `farbe` |
| `blinker` | Blinker / Spinner | Spinnfischen / Blinkern | `typ`, `gewicht`, `farbe`, `zielfisch`, `hersteller` |
| `gummikoeder` | Gummiköder | Spinnfischen / Blinkern | `form`, `laenge`, `jigkopf`, `farbe`, `zielfisch` |
| `spinnrute` | Spinnrute | Spinnfischen / Blinkern | `wurfgewicht`, `laenge`, `aktion`, `zielfisch`, `hersteller` |
| `stationaerrolle` | Stationärrolle | Spinnfischen / Blinkern | `groesse`, `uebersetzung`, `bremskraft`, `schnurfassung`, `hersteller` |
| `wobbler` | Wobbler | Spinnfischen / Blinkern | `laenge`, `gewicht`, `tauchtiefe`, `lauf`, `farbe` |
| `anlage` | Anlage | Strukturelement | `anlagennummer`, `bezeichnung`, `standort`, `verantwortlich`, `status` |
| `teilanlage` | Teilanlage | Strukturelement | `teilanlagennummer`, `bezeichnung`, `uebergeordneteanlage`, `verantwortlich`, `status` |
| `betriebsmeldung` | Betriebsmeldung | Störung | `datum`, `meldender`, `beschreibung`, `prioritaet`, `status` |
| `n1stoerung` | N1-Störung | Störung | `datum`, `ursache`, `auswirkung`, `sofortmassnahme`, `status` |
| `aal` | Aal | Süßwasserfische (Arten) | `latein`, `groesse`, `gewicht`, `fangzeitpunkt`, `koeder`, `schonzeit`, `mindestmass`, `lebensraum` |
| `aalrutte` | Aalrutte / Quappe | Süßwasserfische (Arten) | `latein`, `groesse`, `gewicht`, `fangzeitpunkt`, `koeder`, `schonzeit`, `mindestmass`, `lebensraum` |
| `aesche` | Äsche | Süßwasserfische (Arten) | `latein`, `groesse`, `gewicht`, `fangzeitpunkt`, `koeder`, `schonzeit`, `mindestmass`, `lebensraum` |
| `aitel` | Aitel / Döbel | Süßwasserfische (Arten) | `latein`, `groesse`, `gewicht`, `fangzeitpunkt`, `koeder`, `schonzeit`, `mindestmass`, `lebensraum` |
| `bachforelle` | Bachforelle | Süßwasserfische (Arten) | `latein`, `groesse`, `gewicht`, `fangzeitpunkt`, `koeder`, `schonzeit`, `mindestmass`, `lebensraum` |
| `bachsaibling` | Bachsaibling | Süßwasserfische (Arten) | `latein`, `groesse`, `gewicht`, `fangzeitpunkt`, `koeder`, `schonzeit`, `mindestmass`, `lebensraum` |
| `barbe` | Barbe | Süßwasserfische (Arten) | `latein`, `groesse`, `gewicht`, `fangzeitpunkt`, `koeder`, `schonzeit`, `mindestmass`, `lebensraum` |
| `brachse` | Brachse / Brasse | Süßwasserfische (Arten) | `latein`, `groesse`, `gewicht`, `fangzeitpunkt`, `koeder`, `schonzeit`, `mindestmass`, `lebensraum` |
| `flussbarsch` | Flussbarsch | Süßwasserfische (Arten) | `latein`, `groesse`, `gewicht`, `fangzeitpunkt`, `koeder`, `schonzeit`, `mindestmass`, `lebensraum` |
| `graskarpfen` | Graskarpfen / Amur | Süßwasserfische (Arten) | `latein`, `groesse`, `gewicht`, `fangzeitpunkt`, `koeder`, `schonzeit`, `mindestmass`, `lebensraum` |
| `hecht` | Hecht | Süßwasserfische (Arten) | `latein`, `groesse`, `gewicht`, `fangzeitpunkt`, `koeder`, `schonzeit`, `mindestmass`, `lebensraum` |
| `huchen` | Huchen | Süßwasserfische (Arten) | `latein`, `groesse`, `gewicht`, `fangzeitpunkt`, `koeder`, `schonzeit`, `mindestmass`, `lebensraum` |
| `karpfen` | Karpfen | Süßwasserfische (Arten) | `latein`, `groesse`, `gewicht`, `fangzeitpunkt`, `koeder`, `schonzeit`, `mindestmass`, `lebensraum` |
| `nase` | Nase | Süßwasserfische (Arten) | `latein`, `groesse`, `gewicht`, `fangzeitpunkt`, `koeder`, `schonzeit`, `mindestmass`, `lebensraum` |
| `rapfen` | Rapfen / Schied | Süßwasserfische (Arten) | `latein`, `groesse`, `gewicht`, `fangzeitpunkt`, `koeder`, `schonzeit`, `mindestmass`, `lebensraum` |
| `regenbogenforelle` | Regenbogenforelle | Süßwasserfische (Arten) | `latein`, `groesse`, `gewicht`, `fangzeitpunkt`, `koeder`, `schonzeit`, `mindestmass`, `lebensraum` |
| `reinanke` | Reinanke / Renke / Maräne | Süßwasserfische (Arten) | `latein`, `groesse`, `gewicht`, `fangzeitpunkt`, `koeder`, `schonzeit`, `mindestmass`, `lebensraum` |
| `rotauge` | Rotauge / Plötze | Süßwasserfische (Arten) | `latein`, `groesse`, `gewicht`, `fangzeitpunkt`, `koeder`, `schonzeit`, `mindestmass`, `lebensraum` |
| `rotfeder` | Rotfeder | Süßwasserfische (Arten) | `latein`, `groesse`, `gewicht`, `fangzeitpunkt`, `koeder`, `schonzeit`, `mindestmass`, `lebensraum` |
| `schleie` | Schleie | Süßwasserfische (Arten) | `latein`, `groesse`, `gewicht`, `fangzeitpunkt`, `koeder`, `schonzeit`, `mindestmass`, `lebensraum` |
| `seeforelle` | Seeforelle | Süßwasserfische (Arten) | `latein`, `groesse`, `gewicht`, `fangzeitpunkt`, `koeder`, `schonzeit`, `mindestmass`, `lebensraum` |
| `seesaibling` | Seesaibling | Süßwasserfische (Arten) | `latein`, `groesse`, `gewicht`, `fangzeitpunkt`, `koeder`, `schonzeit`, `mindestmass`, `lebensraum` |
| `wels` | Wels / Waller | Süßwasserfische (Arten) | `latein`, `groesse`, `gewicht`, `fangzeitpunkt`, `koeder`, `schonzeit`, `mindestmass`, `lebensraum` |
| `zander` | Zander | Süßwasserfische (Arten) | `latein`, `groesse`, `gewicht`, `fangzeitpunkt`, `koeder`, `schonzeit`, `mindestmass`, `lebensraum` |
| `haustier` | Haustier | Tiere | `tierart`, `rasse`, `alter`, `halter`, `besonderheiten` |
| `nutztier` | Nutztier | Tiere | `tierart`, `rasse`, `haltungsort`, `verwendungszweck`, `besonderheiten` |
| `wildtier` | Wildtier | Tiere | `tierart`, `lebensraum`, `bestand`, `schutzstatus`, `besonderheiten` |
| `campingplatz` | Campingplatz | Unterkunft & Gastronomie | `adresse`, `telefon`, `website`, `stellplaetze`, `saison` |
| `ferienwohnung` | Ferienwohnung | Unterkunft & Gastronomie | `adresse`, `telefon`, `email`, `website`, `maxpersonen` |
| `gastgeber` | Gastgeber / Betreiber | Unterkunft & Gastronomie | `rolle`, `telefon`, `email`, `erreichbarkeit`, `seit` |
| `gasthof` | Gasthof / Gasthaus | Unterkunft & Gastronomie | `adresse`, `telefon`, `email`, `website`, `ruhetag` |
| `hotel` | Hotel | Unterkunft & Gastronomie | `adresse`, `telefon`, `email`, `website`, `sterne` |
| `privatzimmer` | Privatzimmer / Pension | Unterkunft & Gastronomie | `adresse`, `telefon`, `email`, `anzahlzimmer`, `anzahlbetten` |
| `restaurant` | Restaurant / Café / Bar | Unterkunft & Gastronomie | `adresse`, `kueche`, `preisniveau`, `oeffnungszeiten`, `reservierung` |
| `tourismusinfo` | Tourismus-Information | Unterkunft & Gastronomie | `adresse`, `telefon`, `email`, `website`, `oeffnungszeiten` |
| `aufzuventil` | Auf/Zu-Ventil | Ventile | `nennweite`, `stellantrieb`, `failsafe`, `werkstoff`, `medium` |
| `regelventil` | Regelventil | Ventile | `kennlinie`, `nennweite`, `stellbereich`, `antriebsart`, `medium` |
| `sicherheitsventil` | Sicherheitsventil | Ventile | `ansprechdruck`, `nennweite`, `abblasemenge`, `norm`, `medium` |
| `autovermietung` | Autovermietung | Verkehr & Mobilität | `anbieter`, `adresse`, `oeffnungszeiten`, `fahrzeugklassen`, `kontakt` |
| `bahnhof` | Bahnhof / Haltestelle Bahn | Verkehr & Mobilität | `betreiber`, `linien`, `barrierefrei`, `services`, `parken` |
| `bushaltestelle` | Bushaltestelle / Busbahnhof | Verkehr & Mobilität | `linien`, `betreiber`, `takt`, `richtung`, `barrierefrei` |
| `faehre` | Fährverbindung | Verkehr & Mobilität | `route`, `betreiber`, `fahrzeit`, `frequenz`, `preis` |
| `flughafen` | Flughafen | Verkehr & Mobilität | `iata`, `betreiber`, `terminals`, `anbindung`, `website` |
| `grenzuebergang` | Grenzübergang | Verkehr & Mobilität | `laender`, `strasse`, `oeffnungszeiten`, `kontrollen`, `wartezeit` |
| `hafen` | Hafen / Marina | Verkehr & Mobilität | `hafentyp`, `betreiber`, `liegeplaetze`, `verbindungen`, `kontakt` |
| `ladestation` | E-Ladestation | Verkehr & Mobilität | `betreiber`, `leistung`, `stecker`, `ladepunkte`, `tarif` |
| `mautstelle` | Mautstelle / Vignette | Verkehr & Mobilität | `strecke`, `gebuehr`, `zahlung`, `vignette`, `besonderheiten` |
| `parkplatz` | Parkplatz / Parkhaus | Verkehr & Mobilität | `art`, `stellplaetze`, `gebuehr`, `einfahrtshoehe`, `wohnmobil` |
| `pass` | Pass / Panoramastraße | Verkehr & Mobilität | `hoehe`, `strasse`, `steigung`, `wintersperre`, `maut` |
| `seilbahn` | Seilbahn / Bergbahn | Verkehr & Mobilität | `strecke`, `hoehenunterschied`, `fahrzeit`, `betriebszeiten`, `preis` |
| `stellplatz` | Wohnmobil-Stellplatz | Verkehr & Mobilität | `stellplaetze`, `gebuehr`, `ver_entsorgung`, `strom`, `kontakt` |
| `tankstelle` | Tankstelle | Verkehr & Mobilität | `marke`, `kraftstoffe`, `oeffnungszeiten`, `shop`, `zahlung` |
| `wartungsplan` | Wartungsplan | Wartung | `intervall`, `verantwortlich`, `letztewartung`, `naechstewartung`, `umfang` |
| `salzwasserfisch` | Salzwasserfisch | Wassertiere | `art`, `lebensraum`, `groesse`, `ernaehrung`, `schutzstatus` |
| `suesswasserfisch` | Süßwasserfisch | Wassertiere | `art`, `gewaessertyp`, `groesse`, `ernaehrung`, `schutzstatus` |
| `wal` | Wal | Wassertiere | `art`, `typ`, `groesseGewicht`, `lebensraum`, `schutzstatus` |
| `haus` | Haus | Wohnen | `adresse`, `wohnflaeche`, `grundstuecksflaeche`, `baujahr`, `eigentuemer` |
| `wohnung` | Wohnung | Wohnen | `adresse`, `wohnflaeche`, `zimmeranzahl`, `mietstatus`, `kontakt` |
| `neutral` | Unbestimmt | – (Reset-Klasse) | – |

Hinweis Süßwasserfische: Beim Zuweisen einer Fischart-Klasse füllt das Tool das Feld `latein` (wissenschaftlicher Name) automatisch vor, falls es leer ist. In einer Modelfile darf es trotzdem gesetzt werden. Das Feld `fangzeitpunkt` erwartet Datum und Uhrzeit im Format `JJJJ-MM-TTThh:mm` (z. B. `"2026-09-30T13:54"`, für `de` und `en` gleich); `groesse` in cm und `gewicht` in kg als Text (z. B. `"68"`, `"2,9"`).

---

## 5. Empfehlungen für Positionierung (`pos2d` / `pos3d`)

Das Tool bietet eine Funktion „Neu anordnen", die alle Positionen automatisch neu berechnet
(Force-Layout). Eine KI, die eine neue Datei erzeugt, kann daher:

- **Einfachste Variante:** Alle Nodes auf `{"x":0,"y":0}` (bzw. `{"x":0,"y":0,"z":0}`) setzen.
  Das Modell lädt fehlerfrei; alle Objekte liegen zunächst übereinander, bis der Nutzer im Tool
  auf „Neu anordnen" klickt.
- **Bessere Variante (empfohlen):** Objekte thematisch/hierarchisch in Kreisen oder Rastern um
  sinnvolle Mittelpunkte verteilen, damit das Modell auch ohne Neuanordnen sofort lesbar ist.
  Bewährtes Muster: pro Hauptkategorie (z. B. Allianz, Konzern, Kategorie) einen Mittelpunkt in
  großem Abstand (z. B. 1500–3000 Einheiten) wählen, zugehörige Objekte im Kreis mit kleinerem
  Radius (z. B. 250–800 Einheiten) darum anordnen, Unterobjekte wiederum in kleinerem Radius um
  ihr übergeordnetes Objekt.
- `pos3d` kann identisch zu `pos2d` mit `z: 0` gesetzt werden, wenn keine echte 3D-Anordnung
  gewünscht ist.

---

## 6. Validierungs-Checkliste vor dem Speichern

Eine KI, die eine Modelfile erzeugt, sollte vor der Ausgabe prüfen:

1. ✅ `format` ist exakt `"objekt-beziehungsmodell-v1"`.
2. ✅ Jede `id` (Node und Edge) kommt nur genau einmal vor (keine Duplikate).
3. ✅ Jeder `class`-Wert eines Node ist einer der Klassen-Keys aus Abschnitt 4 (oder `neutral`).
4. ✅ Jedes `params`-Feld ist ein Objekt der Form `{"de": "...", "en": "..."}`, kein reiner String.
5. ✅ Jede Edge verweist mit `from` und `to` auf tatsächlich existierende Node-`id`-Werte.
6. ✅ Kein Node hat `from === to` in einer selbstreferenzierenden Kante (technisch möglich, aber i. d. R. nicht gewünscht).
7. ✅ Alle Pflichtfelder (`id`, `class`, `name`, `params`, `group`, `pos2d`, `pos3d`, `pinned2d`, `pinned3d` bei Nodes; `id`, `from`, `to`, `rolle`, `kontakt` bei Edges) sind vorhanden.

---

## 7. Minimales, vollständiges Beispiel

Dieses Beispiel ist bereits fehlerfrei ladbar und zeigt zwei Objekte plus eine Beziehung:

```json
{
  "format": "objekt-beziehungsmodell-v1",
  "name": "Beispielmodell",
  "nodes": [
    {
      "id": "obj_0000001",
      "class": "unternehmen",
      "name": "Beispiel GmbH",
      "params": {
        "branche": { "de": "Softwareentwicklung", "en": "Software development" },
        "kontakt": { "de": "info@beispiel.at", "en": "info@beispiel.at" }
      },
      "group": "A",
      "pos2d": { "x": 0, "y": 0 },
      "pos3d": { "x": 0, "y": 0, "z": 0 },
      "pinned2d": false,
      "pinned3d": false
    },
    {
      "id": "obj_0000002",
      "class": "neutral",
      "name": "Max Mustermann",
      "params": {},
      "group": "A",
      "pos2d": { "x": 300, "y": 0 },
      "pos3d": { "x": 300, "y": 0, "z": 0 },
      "pinned2d": false,
      "pinned3d": false
    }
  ],
  "edges": [
    {
      "id": "edge_0000001",
      "from": "obj_0000002",
      "to": "obj_0000001",
      "rolle": { "de": "ist Mitarbeiter von", "en": "is an employee of" },
      "kontakt": { "de": "", "en": "" }
    }
  ],
  "exportedAt": 0
}
```

---

## 8. Kurzanleitung für eine KI, die dieses Format nutzen soll

1. Lies Abschnitt 4 und wähle für jedes reale Objekt die am besten passende Klasse. Falls keine
   passt, verwende `neutral`.
2. Vergib pro Objekt eine eindeutige `id`, fülle `name` und die relevanten `params`-Felder
   (im `{de, en}`-Format) gemäß der Feldliste der gewählten Klasse.
3. Verbinde Objekte über `edges` mit einem beschreibenden `rolle`-Text; nutze `kontakt` für
   Begründungen/Quellenangaben zur Beziehung. Lege `from`/`to` möglichst so an, dass `from` auf
   `to` wirkt (dann ist keine Angabe zu `flow` nötig); sonst `"flow": "backward"` setzen, bei
   Beziehungen ohne sinnvolle Richtung `"showArrow": false`.
4. Ergänze optional `lat`/`lon` bei real verorteten Objekten und `createdAt`, falls ein
   Erstellungszeitpunkt bekannt/sinnvoll ist (siehe Abschnitt 3.1). `images` sollte eine KI in
   der Regel nicht selbst befüllen, da hierfür echte Bilddaten nötig sind.
5. Prüfe die Datei anhand der Checkliste in Abschnitt 6.
6. Speichere die Datei als `.json` — sie kann dann im Tool über „Öffnen" → „📂 Datei vom PC
   wählen" geladen werden.

---

## 9. Verhalten beim Fusionieren zweier Modelle

Nutzt jemand im Tool „Menü" → „Fusionieren", um ein zweites Modelfile in das aktuell geladene
einzufügen, gilt folgende Regel: Zwei Objekte gelten als **dasselbe reale Objekt**, wenn sie
sowohl denselben `class`-Wert **als auch** denselben `name` (ohne Berücksichtigung von
Groß-/Kleinschreibung und umgebenden Leerzeichen) haben. In diesem Fall wird **kein** doppeltes
Objekt angelegt, sondern:

- Leere Felder des bereits vorhandenen Objekts werden mit den Werten aus dem neu eingefügten
  Objekt aufgefüllt.
- Sind beide Felder befüllt, aber unterschiedlich, bleibt der bestehende Wert erhalten; der
  abweichende Wert aus dem fusionierten Modell wird stattdessen automatisch im Feld `bemerkung`
  protokolliert.
- `images` beider Objekte werden zu einer gemeinsamen Galerie zusammengeführt (keine Bilder gehen
  verloren, exakte Duplikate werden nicht doppelt übernommen).
- Fehlen dem bereits vorhandenen Objekt `lat`/`lon`, werden sie vom fusionierten Objekt
  übernommen; hat das bestehende Objekt bereits gültige Koordinaten, bleiben diese unverändert.
- `flow` und `showArrow` einer übernommenen Beziehung bleiben erhalten.
- Bereits vorhandene Beziehungen (`edges`) zu diesem Objekt bleiben erhalten; Kanten aus dem
  fusionierten Modell, die inhaltlich identisch wären (gleiches Ziel, gleiche `rolle`), werden
  nicht doppelt angelegt.

Für eine KI, die ein Modelfile speziell zum späteren Fusionieren mit einem bestehenden Modell
erzeugt, bedeutet das: Objekte, die mit einem bereits bestehenden Objekt zusammengeführt werden
sollen, müssen exakt denselben `name` und dieselbe `class` verwenden wie im Zielmodell.
