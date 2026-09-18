# Beispiel 1: Offene Rechnungen bearbeiten

Persönliche Arbeitsanleitung für die Live-Demo im Vortrag **„Entwicklung von WebViewer-Widgets in Zeiten von KI“**.

Dieses Beispiel ist die Hauptgeschichte des Vortrags. Es zeigt den vollständigen Weg von einer fachlichen Idee über den KI-gestützten Entwicklungsprozess bis zur Einbindung in FileMaker. Das Widget ist bewusst klein, besitzt aber echte Zustandsänderungen und einen Speichervorgang.

## Ziel des Beispiels

Eine Liste offener Rechnungen wird aus FileMaker geladen. Im Widget kann der Status einer Rechnung auf **bezahlt** gesetzt werden. Das Widget übermittelt beim Speichern ausschließlich die Änderungen an FileMaker.

Das Publikum soll dabei verstehen:

- Ein guter Auftrag an den KI-Assistenten beginnt mit einer klaren fachlichen Beschreibung und repräsentativen Beispieldaten.
- FileMaker bleibt Datenquelle und führt die Geschäftslogik aus.
- Das Widget kümmert sich um Darstellung, Bedienung und den vorübergehenden UI-Zustand.
- Die Schnittstelle zwischen FileMaker und Widget ist ein klar vereinbarter JSON-Vertrag.
- Ein KI-Assistent ersetzt weder die Planung noch die Abnahme.

## Projektidentität

| Eigenschaft | Wert |
| --- | --- |
| Anzeigename | Offene Rechnungen |
| FileMaker-Datei | `Offene Rechnungen.fmp12` |
| npm-Paketname | `offene-rechnungen` |
| Widget-ID | `offeneRechnungen` |
| Hauptrichtung der Daten | Widget fragt Daten bei FileMaker an |
| Bearbeitbar | Ja, Status einer Rechnung |
| Speichern | Änderungssatz an FileMaker |

Die FileMaker-Datei wird nicht automatisch vom Assistenten verändert. Die in dieser Anleitung beschriebenen FileMaker-Schritte werden anschließend manuell umgesetzt.

## Fachlicher Ablauf

1. Das Widget startet im Web Viewer.
2. Es fordert über `fmWidget_getData` die offenen Rechnungen an.
3. FileMaker liefert die Daten als JSON.
4. Der Benutzer markiert eine Rechnung als bezahlt.
5. Das Widget zeigt an, dass ungespeicherte Änderungen vorhanden sind.
6. Der Benutzer klickt auf **Änderungen speichern**.
7. Das Widget sendet den Änderungssatz über `fmWidget_handleEvent` an FileMaker.
8. FileMaker verarbeitet die Änderung und meldet Erfolg oder Fehler zurück.
9. Erst bei Erfolg übernimmt das Widget den neuen Zustand als Ausgangszustand.

## Datenfluss

```text
FileMaker-Daten
      │
      │ fmWidget_getData
      ▼
Widget ── Benutzer ändert Status ──► lokaler, ungespeicherter Zustand
      │
      │ fmWidget_handleEvent: save-changes
      ▼
FileMaker verarbeitet Änderung
      │
      ├── Erfolg ──► Widget setzt dirty = false
      └── Fehler  ──► Widget behält Änderung und dirty = true
```

## Repräsentative Beispieldaten

Diese Daten werden für Entwicklung, Tests und die Browser-Demo verwendet. Sie sind absichtlich anonymisiert, aber fachlich realistisch.

```json
{
  "meta": {
    "title": "Offene Rechnungen",
    "currency": "EUR",
    "locale": "de-DE"
  },
  "data": [
    {
      "recordId": "42",
      "invoiceNumber": "RE-2026-1042",
      "customer": "Muster AG",
      "amount": 125.5,
      "dueDate": "2026-09-15",
      "status": "open"
    },
    {
      "recordId": "57",
      "invoiceNumber": "RE-2026-1057",
      "customer": "Beispiel GmbH",
      "amount": 89.9,
      "dueDate": "2026-09-20",
      "status": "open"
    },
    {
      "recordId": "63",
      "invoiceNumber": "RE-2026-1063",
      "customer": "Mozart & Partner",
      "amount": 460,
      "dueDate": "2026-09-08",
      "status": "overdue"
    }
  ]
}
```

