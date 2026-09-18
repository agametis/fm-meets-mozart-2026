# Beispiel 3: Interaktives Kunden- und Projektbeziehungsnetz

Persönliche Arbeitsanleitung für eine optionale Live-Demo im Vortrag **„Entwicklung von WebViewer-Widgets in Zeiten von KI“**.

Dieses Beispiel ist als visuell stärkerer Abschluss oder als Reserve gedacht. Es zeigt Beziehungen, die in klassischen FileMaker-Listen und Ausschnitten zwar vorhanden, aber nur schwer als Ganzes erkennbar sind. Der Benutzer kann das Netz zoomen, verschieben, filtern und schrittweise erweitern.

## Ziel des Beispiels

FileMaker übergibt einen Kunden als Ausgangspunkt. Das Widget visualisiert die Beziehungen zwischen Unternehmen, Ansprechpartnern, Projekten und Rechnungen als interaktives Netz. Ein Klick zeigt Details zum ausgewählten Objekt. Über **Beziehungen laden** fordert das Widget weitere verbundene Datensätze von FileMaker an und fügt sie ohne vollständiges Neuladen in die Darstellung ein.

Das Publikum soll dabei verstehen:

- Ein Web-Viewer-Widget kann relationale FileMaker-Daten in eine völlig andere, räumliche Sicht übersetzen.
- FileMaker bleibt die autoritative Datenquelle; das Widget hält nur ein Darstellungsmodell.
- Die Integration kann in beide Richtungen arbeiten: FileMaker setzt den Startkontext, das Widget fordert gezielt Ergänzungen an.
- Eine externe Bibliothek ist sinnvoll, wenn sie eine klar abgegrenzte, komplexe Aufgabe zuverlässig übernimmt.
- Auch eine spektakuläre Visualisierung benötigt einen präzisen Datenvertrag, stabile IDs und definierte Fehlerzustände.

## Warum dieses Beispiel mehr Wow-Effekt besitzt

- Die Elemente bewegen sich beim Aufbau zu einem lesbaren Netz.
- Der Benutzer kann direkt zoomen, verschieben und Knoten fokussieren.
- Filter verändern die sichtbare Struktur unmittelbar.
- Neue Beziehungen erscheinen schrittweise, ohne dass die gesamte Ansicht neu geladen wird.
- Die FileMaker-Daten wirken nicht wie eine Tabelle, obwohl dieselben Datensätze dahinterliegen.

Der Effekt darf die Erklärung nicht überdecken. Die Demo sollte deshalb nur eine kleine, kontrollierte Datenmenge verwenden.

## Projektidentität

| Eigenschaft | Wert |
| --- | --- |
| Anzeigename | Beziehungsnetz |
| FileMaker-Datei | `Beziehungsnetz.fmp12` |
| npm-Paketname | `beziehungsnetz` |
| Widget-ID | `beziehungsnetz` |
| Start der Datenübertragung | FileMaker schiebt einen initialen Ausschnitt in das Widget |
| Nachladen | Widget fordert weitere Beziehungen über FileMaker an |
| Bearbeitbar | Nein |
| Rückmeldungen | Auswahl und Erweiterungsanforderung |

Die FileMaker-Datei wird nicht automatisch durch den Assistenten bearbeitet. Die Navigation und die Ermittlung verbundener Datensätze werden in der FileMaker-Demo manuell eingerichtet.

## Fachlicher Ablauf

1. Der Benutzer öffnet in FileMaker einen Kunden.
2. FileMaker baut einen kleinen Startgraphen aus Kunde, Kontakten und Projekten auf.
3. FileMaker übergibt diesen Graphen über `fmWidgetLoad(payload)` an das Widget.
4. Das Widget ordnet die Knoten automatisch an.
5. Der Benutzer zoomt, verschiebt die Ansicht oder filtert nach Objekttypen.
6. Ein Klick auf einen Knoten öffnet dessen Detailkarte im Widget.
7. Optional meldet das Widget die Auswahl mit `select-record` an FileMaker.
8. Der Benutzer klickt auf **Beziehungen laden**.
9. Das Widget sendet `expand-node` mit der stabilen Knoten-ID an `fmWidget_handleEvent`.
10. FileMaker liefert zusätzliche Knoten und Verbindungen als Callback-Ergebnis.
11. Das Widget führt die Ergänzungen anhand ihrer IDs mit dem vorhandenen Graphen zusammen.

