# Beispiel 4: Auftragskarte für einen Handwerksbetrieb

Persönliche Arbeitsanleitung für eine weitere Widget-Demo im Vortrag **„Entwicklung von WebViewer-Widgets in Zeiten von KI“**.

Dieses Beispiel verbindet FileMaker-Daten mit einer interaktiven OpenStreetMap-Karte. Aufträge erscheinen als Pins an ihren Einsatzorten. Das Symbol zeigt die Art der Arbeit, der visuelle Status zeigt die Dringlichkeit beziehungsweise den Bearbeitungsstand. Ein Pop-up fasst den Auftrag zusammen und kann den zugehörigen Datensatz in FileMaker öffnen.

## Ziel des Beispiels

FileMaker übergibt die aktuellen Handwerkeraufträge einschließlich geografischer Koordinaten an das Widget. Das Widget stellt sie auf einer Karte dar und verwendet für Sanitär-, Elektro-, Heizungs-, Maler-, Tischler- und Dacharbeiten unterschiedliche Pins.

Beim Klick auf einen Pin öffnet sich ein Pop-up mit:

- Auftragsnummer und Status
- Kunde und Einsatzadresse
- Terminfenster
- Kurzbeschreibung der Arbeit
- Ansprechpartner und Telefonnummer
- Schaltfläche **Mehr Details**

Die Schaltfläche sendet die stabile FileMaker-Datensatz-ID an `fmWidget_handleEvent`. FileMaker öffnet anschließend den zugehörigen Auftragsdatensatz.

Das Publikum soll dabei verstehen:

- Ein Widget kann FileMaker-Daten mit einer externen visuellen Datenquelle kombinieren.
- FileMaker liefert die fachlichen Daten und bereits ermittelte Koordinaten.
- Das Widget übernimmt Kartenansicht, Marker, Filter und Pop-ups.
- Unterschiedliche Icons transportieren die Auftragsart schneller als Text allein.
- Die Navigation zurück zu FileMaker erfolgt über einen kleinen, klar definierten Ereignisvertrag.

## Projektidentität

| Eigenschaft | Wert |
| --- | --- |
| Anzeigename | Auftragskarte |
| FileMaker-Datei | `Auftragskarte.fmp12` |
| npm-Paketname | `auftragskarte` |
| Widget-ID | `auftragskarte` |
| Kartendarstellung | Leaflet |
| Kartendaten | OpenStreetMap |
| Datenrichtung | FileMaker schiebt die Aufträge in das Widget |
| Rückmeldung | Widget fordert das Öffnen eines Auftragsdatensatzes an |
| Bearbeitbar | Nein |

Die FileMaker-Datei wird nicht automatisch durch den KI-Assistenten verändert. Das Öffnen des Auftragsdatensatzes wird als projektspezifische Aktion in FileMaker eingerichtet.

## Warum dieses Beispiel gut funktioniert

- Die Karte ist sofort verständlich und benötigt wenig Erklärung.
- Die verschiedenen Auftragsarten erzeugen einen sichtbaren Wow-Effekt.
- Die Demo besitzt einen klaren FileMaker-Bezug und bleibt praktisch.
- Die Interaktion führt sichtbar von der Karte zurück zum Datensatz.
- Das Beispiel zeigt eine sinnvolle zusätzliche Bibliothek, ohne technisch auszuufern.

## Fachlicher Ablauf

1. Der Benutzer öffnet in FileMaker die Tages- oder Wochenplanung.
2. FileMaker ermittelt die anzuzeigenden Aufträge.
3. FileMaker baut eine JSON-Nutzlast mit Adresse, Koordinaten, Auftragsart und Detailinformationen.
4. FileMaker übergibt die Nutzlast mit `fmWidgetLoad(payload)` an das Widget.
5. Das Widget zeigt die OpenStreetMap-Karte und setzt die Marker.
6. Die Kartenansicht wird so angepasst, dass alle gültigen Marker sichtbar sind.
7. Der Benutzer klickt auf einen Pin.
8. Leaflet öffnet das Pop-up mit den Auftragsinformationen.
9. Der Benutzer klickt auf **Mehr Details**.
10. Das Widget sendet `open-order` mit `recordId` und `orderNumber` an `fmWidget_handleEvent`.
11. FileMaker validiert die ID und führt die projektspezifische Navigation aus.