Wichtige Regeln für die Daten:

- `recordId` ist die stabile FileMaker-Datensatz-ID und darf nicht aus der sichtbaren Sortierung abgeleitet werden.
- Datumswerte werden im ISO-Format `YYYY-MM-DD` übertragen.
- Geldbeträge werden als Zahlen und nicht als bereits formatierte Texte übertragen.
- Die Darstellung in Euro und im deutschen Zahlenformat übernimmt das Widget anhand von `currency` und `locale`.
- Mögliche Statuswerte sind in diesem Beispiel `open`, `overdue` und `paid`.

## Änderungssatz beim Speichern

Das Widget sendet nicht die vollständige Liste zurück, sondern nur die tatsächlich geänderten Felder.

```json
{
  "event": "save-changes",
  "data": {
    "updated": [
      {
        "recordId": "42",
        "changes": {
          "status": "paid"
        }
      }
    ],
    "created": [],
    "deleted": []
  }
}
```

## Fertiger Start-Prompt für den KI-Assistenten

Den folgenden Text kann ich nach dem Anlegen des Projekts als ersten fachlichen Auftrag verwenden. Die Beispieldaten aus dem vorherigen Abschnitt werden zusammen mit dem Prompt bereitgestellt.

```text
Ich möchte ein FileMaker-Web-Viewer-Widget mit dem Namen „Offene Rechnungen“ entwickeln.

Das Widget soll beim Start seine Daten von FileMaker anfordern und eine kompakte, gut lesbare Tabelle offener Rechnungen anzeigen. Jede Zeile zeigt Rechnungsnummer, Kunde, Fälligkeit, Betrag und Status. Die Liste soll nach den sichtbaren Spalten sortierbar sein. Bei den zu erwartenden wenigen hundert Datensätzen genügt eine einfache clientseitige Sortierung.

Der Benutzer darf ausschließlich den Status einer Rechnung auf „bezahlt“ setzen. Eine lokale Änderung muss sofort sichtbar sein und den Zustand des Widgets auf dirty setzen. Ein deutlich sichtbarer Button „Änderungen speichern“ sendet nur den Änderungssatz an FileMaker. Während des Speicherns ist der Button deaktiviert. Bei erfolgreichem Speichern wird eine kurze Bestätigung angezeigt und dirty auf false gesetzt. Bei einem Fehler bleibt die Änderung erhalten, dirty bleibt true und eine verständliche Fehlermeldung wird angezeigt.

Verwende die bestehenden Schnittstellen und festen Skriptnamen des Templates. Erfinde keine zusätzliche FileMaker-Brücke. Nutze die kleinste sinnvolle technische Lösung und füge keine Tabellenbibliothek hinzu, solange die Anforderungen mit den vorhandenen Mitteln gut erfüllbar sind.

Bitte beginne mit Rückfragen, falls Informationen fehlen. Erstelle danach einen konkreten Plan und warte auf meine ausdrückliche Freigabe, bevor du Dateien änderst. Lege den freigegebenen Plan unter docs/PLAN.md ab und dokumentiere die FileMaker-Integration unter docs/FM-INTEGRATION.md. Die FileMaker-Datei selbst soll nicht automatisch bearbeitet werden.
```

## Erwartete Rückfragen des Assistenten

Der Assistent sollte mindestens klären:

- Welche Spalten werden gezeigt?
- Welche Felder dürfen verändert werden?
- Soll eine Statusänderung direkt oder erst über einen Speichern-Button übertragen werden?
- Wie viele Datensätze sind realistisch?
- Welche Lade-, Leer-, Fehler- und Speicherzustände werden benötigt?
- Was geschieht, wenn der Benutzer den Web Viewer mit ungespeicherten Änderungen verlassen möchte?
- Wie soll FileMaker auf einen erfolgreichen oder fehlgeschlagenen Speichervorgang reagieren?

Wenn der Assistent diese Fragen nicht stellt, kann ich sie selbst ergänzen. Der Zweck der Demo ist nicht, dass der erste Prompt magisch perfekt funktioniert, sondern dass ein kontrollierter Dialog zu einem belastbaren Plan führt.