## Datenfluss

```text
Aktueller Kunde in FileMaker
             │
             │ fmWidgetLoad(payload)
             ▼
      interaktiver Startgraph
             │
             ├── Auswahl ──► select-record ──► FileMaker-Aktion
             │
             └── Erweitern ─► expand-node ───► FileMaker sucht Beziehungen
                                      │
                                      ▼
                         zusätzliche Knoten + Kanten
                                      │
                                      ▼
                         Widget ergänzt den Graphen
```

## Repräsentative Startdaten

Die Startdaten bleiben bewusst überschaubar. Der Wow-Effekt entsteht durch die Interaktion und nicht durch hunderte unlesbare Knoten.

Für einen gut lesbaren Ein-Firmen-Zustand steht die Datei [`03-beziehungsnetz-kompakt.json`](./03-beziehungsnetz-kompakt.json) bereit. Sie enthält einen Kunden mit seinen Ansprechpartnern, zwei Projekten, den zugehörigen Rechnungen und mindestens einem Beispiel jedes weiteren Entitätstyps.

Für einen komplexeren, vollständig geladenen Präsentationszustand steht außerdem die Datei [`03-beziehungsnetz-komplex.json`](./03-beziehungsnetz-komplex.json) bereit. Beide Datensätze enthalten alle für diese Demo definierten Entitätstypen:

- Unternehmen
- Kontakte
- Mitarbeitende
- Projekte
- Aufgaben
- Meilensteine
- Angebote
- Verträge
- Rechnungen
- Produkte
- Tickets
- Dokumente

Der kompakte Datensatz eignet sich für die reguläre Demo und bleibt auch auf dem Beamer gut nachvollziehbar. Der große Datensatz eignet sich besonders als Wow-Endzustand sowie zum Testen der Filter und der automatischen Anordnung. Für das schrittweise Nachladen kann zunächst der kleine Startgraph verwendet und anschließend ein Teil eines der beiden Datensätze als Callback-Antwort geliefert werden.

```json
{
  "meta": {
    "title": "Beziehungsnetz",
    "rootId": "company:1001",
    "locale": "de-DE"
  },
  "nodes": [
    {
      "id": "company:1001",
      "recordId": "1001",
      "type": "company",
      "label": "Muster AG",
      "subtitle": "Kunde",
      "status": "active",
      "expandable": true
    },
    {
      "id": "contact:2101",
      "recordId": "2101",
      "type": "contact",
      "label": "Anna Berger",
      "subtitle": "Projektleitung",
      "status": "active",
      "expandable": false
    },
    {
      "id": "contact:2102",
      "recordId": "2102",
      "type": "contact",
      "label": "Lukas Kern",
      "subtitle": "Einkauf",
      "status": "active",
      "expandable": false
    },
    {
      "id": "project:3101",
      "recordId": "3101",
      "type": "project",
      "label": "Website Relaunch",
      "subtitle": "in Umsetzung",
      "status": "active",
      "expandable": true
    },
    {
      "id": "project:3102",
      "recordId": "3102",
      "type": "project",
      "label": "Serviceportal",
      "subtitle": "geplant",
      "status": "planned",
      "expandable": true
    }
  ],
  "edges": [
    {
      "id": "edge:1",
      "source": "company:1001",
      "target": "contact:2101",
      "type": "employs",
      "label": "Ansprechpartnerin"
    },
    {
      "id": "edge:2",
      "source": "company:1001",
      "target": "contact:2102",
      "type": "employs",
      "label": "Ansprechpartner"
    },
    {
      "id": "edge:3",
      "source": "company:1001",
      "target": "project:3101",
      "type": "owns",
      "label": "Projekt"
    },
    {
      "id": "edge:4",
      "source": "company:1001",
      "target": "project:3102",
      "type": "owns",
      "label": "Projekt"
    },
    {
      "id": "edge:5",
      "source": "contact:2101",
      "target": "project:3101",
      "type": "leads",
      "label": "leitet"
    }
  ]
}
```

## Regeln des Datenvertrags