## Datenfluss

```text
Aufträge in FileMaker
        │
        │ JSON mit Adresse und Koordinaten
        │ fmWidgetLoad(payload)
        ▼
OpenStreetMap-Karte im Widget
        │
        ├── Pin anklicken ──► Pop-up anzeigen
        │
        └── Mehr Details ──► open-order
                                  │
                                  ▼
                     FileMaker öffnet Auftragsdatensatz
```

## Repräsentative Beispieldaten

Der vollständige Datensatz liegt in [`04-auftragskarte.json`](./04-auftragskarte.json). Er enthält sechs anonymisierte Aufträge im Salzburger Stadtgebiet:

| Auftragsart | Symbolidee | Beispielstatus |
| --- | --- | --- |
| Sanitär | Wasserhahn oder Wassertropfen | Dringend |
| Elektro | Blitz | Eingeplant |
| Heizung | Thermometer oder Flamme | In Arbeit |
| Malerarbeiten | Farbrolle | Eingeplant |
| Tischlerei | Hammer oder Werkzeug | Neu |
| Dacharbeiten | Hausdach | Erledigt |

Die Adressen und Kontaktdaten sind ausschließlich für die Demo vorgesehen. E-Mail-Adressen oder Telefonnummern aus echten Kundendaten sollten nicht in Entwicklungsfixtures übernommen werden.

## Datenvertrag

### Metadaten

```json
{
  "meta": {
    "title": "Auftragskarte Salzburg",
    "locale": "de-AT",
    "center": {
      "latitude": 47.8014,
      "longitude": 13.0448
    },
    "zoom": 13,
    "fitMarkers": true,
    "tileUrl": "https://tile.openstreetmap.org/{z}/{x}/{y}.png",
    "attribution": "© OpenStreetMap contributors"
  }
}
```

`tileUrl` bleibt konfigurierbar. Dadurch kann ein anderes OSM-basiertes Kachelangebot oder eine eigene Infrastruktur verwendet werden, ohne die Widgetlogik umzubauen.

### Einzelner Auftrag

```json
{
  "recordId": "5001",
  "orderNumber": "AU-2026-184",
  "jobType": "plumbing",
  "jobLabel": "Sanitär",
  "status": "urgent",
  "statusLabel": "Dringend",
  "customer": "Musterhaus Verwaltung",
  "address": {
    "street": "Getreidegasse 15",
    "postalCode": "5020",
    "city": "Salzburg",
    "latitude": 47.80039,
    "longitude": 13.04399
  },
  "appointment": {
    "date": "2026-09-15",
    "timeFrom": "09:00",
    "timeTo": "10:30"
  },
  "summary": "Wasserverlust unter dem Waschbecken prüfen",
  "contactName": "Maria Leitner",
  "contactPhone": "+43 662 555 0101"
}
```

### Regeln

- `recordId` ist die stabile FileMaker-Datensatz-ID.
- `orderNumber` dient nur der sichtbaren Darstellung und nicht der Datensatzsuche.
- `jobType` bestimmt das Pin-Symbol.
- `status` bestimmt Farbe, Kontur oder Statuspunkt des Pins.
- `jobLabel` und `statusLabel` sind bereits für die Anzeige geeignete Texte.
- `latitude` liegt zwischen `-90` und `90`.
- `longitude` liegt zwischen `-180` und `180`.
- Datumswerte werden als `YYYY-MM-DD` übertragen.
- Zeitwerte werden als `HH:mm` übertragen.
- FileMaker übergibt die Koordinaten. Das Widget führt kein automatisches Geocoding aus.

## Ereignis für „Mehr Details“

Das Widget sendet:

```json
{
  "event": "open-order",
  "data": {
    "recordId": "5001",
    "orderNumber": "AU-2026-184"
  }
}
```

Für die Navigation verwendet FileMaker ausschließlich die stabile `recordId`. Die Auftragsnummer kann für Protokollierung oder Fehlermeldungen verwendet werden.

## Pin- und Statuskonzept

Die Auftragsart und der Bearbeitungsstatus beantworten zwei unterschiedliche Fragen:

- **Symbol:** Welche Arbeit ist erforderlich?
- **Farbe oder Kontur:** In welchem Zustand befindet sich der Auftrag?

Empfohlene Statusdarstellung:

| Status | Darstellung |
| --- | --- |
| `urgent` | kräftige rote Kontur und kleines Ausrufezeichen |
| `new` | blaue Kontur |
| `scheduled` | violette Kontur |
| `in-progress` | orangefarbene Kontur |
| `completed` | grüne Kontur und reduzierte Betonung |

Die Bedeutung darf nicht ausschließlich über Farbe vermittelt werden. Pop-up, Tooltip und zugänglicher Markertext nennen Auftragsart und Status zusätzlich als Text.

## Fertiger Start-Prompt für den KI-Assistenten

```text
Ich möchte ein FileMaker-Web-Viewer-Widget mit dem Namen „Auftragskarte“ entwickeln.

FileMaker übergibt dem Widget eine Liste von Handwerkeraufträgen einschließlich stabiler Datensatz-ID, Auftragsnummer, Auftragsart, Status, Kunde, Adresse, Koordinaten, Terminfenster, Kurzbeschreibung und Ansprechpartner. Das Widget soll diese Aufträge mit Leaflet auf einer OpenStreetMap-Karte darstellen.

Jede Auftragsart benötigt ein eigenes, gut unterscheidbares Pin-Symbol: Sanitär, Elektro, Heizung, Malerarbeiten, Tischlerei und Dacharbeiten. Der Auftragsstatus soll zusätzlich über Farbe oder Kontur sichtbar sein. Die Symbole müssen auch ohne Farbe unterscheidbar bleiben. Verwende lokal gebündelte SVG-Symbole oder Leaflet DivIcons und lade keine Icons zur Laufzeit aus dem Internet.

Beim Klick auf einen Pin öffnet sich ein Pop-up mit Auftragsnummer, Status, Kunde, vollständiger Adresse, Terminfenster, Kurzbeschreibung, Ansprechpartner und Telefonnummer. Das Pop-up enthält eine Schaltfläche „Mehr Details“. Diese sendet das Ereignis „open-order“ mit recordId und orderNumber über die vorhandene FileMaker-Brücke. FileMaker entscheidet anschließend über die Navigation zum Datensatz.

Die Karte soll beim Laden alle gültigen Marker einpassen. Zusätzlich werden eine kompakte Legende und Filter nach Auftragsart benötigt. Ungültige oder fehlende Koordinaten dürfen die Karte nicht zum Absturz bringen. Solche Aufträge werden ausgelassen und in einem verständlichen Hinweis gezählt. Wenn keine gültigen Aufträge vorhanden sind, zeigt das Widget einen Leerzustand.

Das Widget bearbeitet keine fachlichen Daten. Filter, Zoom, Kartenausschnitt, geöffnetes Pop-up und Markerauswahl sind reine Darstellungszustände; dirty bleibt immer false.

Nutze Leaflet als einzige zusätzliche Laufzeitbibliothek. Bündle Leaflet, dessen CSS und die eigenen Marker-Assets lokal in den Produktions-Build. Die Kartenkacheln dürfen über eine konfigurierbare tileUrl geladen werden. Zeige die Attribution sichtbar auf der Karte und beachte die Nutzungsregeln des gewählten Kachelanbieters. Implementiere kein Vorladen und keinen Offline-Download der offiziellen OpenStreetMap-Kacheln.

Erzeuge Pop-up-Inhalte nicht durch ungeprüftes Einsetzen von FileMaker-Texten in einen HTML-String. Baue die Inhalte als DOM-Elemente auf und verwende textContent für übertragene Texte.

Verwende die bestehenden Schnittstellen und festen Skriptnamen des Templates. Erfinde keine weitere FileMaker-Brücke. Beginne mit Rückfragen, falls Informationen fehlen. Erstelle danach einen konkreten Plan und warte auf meine ausdrückliche Freigabe, bevor du Dateien änderst. Lege den freigegebenen Plan unter docs/PLAN.md ab und dokumentiere die FileMaker-Integration unter docs/FM-INTEGRATION.md. Die FileMaker-Datei selbst soll nicht automatisch bearbeitet werden.
```