## UI-Anforderungen

### Normalzustand

- Überschrift **Offene Rechnungen**
- kompakte Tabelle mit gut lesbaren Spalten
- Betrag im deutschen Währungsformat
- überfällige Rechnungen visuell erkennbar, aber nicht alarmistisch gestaltet
- Statusaktion eindeutig beschriftet

### Ladezustand

- sichtbare, ruhige Ladeanzeige
- keine leere Tabelle, die fälschlich wie ein Ergebnis wirkt

### Leerzustand

- Text wie: **Keine offenen Rechnungen vorhanden.**

### Ungespeicherter Zustand

- Speichern-Button wird aktiv
- eine kurze Statusanzeige weist auf ungespeicherte Änderungen hin
- `dirty` ist `true`

### Speicherzustand

- Speichern-Button ist vorübergehend deaktiviert
- doppeltes Absenden wird verhindert

### Erfolg

- kurze Bestätigung
- geänderte Daten bleiben sichtbar
- neuer Ausgangszustand wird gesetzt
- `dirty` wird `false`

### Fehler

- verständliche Meldung ohne technischen Stacktrace
- lokale Änderung bleibt erhalten
- erneutes Speichern ist möglich
- `dirty` bleibt `true`

## Zustandsregeln

Das Widget vergleicht den aktuellen bearbeitbaren Zustand mit einem Ausgangszustand.

- Nach erfolgreichem Laden: Ausgangszustand aktualisieren, `dirty = false`.
- Nach einer fachlichen Änderung: `dirty = true`.
- Nach erfolgreichem Speichern: Ausgangszustand aktualisieren, `dirty = false`.
- Nach fehlgeschlagenem Speichern: Ausgangszustand nicht verändern, `dirty = true`.
- Reine Sortierung oder Auswahl einer Tabellenzeile ist keine fachliche Datenänderung und setzt `dirty` nicht.

Wenn FileMaker den Zustand abfragt, meldet `fmWidget_reportState` nur:

```json
{
  "dirty": true
}
```

beziehungsweise:

```json
{
  "dirty": false
}
```

## Technischer Rahmen

- Ausgangspunkt ist das Repository `fm-starter-ai` beziehungsweise ein daraus erstelltes Projekt.
- Projektanlage mit dem Skill `aga-fm-start`.
- Planung und Umsetzung im Projekt mit dem dort enthaltenen Skill `aga-fm-widget`.
- Bestehenden Stack des Templates verwenden.
- Keine zusätzliche Tabellen- oder State-Management-Bibliothek für dieses Beispiel.
- Entwicklungsmodus mit Beispieldaten: `http://localhost:5173/?data=test`
- Beispieldaten dürfen nicht im Produktions-Build enthalten sein.

## Verbindliche Schnittstelle zum Template

### Skripte in FileMaker

- `fmWidget_getData`
- `fmWidget_handleEvent`
- `fmWidget_upload`
- `fmWidget_reportData`
- `fmWidget_reportState`

Für dieses Beispiel werden hauptsächlich `fmWidget_getData`, `fmWidget_handleEvent` und `fmWidget_reportState` verwendet. Die festen Namen werden nicht projektspezifisch umbenannt.

### JavaScript-API des Widgets

- `window.fmWidget.load(payload)`
- `window.fmWidget.refresh()`
- `window.fmWidget.requestData()`
- `window.fmWidget.requestState()`

### Globale Adapter für Aufrufe aus FileMaker

FileMaker ruft globale Funktionsnamen auf und keine Namen mit Punkten:

- `fmWidgetLoad(payload)`
- `fmWidgetRefresh()`
- `fmWidgetRequestData()`
- `fmWidgetRequestState()`

### FMGofer-Hülle

Aufrufe des Widgets an FileMaker enthalten:

- `parameter`
- `callbackName`
- `promiseID`

FileMaker beendet einen erfolgreichen Aufruf über den übergebenen Callback mit `$promiseID`, Nutzlast und `False`. Ein Fehler wird mit `$promiseID`, Fehlermeldung und `True` zurückgegeben.

## FileMaker-Integration

### Vorbereitung