- Jede `id` ist innerhalb des Graphen eindeutig und enthält zur Sicherheit den Objekttyp als Präfix.
- `recordId` ist die stabile FileMaker-Datensatz-ID.
- Der kleine Startgraph verwendet zunächst `company`, `contact`, `project` und `invoice`.
- Das vollständige Demo-Schema ergänzt `employee`, `task`, `milestone`, `quote`, `contract`, `product`, `ticket` und `document`.
- Jede Kante verweist mit `source` und `target` auf existierende Knoten-IDs.
- Knoten und Kanten werden anhand ihrer `id` zusammengeführt, nicht anhand des sichtbaren Labels.
- `expandable` legt fest, ob weitere Beziehungen angefordert werden können.
- Sichtbare Texte sind Darstellungsdaten und dürfen niemals zur Datensatzsuche verwendet werden.
- Die Startmenge sollte etwa 5 bis 12 Knoten umfassen.
- Für die Demo sollte das Netz nach dem Nachladen insgesamt unter etwa 30 Knoten bleiben.

## Anforderung zum Nachladen

Beim Erweitern eines Knotens sendet das Widget:

```json
{
  "event": "expand-node",
  "data": {
    "nodeId": "project:3101",
    "recordId": "3101",
    "type": "project"
  }
}
```

FileMaker antwortet über den vorhandenen Callback mit einer Ergänzung:

```json
{
  "nodes": [
    {
      "id": "invoice:4107",
      "recordId": "4107",
      "type": "invoice",
      "label": "RE-2026-1042",
      "subtitle": "1.250,00 € · offen",
      "status": "open",
      "expandable": false
    },
    {
      "id": "contact:2103",
      "recordId": "2103",
      "type": "contact",
      "label": "Mira Wolf",
      "subtitle": "Entwicklung",
      "status": "active",
      "expandable": false
    }
  ],
  "edges": [
    {
      "id": "edge:6",
      "source": "project:3101",
      "target": "invoice:4107",
      "type": "billed-by",
      "label": "Rechnung"
    },
    {
      "id": "edge:7",
      "source": "contact:2103",
      "target": "project:3101",
      "type": "works-on",
      "label": "arbeitet an"
    }
  ]
}
```

Das Widget führt vorhandene IDs nicht doppelt ein. Ein bereits erfolgreich erweiterter Knoten wird entsprechend markiert, sodass ein erneuter Klick nicht dieselben Daten anfordert.

## Optionales Auswahlereignis

Wenn FileMaker auf eine Auswahl reagieren soll, sendet das Widget:

```json
{
  "event": "select-record",
  "data": {
    "nodeId": "project:3101",
    "recordId": "3101",
    "type": "project"
  }
}
```

Die konkrete Reaktion ist projektspezifisch. Für die Demo kann FileMaker beispielsweise ein passendes Detail-Popover öffnen oder zum Datensatz navigieren. Die Visualisierung selbst darf dabei sichtbar bleiben.

## Fertiger Start-Prompt für den KI-Assistenten

```text
Ich möchte ein FileMaker-Web-Viewer-Widget mit dem Namen „Beziehungsnetz“ entwickeln.

FileMaker übergibt dem Widget einen kleinen Startgraphen für den aktuellen Kunden. Der Graph enthält Knoten für Unternehmen, Kontakte, Projekte und Rechnungen sowie Kanten für deren Beziehungen. Das Widget soll diese Daten als interaktives Netzwerk darstellen. Der Benutzer kann zoomen, die Ansicht verschieben, einen Knoten fokussieren und Knotentypen über eine Legende ein- und ausblenden.

Ein Klick auf einen Knoten zeigt eine kompakte Detailkarte. Optional kann die Auswahl als Ereignis „select-record“ mit nodeId, recordId und type an FileMaker gemeldet werden. Bei Knoten mit expandable = true gibt es die Aktion „Beziehungen laden“. Sie sendet das Ereignis „expand-node“ an FileMaker und erwartet als Antwort zusätzliche nodes und edges. Diese Daten werden anhand ihrer stabilen IDs ohne Duplikate in den vorhandenen Graphen eingefügt. Während des Nachladens muss der betroffene Knoten einen Ladezustand zeigen. Bei einem Fehler bleibt der bisherige Graph erhalten und ein erneuter Versuch ist möglich.

Das Widget bearbeitet keine fachlichen Daten und bleibt deshalb dirty = false. Filter, Auswahl, Zoom und Knotenpositionen sind reine Darstellungszustände.

Für dieses Beispiel ist eine etablierte Graphbibliothek zulässig, sofern sie lokal in den Produktions-Build gebündelt wird und die Anforderungen an Layout, Zoom, Verschieben und Auswahl deutlich vereinfacht. Vergleiche kurz die Umsetzung ohne Bibliothek mit einer kleinen Lösung auf Basis von Cytoscape.js und begründe die Empfehlung. Verwende keine zusätzlichen Plugins, solange die Kernbibliothek genügt.

Verwende die bestehenden Schnittstellen und festen Skriptnamen des Templates. Erfinde keine weitere FileMaker-Brücke. Beginne mit Rückfragen, falls Informationen fehlen. Erstelle danach einen konkreten Plan und warte auf meine ausdrückliche Freigabe, bevor du Dateien änderst. Lege den freigegebenen Plan unter docs/PLAN.md ab und dokumentiere die FileMaker-Integration unter docs/FM-INTEGRATION.md. Die FileMaker-Datei selbst soll nicht automatisch bearbeitet werden.
```