## Erwartete Rückfragen des Assistenten

Der Assistent sollte mindestens klären:

- Welche Auftragsarten und Statuswerte existieren?
- Wer liefert die geografischen Koordinaten?
- Welche Informationen dürfen im Pop-up erscheinen?
- Nach welchen Eigenschaften soll gefiltert werden?
- Wie viele Marker werden typischerweise gleichzeitig angezeigt?
- Was soll bei mehreren Aufträgen an derselben Adresse geschehen?
- Wie soll FileMaker auf `open-order` reagieren?
- Welcher Kachelanbieter wird für Demo und produktiven Betrieb verwendet?
- Muss die Karte ohne Internetverbindung funktionieren?

Empfohlene Entscheidungen für die Demo:

- sechs Auftragsarten,
- maximal etwa 50 sichtbare Marker,
- Koordinaten werden in FileMaker gespeichert und mitgeliefert,
- Filter ausschließlich nach Auftragsart,
- mehrere Aufträge an derselben Adresse erhalten leicht versetzte Marker oder eine gemeinsame Auftragsliste im Pop-up,
- Leaflet ohne zusätzliche Clusterbibliothek,
- kein Offlinebetrieb mit dem offiziellen OpenStreetMap-Kachelserver,
- immer `dirty = false`.

## UI-Anforderungen

### Kartenansicht

- helle, beamergeeignete Karte
- sichtbare OpenStreetMap-Attribution
- Schaltfläche **Alle Aufträge anzeigen** zum erneuten Einpassen
- Zoom-Steuerung
- kompakte Legende der Auftragssymbole
- Filter für die sechs Auftragsarten
- keine Überlagerung der Attribution durch eigene Bedienelemente

### Marker

- mindestens etwa 32 × 40 Pixel groß
- klarer Ankerpunkt an der Adresse
- Symbol und zugänglicher Text entsprechen `jobType`
- Statusdarstellung bleibt zusätzlich im Pop-up lesbar
- Tastaturbedienung für Marker bleibt aktiviert

### Pop-up

- Auftragsnummer und Status als Kopfzeile
- Kunde, Adresse, Termin und Beschreibung gut lesbar
- Telefonnummer als Text, optional als anklickbarer `tel:`-Link
- deutlich sichtbare Schaltfläche **Mehr Details**
- keine ungeprüfte HTML-Ausgabe von FileMaker-Texten

### Ladezustand

- Kartenfläche mit ruhiger Ladeanzeige, bis Daten und Kartenbibliothek bereit sind
- Fehlermeldung, wenn keine Kartenkacheln geladen werden können

### Leerzustand

- Text wie: **Für den gewählten Zeitraum liegen keine Aufträge mit Kartenposition vor.**

### Ungültige Koordinaten

- betreffende Aufträge werden nicht als Marker dargestellt
- Hinweis nennt die Anzahl ausgelassener Aufträge
- gültige Aufträge bleiben weiterhin sichtbar

### Fehler bei „Mehr Details“

- Pop-up bleibt geöffnet
- Schaltfläche wird wieder aktiv
- verständliche Meldung erlaubt einen erneuten Versuch
- keine technische Fehlermeldung im sichtbaren Bereich

## Zustandsregeln

Das Widget bearbeitet keine fachlichen Daten.

- `dirty` bleibt immer `false`.
- Kartenposition, Zoom, Filter und geöffnetes Pop-up sind Darstellungszustände.
- `fmWidgetLoad(payload)` ersetzt die aktuelle Auftragsmenge vollständig.
- Nach neuen Daten werden Filter auf einen definierten Ausgangszustand gesetzt.
- Bei ungültigen Daten bleibt die Karte für alle gültigen Aufträge bedienbar.
- Das Ereignis `open-order` verändert keinen Widgetdatensatz.

Falls FileMaker den Zustand abfragt, meldet `fmWidget_reportState`:

```json
{
  "dirty": false
}
```

## Technischer Rahmen