1. Web Viewer auf dem gewünschten Layout anlegen.
2. Option **„JavaScript darf FileMaker-Skripte ausführen“** aktivieren.
3. Produktionsdatei des Widgets in das vorgesehene Container-/Textfeld oder in den vom Template vorgesehenen Ablageort übernehmen.
4. Die vom Projekt erzeugte Datei `docs/FM-INTEGRATION.md` als konkrete Integrationsanleitung verwenden.

### `fmWidget_getData`

Das Skript:

1. liest `callbackName` und `promiseID` aus der FMGofer-Hülle,
2. ermittelt die offenen Rechnungen im aktuellen FileMaker-Kontext,
3. baut die vereinbarte JSON-Nutzlast,
4. ruft den Callback mit `$promiseID`, der JSON-Nutzlast und `False` auf,
5. gibt im Fehlerfall eine verständliche Meldung mit `True` zurück.

### `fmWidget_handleEvent`

Das Skript:

1. liest `parameter.event` und `parameter.data`,
2. behandelt im Beispiel das Ereignis `save-changes`,
3. validiert die stabilen `recordId`-Werte und erlaubten Statusänderungen,
4. aktualisiert die Datensätze in FileMaker,
5. bestätigt den Erfolg über den Callback,
6. gibt Validierungs- oder Speicherfehler als Ablehnung zurück.

### `fmWidget_reportState`

FileMaker kann vor Schließen, Layoutwechsel oder Navigation `fmWidgetRequestState()` aufrufen. Der gemeldete `dirty`-Wert entscheidet, ob eine Rückfrage zu ungespeicherten Änderungen nötig ist.

## Arbeitsablauf zur Vorbereitung

### 1. Projekt erzeugen

Den installierten Skill verwenden:

```text
Verwende den Skill aga-fm-start, um ein FileMaker-Web-Viewer-Widget zu erstellen.
```

Als Projektname **Offene Rechnungen** und als Ziel einen neuen, eindeutig benannten Ordner wählen.

### 2. Projekt öffnen und lokalen Skill verwenden

Im erzeugten Projekt den enthaltenen Skill `aga-fm-widget` für Planung und Entwicklung verwenden. Start-Prompt und Beispieldaten aus dieser Datei übergeben.

### 3. Plan prüfen

Vor der Freigabe kontrollieren:

- Ist die Datenrichtung korrekt: Widget fordert Daten an?
- Werden nur Statusänderungen erlaubt?
- Wird ein Änderungssatz statt der vollständigen Liste gespeichert?
- Sind Lade-, Leer-, Fehler- und Speicherzustand geplant?
- Bleiben Änderungen nach einem Fehler erhalten?
- Wird keine unnötige Bibliothek eingeführt?
- Bleiben die festen Skript- und Adapter-Namen unverändert?
- Werden Beispieldaten vom Produktions-Build ausgeschlossen?

Erst danach die Umsetzung ausdrücklich freigeben.

### 4. Im Browser prüfen

Mit den Beispieldaten testen:

1. Start im Mock-Modus.
2. Darstellung und deutsches Zahlenformat prüfen.
3. Sortierung jeder sichtbaren Spalte prüfen.
4. Eine Rechnung auf bezahlt setzen.
5. `dirty` und Speichern-Button prüfen.
6. Speichern erfolgreich simulieren.
7. Speicherfehler simulieren und Erhalt der Änderung prüfen.
8. Lade-, Leer- und Fehlerzustand gezielt prüfen.

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

Die projektbezogene `docs/FM-INTEGRATION.md` Schritt für Schritt umsetzen. Danach dieselben Kernfälle erneut mit echten FileMaker-Daten prüfen.

## Abnahmekriterien

- Das Widget fordert beim Start automatisch Daten von FileMaker an.
- Die Beispieldaten werden korrekt und lesbar dargestellt.
- Beträge erscheinen im deutschen Euroformat.
- Die Liste lässt sich sortieren, ohne den fachlichen Zustand zu verändern.
- Nur der Rechnungsstatus kann bearbeitet werden.
- Eine Änderung setzt `dirty` auf `true`.
- Beim Speichern wird nur der Änderungssatz gesendet.
- Während des Speicherns ist kein doppeltes Absenden möglich.
- Nach erfolgreichem Speichern ist `dirty` wieder `false`.
- Bei abgelehntem Speichern bleiben Änderung und `dirty = true` erhalten.
- FileMaker kann vor dem Verlassen den Dirty-Zustand abfragen.
- Der Produktions-Build enthält keine Beispieldaten.
- Type-Check, Lint, Tests und Build laufen erfolgreich durch.