## Erwartete Rückfragen des Assistenten

Der Assistent sollte mindestens klären:

- Welche Knoten- und Beziehungstypen werden benötigt?
- Wie groß sind Startgraph und maximaler Graph?
- Nach welchem Layout sollen Knoten angeordnet werden?
- Welche Informationen erscheinen direkt am Knoten und welche erst in der Detailkarte?
- Welche Filter werden benötigt?
- Was soll ein einfacher Klick, Doppelklick oder die Schaltfläche **Beziehungen laden** auslösen?
- Darf ein Knoten mehrfach erweitert werden?
- Wie werden doppelte Knoten und Kanten verhindert?
- Welche FileMaker-Aktion folgt auf `select-record`?
- Wie wird mit einer fehlerhaften Nachladeanforderung umgegangen?

Empfohlene Entscheidungen für die Demo:

- maximal etwa 30 sichtbare Knoten,
- Start mit einem konzentrischen oder kraftbasierten Layout,
- Details in einer seitlichen Karte,
- Filter für Unternehmen, Kontakte, Projekte und Rechnungen,
- Erweiterung ausschließlich über eine sichtbare Schaltfläche und nicht über einen versteckten Doppelklick,
- ein Knoten wird nach erfolgreicher Erweiterung als geladen markiert,
- Cytoscape.js als einzige neue Laufzeitabhängigkeit,
- kein Speichern und immer `dirty = false`.

## UI-Anforderungen

### Grundansicht

- helle, beamergeeignete Darstellung
- großer Zeichenbereich für das Netz
- kompakte Legende mit ein- und ausschaltbaren Knotentypen
- Schaltflächen **Alles einpassen** und **Auswahl zurücksetzen**
- seitliche Detailkarte für den ausgewählten Knoten
- gut lesbare Labels ohne unnötig lange Texte

### Visuelle Kodierung

- unterschiedliche Form oder Symbolik für Unternehmen, Kontakt, Projekt und Rechnung
- Farbe unterstützt die Unterscheidung, ist aber nicht das einzige Merkmal
- Wurzelknoten ist eindeutig erkennbar
- ausgewählter Knoten besitzt eine sichtbare Kontur
- ausgeblendete Knotentypen entfernen auch nicht mehr sinnvolle Kanten aus der Ansicht

### Nachladezustand

- nur der betroffene Knoten zeigt einen Ladeindikator
- **Beziehungen laden** ist während des Aufrufs deaktiviert
- der bestehende Graph bleibt bedienbar

### Erfolg beim Nachladen

- neue Knoten und Kanten werden ergänzt
- vorhandene Elemente bleiben erhalten
- der neue Ausschnitt wird sichtbar, ohne die Orientierung vollständig zu zerstören
- der erweiterte Knoten wird als geladen markiert

### Fehler beim Nachladen

- bisheriger Graph bleibt unverändert
- verständliche Meldung in der Detailkarte
- erneuter Versuch ist möglich
- keine technische Fehlermeldung oder kein Stacktrace in der Benutzeroberfläche

### Lade-, Leer- und Validierungszustand