- Ausgangspunkt ist `fm-starter-ai` beziehungsweise ein daraus erzeugtes Projekt.
- Projektanlage mit `aga-fm-start`.
- Planung und Umsetzung mit dem projektlokalen Skill `aga-fm-widget`.
- React für Oberfläche und Zustandskoordination.
- Leaflet für Kartenansicht, Marker und Pop-ups.
- Leaflet und Marker-Assets lokal bündeln.
- OpenStreetMap-Kacheln nur für die aktuell sichtbare Kartenansicht laden.
- Keine Geocoding-Anfragen aus dem Widget.
- Entwicklungsmodus mit Beispieldaten: `http://localhost:5173/?data=test`
- Beispieldaten dürfen nicht im Produktions-Build enthalten sein.

## Sicherheit der Pop-up-Inhalte

Leaflet kann Pop-up-Inhalte als HTML-String übernehmen. Da Texte aus FileMaker stammen können, werden sie nicht direkt in einen HTML-String eingesetzt.

Empfohlene Vorgehensweise:

1. Pop-up-Wurzelelement mit `document.createElement` erzeugen.
2. Textelemente anlegen.
3. übertragene Werte ausschließlich über `textContent` setzen.
4. Schaltfläche als echtes `button`-Element anlegen.
5. Klick-Handler direkt an dieses Element binden.
6. fertiges DOM-Element an Leaflet übergeben.

Damit bleiben Sonderzeichen korrekt und unerwartetes HTML aus den Daten wird nicht ausgeführt.

## OpenStreetMap-Nutzung

Für eine kleine, normale Konferenzdemo können die offiziellen Rasterkacheln interaktiv geladen werden. Dabei gelten insbesondere:

- korrekte HTTPS-Kachel-URL,
- sichtbare Attribution **© OpenStreetMap contributors**,
- normales Browser-Caching respektieren,
- kein massenhaftes Vorladen,
- kein Offline-Download,
- keine künstlichen automatisierten Kartenbewegungen zum Kacheldownload.

Der offizielle Kachelservice besitzt keine Verfügbarkeitsgarantie. Für eine produktive gewerbliche Anwendung sollte die Kachelquelle konfigurierbar bleiben und ein geeigneter OSM-basierter Anbieter oder eine eigene Infrastruktur gewählt werden.

Wichtige Quellen:

- [OpenStreetMap Tile Usage Policy](https://operations.osmfoundation.org/policies/tiles/)
- [Leaflet: Markers With Custom Icons](https://leafletjs.com/examples/custom-icons/)
- [Leaflet Reference: Marker, Popup und Icon](https://leafletjs.com/reference)

## Verbindliche Schnittstelle zum Template

### Skripte in FileMaker

- `fmWidget_getData`
- `fmWidget_handleEvent`
- `fmWidget_upload`
- `fmWidget_reportData`
- `fmWidget_reportState`

Der Hauptablauf verwendet `fmWidget_handleEvent` für `open-order`. Die Aufträge werden aktiv mit `fmWidgetLoad(payload)` übergeben, daher wird `fmWidget_getData` im Hauptablauf nicht benötigt.

### JavaScript-API des Widgets

- `window.fmWidget.load(payload)`
- `window.fmWidget.refresh()`
- `window.fmWidget.requestData()`
- `window.fmWidget.requestState()`

### Globale Adapter für Aufrufe aus FileMaker

- `fmWidgetLoad(payload)`
- `fmWidgetRefresh()`
- `fmWidgetRequestData()`
- `fmWidgetRequestState()`

### FMGofer-Hülle

Der Aufruf von `open-order` enthält:

- `parameter`
- `callbackName`
- `promiseID`

FileMaker bestätigt einen erfolgreichen Aufruf mit `$promiseID`, einer kleinen Ergebnisnutzlast und `False`. Ein Fehler wird mit `$promiseID`, einer verständlichen Fehlermeldung und `True` zurückgegeben.

## FileMaker-Integration

### Vorbereitung

1. Web Viewer auf dem Dispositions- oder Auftragslayout anlegen.
2. Option **„JavaScript darf FileMaker-Skripte ausführen“** aktivieren.
3. Produktionsdatei des Widgets in den vom Template vorgesehenen Ablageort übernehmen.
4. Erzeugte Datei `docs/FM-INTEGRATION.md` für die konkrete Einbindung verwenden.

### Auftragsdaten an die Karte übergeben

Ein projektspezifisches FileMaker-Skript:

1. ermittelt die Aufträge für den gewählten Zeitraum oder Mitarbeiter,
2. liest die bereits gespeicherten Koordinaten,
3. erzeugt die vereinbarte JSON-Nutzlast,
4. ruft `fmWidgetLoad` im Web Viewer auf.

Die Geocodierung einer neu erfassten Adresse gehört in einen separaten, kontrollierten FileMaker-Prozess. Sie darf nicht bei jedem Kartenaufbau erneut stattfinden.

### `fmWidget_handleEvent`: `open-order`

Das feste Skript:

1. liest `parameter.event` und `parameter.data`,
2. erkennt das Ereignis `open-order`,
3. validiert die übergebene `recordId`,
4. sucht den Datensatz im vorgesehenen Auftragskontext,
5. führt die projektspezifische Navigation aus,
6. bestätigt den Aufruf über den Callback oder gibt eine verständliche Ablehnung zurück.

Die konkrete Navigation kann je nach FileMaker-Lösung beispielsweise zu einem Auftragslayout führen oder eine Detailkarte öffnen.

## Arbeitsablauf zur Vorbereitung

### 1. Projekt erzeugen

```text
Verwende den Skill aga-fm-start, um ein FileMaker-Web-Viewer-Widget zu erstellen.
```

Als Projektname **Auftragskarte** und als Ziel einen neuen, eindeutig benannten Ordner wählen.

### 2. Projekt öffnen und lokalen Skill verwenden

Im erzeugten Projekt den Skill `aga-fm-widget` verwenden. Start-Prompt und JSON-Daten aus dieser Anleitung übergeben.

### 3. Plan prüfen

Vor der Freigabe kontrollieren:

- Liefert FileMaker die Koordinaten direkt?
- Bestimmt `jobType` das Symbol und `status` die zusätzliche Statusdarstellung?
- Wird `recordId` für FileMaker-Navigation verwendet?
- Wird `open-order` über `fmWidget_handleEvent` gesendet?
- Werden Pop-up-Texte sicher als Text aufgebaut?
- Bleibt `dirty` immer `false`?
- Sind Attribution und konfigurierbare Kachelquelle berücksichtigt?
- Wird auf Vorladen und Offline-Download offizieller OSM-Kacheln verzichtet?
- Ist Leaflet die einzige neue Laufzeitbibliothek?
- Werden Beispieldaten vom Produktions-Build ausgeschlossen?

Erst danach die Umsetzung ausdrücklich freigeben.

### 4. Im Browser prüfen

1. Start im Mock-Modus.
2. alle sechs Pin-Symbole prüfen.
3. alle Marker mit **Alle Aufträge anzeigen** einpassen.
4. Filter einzeln aktivieren und deaktivieren.
5. jeden Pin per Maus öffnen.
6. mindestens einen Marker per Tastatur öffnen.
7. Pop-up-Inhalte mit den JSON-Daten vergleichen.
8. `open-order` und die übertragene `recordId` prüfen.
9. ungültige Koordinaten testen.
10. leere Datenmenge testen.
11. nicht erreichbare Kachelquelle testen.
12. sicherstellen, dass alle Interaktionen `dirty = false` lassen.

### 5. Qualitätsprüfungen ausführen

```text
npm run type-check
npm run lint
npm test
npm run build
```

Optional, wenn die lokale FileMaker-Integration vorbereitet ist:

```text
npm run deploy-to-fm
```

### 6. In FileMaker integrieren

Die projektbezogene `docs/FM-INTEGRATION.md` umsetzen. Anschließend das Laden echter Aufträge und die Navigation über **Mehr Details** prüfen.

## Abnahmekriterien

- `fmWidgetLoad(payload)` ersetzt die angezeigten Aufträge vollständig.
- alle gültigen Marker sind nach dem Laden sichtbar.
- jede Auftragsart besitzt ein eindeutig unterscheidbares Symbol.
- der Status ist zusätzlich visuell und textlich erkennbar.
- Attribution bleibt sichtbar.
- Auftragsarten lassen sich filtern.
- ein Klick auf einen Pin öffnet das richtige Pop-up.
- Pop-up-Texte werden nicht als ungeprüftes HTML ausgegeben.
- **Mehr Details** sendet `open-order` mit stabiler `recordId`.
- FileMaker öffnet den vorgesehenen Auftragsdatensatz oder meldet einen verständlichen Fehler.
- ungültige Koordinaten beeinträchtigen gültige Marker nicht.
- ohne Aufträge erscheint ein verständlicher Leerzustand.
- Karteninteraktionen verändern keine fachlichen Daten und `dirty` bleibt `false`.
- Leaflet und Marker-Assets sind lokal gebündelt.
- der Produktions-Build enthält keine Beispieldaten.
- Type-Check, Lint, Tests und Build laufen erfolgreich durch.

## Ablauf der Live-Demo

Die Demo sollte etwa 5 bis 7 Minuten dauern.

1. In FileMaker die Tagesplanung mit mehreren Aufträgen öffnen.
2. Zur Kartenansicht wechseln und die sechs unterschiedlichen Pins zeigen.
3. Kurz erklären: Symbol steht für Auftrag, Farbe beziehungsweise Kontur für Status.
4. den Filter **Elektro** aktivieren und anschließend wieder alle Aufträge anzeigen.
5. einen dringenden Sanitär-Auftrag öffnen.
6. Adresse, Terminfenster und Kurzbeschreibung im Pop-up zeigen.
7. auf **Mehr Details** klicken.
8. zeigen, dass FileMaker den richtigen Auftragsdatensatz öffnet.
9. zum Abschluss auf die Trennung hinweisen: FileMaker besitzt die Daten, das Widget liefert die Karteninteraktion.

Passender Sprechsatz:

> Die Karte kennt keine FileMaker-Layouts. Sie kennt nur Koordinaten und eine stabile Datensatz-ID. Erst der Klick auf „Mehr Details“ übergibt die Kontrolle wieder an FileMaker.

## Kürzere Reserve-Demo

Wenn wenig Zeit bleibt, reichen etwa 90 Sekunden:

1. Karte mit allen Markern öffnen.
2. zwei unterschiedliche Auftragsarten zeigen.
3. einen Pin öffnen.
4. **Mehr Details** anklicken.
5. den geöffneten FileMaker-Datensatz zeigen.

## Demo-Vorbereitung und Ausfallsicherheit

Vor dem Vortrag:

- Internetverbindung am Veranstaltungsort testen,
- alle Kartenausschnitte einmal normal aufrufen, ohne Kacheln massenhaft vorzuladen,
- Kachelquelle und Attribution prüfen,
- fertigen Browser-Mock bereithalten,
- FileMaker-Demo mit bekannten Datensatz-IDs vorbereiten,
- Pin-Symbole auf dem Beamer testen,
- Schriftgröße der Pop-ups ausreichend groß einstellen,
- eine Screenshot-Reserve der Kartenansicht und eines geöffneten Pop-ups bereithalten.

Wenn keine Kartenkacheln geladen werden:

1. erklären, dass Marker und FileMaker-Integration unabhängig von der Kachelquelle funktionieren,
2. zur vorbereiteten Screenshot-Reserve wechseln,
3. das `open-order`-Ereignis anhand des JSON-Vertrags erläutern,
4. die Navigation in FileMaker trotzdem vorführen.

Für eine wirklich offlinefähige Demo dürfen die offiziellen OSM-Kacheln nicht vorab heruntergeladen werden. Dafür wäre ein eigener Kachelbestand oder ein Anbieter erforderlich, dessen Bedingungen Offline-Nutzung ausdrücklich erlauben.

## Persönliche Abschlussfragen

Nach der Vorbereitung sollte ich diese Fragen ohne Nachschlagen beantworten können:

- Warum liefert FileMaker bereits Breiten- und Längengrad?
- Warum werden Auftragsart und Status unterschiedlich kodiert?
- Warum navigiert FileMaker anhand der `recordId` und nicht anhand der Auftragsnummer?
- Welche Aufgabe übernimmt Leaflet und welche OpenStreetMap?
- Warum bleibt `tileUrl` konfigurierbar?
- Warum werden FileMaker-Texte nicht direkt in einen HTML-String eingesetzt?
- Warum ist ein Offline-Download der offiziellen OSM-Kacheln keine zulässige Demo-Reserve?
- Welche Teile gehören zum Widget und welche zur FileMaker-Lösung?