## Ablauf der Live-Demo

Die Demo kann in zwei kurze Teile geteilt werden. Beide Teile erzählen dennoch eine zusammenhängende Geschichte.

### Live-Demo 1: Von der Idee zum Plan — etwa 6 bis 8 Minuten

1. Leeres beziehungsweise frisch erzeugtes Projekt zeigen.
2. Kurz erklären, dass das Template die technische FileMaker-Brücke bereits vorgibt.
3. Start-Prompt und repräsentative JSON-Daten an den Assistenten geben.
4. Rückfragen des Assistenten beantworten.
5. Den erzeugten Plan öffnen.
6. An drei Punkten zeigen, dass fachliche Entscheidungen sichtbar geworden sind:
   - Datenrichtung,
   - Dirty-State und Fehlerverhalten,
   - Änderungssatz statt vollständiger Datenrückgabe.
7. Plan freigeben oder für die Demo einen bereits vorbereiteten Stand verwenden.

Passender Sprechsatz:

> Die KI beginnt nicht mit dem Programmieren. Zuerst machen wir aus einer Idee einen überprüfbaren Vertrag zwischen FileMaker, Widget und Benutzeroberfläche.

### Live-Demo 2: Ergebnis, Änderung und FileMaker-Brücke — etwa 6 bis 8 Minuten

1. Fertigen Entwicklungsstand im Browser mit `?data=test` öffnen.
2. Tabelle kurz erklären und sortieren.
3. Eine Rechnung auf bezahlt setzen.
4. Auf die Anzeige ungespeicherter Änderungen hinweisen.
5. Änderung speichern und Erfolg zeigen.
6. Optional einen vorbereiteten Fehlerfall zeigen: Änderung bleibt bestehen.
7. Zu FileMaker wechseln.
8. Widget laden beziehungsweise aktualisieren.
9. Eine echte Änderung durchführen und speichern.
10. Kurz die erzeugte Integrationsdokumentation zeigen.

Passender Sprechsatz:

> Der Browser-Mock beschleunigt die Entwicklung. Die eigentliche Abnahme findet trotzdem an der realen Grenze zu FileMaker statt.

## Demo-Vorbereitung und Reserve

Vor dem Vortrag:

- Entwicklungsserver und benötigte Projektstände testen.
- Browser-URL mit `?data=test` als Lesezeichen bereithalten.
- fertigen Stand zusätzlich separat bereithalten, falls die KI-Generierung länger dauert.
- FileMaker-Datei mit bekannten Ausgangsdaten öffnen.
- mindestens eine erfolgreiche und eine absichtlich fehlschlagende Speicherung vorbereiten.
- Schriftgröße in VS Code und Browser für den Beamer erhöhen.
- Benachrichtigungen deaktivieren.
- Produktions-Build vorab erzeugen.

Wenn die Live-Generierung scheitert oder zu lange dauert:

1. Problem nicht im Vortrag debuggen.
2. Auf den vorbereiteten fertigen Projektstand wechseln.
3. Den bereits erzeugten Plan zeigen.
4. Mit Browser- und FileMaker-Demo fortfahren.

## Persönliche Abschlussfragen

Nach der Vorbereitung sollte ich diese Fragen ohne Nachschlagen beantworten können:

- Warum fordert hier das Widget die Daten an, statt dass FileMaker sie ungefragt schickt?
- Warum werden stabile Datensatz-IDs benötigt?
- Warum sendet das Widget nur einen Änderungssatz zurück?
- Wann wird `dirty` zurückgesetzt — und wann ausdrücklich nicht?
- Was ist Aufgabe des Widgets und was bleibt Aufgabe von FileMaker?
- Welche Teile hat der KI-Assistent erzeugt, und welche Entscheidungen habe ich selbst getroffen?