- Ladeanzeige, solange noch kein Startgraph übergeben wurde
- Leerzustand, wenn keine Beziehungen vorliegen
- klare Meldung bei fehlenden Knoten-IDs oder Kanten mit unbekannten Endpunkten
- ungültige Ergänzungen werden abgewiesen, ohne den vorhandenen Graphen zu beschädigen

## Zustandsregeln

Das Widget bearbeitet keine fachlichen Daten.

- `dirty` bleibt immer `false`.
- Auswahl, Zoom, Verschieben, Filter und berechnete Knotenpositionen sind Darstellungszustände.
- Nach `fmWidgetLoad(payload)` wird der bisherige Graph vollständig ersetzt.
- Nach erfolgreichem `expand-node` werden neue Elemente in den aktuellen Graphen eingefügt.
- Nach fehlgeschlagenem `expand-node` bleibt der Graph unverändert.
- Bei einem erneuten Startkontext werden Auswahl, Filter und Erweiterungsstatus auf definierte Ausgangswerte gesetzt.

Falls FileMaker den Zustand abfragt, meldet `fmWidget_reportState`:

```json
{
  "dirty": false
}
```

## Technischer Rahmen

- Ausgangspunkt ist `fm-starter-ai` beziehungsweise ein daraus erstelltes Projekt.
- Projektanlage mit `aga-fm-start`.
- Planung und Umsetzung mit dem projektlokalen Skill `aga-fm-widget`.
- React für Oberfläche und Zustandskoordination.
- Cytoscape.js als begründete Graphbibliothek für Layout, Zoom, Verschieben und Auswahl.
- Keine zusätzlichen Layout-Plugins für die erste Version.
- Bibliothek und Anwendung werden vollständig lokal gebündelt; im Betrieb ist keine Internetverbindung erforderlich.
- Entwicklungsmodus mit Beispieldaten: `http://localhost:5173/?data=test`
- Beispieldaten dürfen nicht im Produktions-Build enthalten sein.

Die zusätzliche Bibliothek ist hier kein Selbstzweck. Eine robuste Graphdarstellung mit Kantenführung, Layout, Zoom und Treffererkennung wäre ohne sie der größte Teil des Projekts und würde vom eigentlichen FileMaker-Beispiel ablenken.

## Verbindliche Schnittstelle zum Template

### Skripte in FileMaker

- `fmWidget_getData`
- `fmWidget_handleEvent`
- `fmWidget_upload`
- `fmWidget_reportData`
- `fmWidget_reportState`

Für dieses Beispiel übernimmt `fmWidget_handleEvent` die Ereignisse `expand-node` und optional `select-record`. Der Startgraph wird von FileMaker aktiv übergeben; `fmWidget_getData` ist deshalb im Hauptablauf nicht erforderlich.

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

FileMaker verwendet `fmWidgetLoad(payload)`, um den Startgraphen oder einen vollständig neuen Kontext zu übergeben.

### FMGofer-Hülle

Aufrufe des Widgets an FileMaker enthalten:

- `parameter`
- `callbackName`
- `promiseID`

FileMaker beendet einen erfolgreichen Aufruf über den übergebenen Callback mit `$promiseID`, Nutzlast und `False`. Ein Fehler wird mit `$promiseID`, Fehlermeldung und `True` zurückgegeben. Beim Ereignis `expand-node` ist die Nutzlast die Ergänzung aus `nodes` und `edges`.

## FileMaker-Integration

### Vorbereitung

1. Web Viewer auf dem Kunden- oder Projektlayout anlegen.
2. Option **„JavaScript darf FileMaker-Skripte ausführen“** aktivieren.
3. Produktionsdatei des Widgets in den vom Template vorgesehenen Ablageort übernehmen.
4. Die erzeugte Datei `docs/FM-INTEGRATION.md` für die konkrete Einbindung verwenden.

### Startgraph an das Widget übergeben

Ein projektspezifisches FileMaker-Skript:

1. ermittelt den aktuellen Kunden,
2. sammelt einen kleinen, kontrollierten Ausschnitt verbundener Datensätze,
3. erzeugt `nodes` und `edges` mit stabilen IDs,
4. ruft `fmWidgetLoad` im Web Viewer mit der JSON-Nutzlast auf.

Der Name dieses Skripts darf zur eigenen Lösung passen. Der Adapter `fmWidgetLoad` bleibt unverändert.

### `fmWidget_handleEvent`: `expand-node`

Das feste Skript:

1. liest `parameter.event` und `parameter.data`,
2. validiert `nodeId`, `recordId` und `type`,
3. ermittelt nur die für diesen Knoten vorgesehenen weiteren Beziehungen,
4. baut die Ergänzung aus `nodes` und `edges`,
5. löst den Callback mit `$promiseID`, Ergänzungs-JSON und `False` auf,
6. gibt bei einem Fehler eine verständliche Meldung mit `True` zurück.

### `fmWidget_handleEvent`: `select-record`

Optional führt dasselbe feste Skript die projektspezifische Auswahlaktion aus. Für die Demo sollte diese Aktion die Netzansicht nicht verdecken, damit der visuelle Zusammenhang erhalten bleibt.

## Arbeitsablauf zur Vorbereitung

### 1. Projekt erzeugen

```text
Verwende den Skill aga-fm-start, um ein FileMaker-Web-Viewer-Widget zu erstellen.
```

Als Projektname **Beziehungsnetz** und als Ziel einen neuen, eindeutig benannten Ordner wählen.

### 2. Projekt öffnen und lokalen Skill verwenden

Im erzeugten Projekt den Skill `aga-fm-widget` verwenden. Start-Prompt und repräsentative JSON-Daten aus dieser Datei übergeben.

### 3. Plan prüfen

Vor der Freigabe kontrollieren:

- Ist die gemischte Datenrichtung korrekt beschrieben?
- Ersetzt `fmWidgetLoad` den gesamten Startgraphen?
- Ergänzt `expand-node` vorhandene Daten ohne Duplikate?
- Bleibt der bisherige Graph bei Fehlern erhalten?
- Werden stabile IDs statt sichtbarer Labels verwendet?
- Ist `dirty` immer `false`?
- Ist Cytoscape.js als einzige neue Abhängigkeit begründet?
- Bleiben die festen Skript- und Adapter-Namen unverändert?
- Sind Lade-, Leer-, Filter-, Auswahl- und Fehlerzustände geplant?
- Werden Beispieldaten vom Produktions-Build ausgeschlossen?

Erst danach die Umsetzung ausdrücklich freigeben.

### 4. Im Browser prüfen

1. Start im Mock-Modus.
2. automatisches Layout und Lesbarkeit prüfen.
3. Zoomen, Verschieben und **Alles einpassen** testen.
4. jeden Knotentyp ein- und ausblenden.
5. Knoten auswählen und Detailkarte prüfen.
6. einen erweiterbaren Knoten nachladen.
7. sicherstellen, dass keine doppelten Knoten entstehen.
8. denselben Knoten erneut betätigen und definiertes Verhalten prüfen.
9. Nachladefehler simulieren und erneuten Versuch testen.
10. ungültige Kante und fehlende Knoten-ID testen.
11. sicherstellen, dass alle Interaktionen `dirty = false` lassen.

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

Die projektbezogene `docs/FM-INTEGRATION.md` Schritt für Schritt umsetzen. Danach sowohl den neuen Startkontext als auch das schrittweise Nachladen mit echten FileMaker-Daten testen.

## Abnahmekriterien

- `fmWidgetLoad(payload)` ersetzt den aktuellen Graphen vollständig.
- Der Wurzelknoten ist nach dem Laden eindeutig sichtbar.
- Knoten und Kanten werden lesbar angeordnet.
- Zoomen, Verschieben und **Alles einpassen** funktionieren.
- Knotentypen lassen sich über die Legende filtern.
- Eine Auswahl zeigt die passende Detailkarte.
- `select-record` enthält stabile IDs und den Knotentyp.
- `expand-node` zeigt einen lokalen Ladezustand.
- Die Callback-Antwort ergänzt Knoten und Kanten ohne Duplikate.
- Ein Fehler beim Nachladen lässt den bisherigen Graphen unverändert.
- Ein erneuter Versuch nach einem Fehler ist möglich.
- Ungültige Referenzen führen nicht zum Absturz der Ansicht.
- Das Widget bearbeitet keine fachlichen Daten und bleibt `dirty = false`.
- Cytoscape.js ist im lokalen Produktions-Build enthalten; es werden keine externen Webressourcen benötigt.
- Der Produktions-Build enthält keine Beispieldaten.
- Type-Check, Lint, Tests und Build laufen erfolgreich durch.

## Ablauf der optionalen Live-Demo

Die Demo sollte etwa 5 bis 7 Minuten dauern. Das Widget wird fertig vorbereitet; eine Live-Generierung ist für dieses komplexere Beispiel nicht nötig.

1. In FileMaker einen Kunden mit zunächst wenigen Beziehungen öffnen.
2. Das Widget laden und den Kunden als Wurzelknoten zeigen.
3. Kurz erklären, dass jeder Knoten einem FileMaker-Datensatz entspricht.
4. Zoomen, die Ansicht verschieben und wieder **Alles einpassen**.
5. Über die Legende Rechnungen oder Kontakte aus- und wieder einblenden.
6. Ein Projekt auswählen und die Detailkarte zeigen.
7. **Beziehungen laden** anklicken.
8. Beobachten, wie zusätzliche Kontakte und eine Rechnung erscheinen.
9. Einen neuen Knoten anklicken und die FileMaker-Reaktion zeigen.
10. Mit dem Dirty-State abschließen: Es wurde intensiv interagiert, aber kein fachlicher Datensatz verändert.

Passender Sprechsatz:

> FileMaker kennt alle Beziehungen bereits. Das Widget verändert nicht die Daten — es verändert den Blick darauf und lädt genau den Ausschnitt nach, den der Benutzer gerade braucht.

## Empfohlene Dramaturgie im Vortrag

Dieses dritte Beispiel funktioniert am besten nach den ersten beiden:

1. **Offene Rechnungen:** Daten holen, bearbeiten und speichern.
2. **Projektzeitstrahl:** Daten von FileMaker empfangen und eine Auswahl zurückmelden.
3. **Beziehungsnetz:** beide Richtungen kombinieren und Daten schrittweise visuell erweitern.

Wenn die Zeit knapp wird, nur eine vorbereitete 60- bis 90-sekündige Kurzfassung zeigen:

1. Startgraph öffnen.
2. einen Knoten auswählen,
3. Beziehungen nachladen,
4. mit dem Hinweis auf stabile IDs und die FileMaker-Brücke abschließen.

## Demo-Vorbereitung und Reserve

Vor dem Vortrag:

- Startgraph mit 5 bis 8 Knoten vorbereiten,
- eine Erweiterung mit 3 bis 5 zusätzlichen Knoten vorbereiten,
- alle Labels kurz halten,
- Netz nach dem Nachladen auf dem Beamer auf Lesbarkeit prüfen,
- Animationen ruhig und kurz einstellen,
- Filter und **Alles einpassen** testen,
- einen erfolgreichen und einen fehlschlagenden Nachladefall vorbereiten,
- Browser-Mock und FileMaker-Version griffbereit halten,
- Produktions-Build vorab erzeugen,
- sicherstellen, dass keine Bibliothek zur Laufzeit aus dem Internet geladen wird.

Wenn das automatische Layout in der Live-Demo unruhig wirkt:

1. Animation nicht wiederholt starten.
2. **Alles einpassen** verwenden.
3. auf den vorbereiteten, bereits angeordneten Zustand wechseln.
4. den Schwerpunkt auf Auswahl und Nachladen legen.

Wenn FileMaker nicht antwortet:

1. den vorhandenen Startgraphen weiter bedienen.
2. die erwartete `expand-node`-Anforderung zeigen.
3. die vorbereitete Callback-Antwort erläutern.
4. optional im Browser-Mock die erfolgreiche Erweiterung zeigen.

## Persönliche Abschlussfragen

Nach der Vorbereitung sollte ich diese Fragen ohne Nachschlagen beantworten können:

- Welchen fachlichen Mehrwert besitzt das Netz gegenüber einer Liste?
- Warum enthält die Knoten-ID auch den Objekttyp?
- Warum startet die Demo mit wenigen Knoten?
- Was ist der Unterschied zwischen `fmWidgetLoad` und `expand-node`?
- Wie verhindert das Widget doppelte Knoten und Kanten?
- Warum ist Cytoscape.js hier eine sinnvolle Abhängigkeit?
- Warum bleiben Zoom, Filter, Auswahl und Nachladen trotzdem `dirty = false`?
- Welche Aufgaben übernimmt FileMaker und welche das Widget?
