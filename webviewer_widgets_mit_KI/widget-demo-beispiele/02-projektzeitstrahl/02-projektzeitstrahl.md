# Beispiel 2: Interaktiver Projektzeitstrahl

Persönliche Arbeitsanleitung für die Live-Demo im Vortrag **„Entwicklung von WebViewer-Widgets in Zeiten von KI“**.

Dieses Beispiel ist der kurze Kontrast zum editierbaren Rechnungswidget. Es zeigt, dass ein Widget nicht immer Daten bearbeiten muss. Hier steht die visuelle Aufbereitung im Vordergrund: FileMaker liefert den aktuellen Projektkontext, das Widget macht zeitliche Zusammenhänge sichtbar und meldet eine Auswahl an FileMaker zurück.

## Ziel des Beispiels

Ein Projekt wird als horizontaler Zeitstrahl dargestellt. Aufgaben sind nach Phasen gruppiert und anhand ihres Start- und Enddatums positioniert. Der Benutzer kann zwischen Tages- und Wochenansicht wechseln und einen Balken anklicken. Das Widget meldet die stabile ID der Auswahl an FileMaker.

Das Publikum soll dabei verstehen:

- Dasselbe Template unterstützt unterschiedliche Datenrichtungen und Bedienmodelle.
- FileMaker kann Daten aktiv in ein bereits geladenes Widget übertragen.
- Ein Widget kann als interaktive Visualisierung dienen, ohne selbst fachliche Daten zu bearbeiten.
- Für eine Auswahl werden stabile IDs und ein klar benanntes Ereignis verwendet.
- Auch ein visuelles Widget braucht Lade-, Leer- und Fehlerzustände sowie konkrete Abnahmekriterien.

## Projektidentität

| Eigenschaft | Wert |
| --- | --- |
| Anzeigename | Projektzeitstrahl |
| FileMaker-Datei | `Projektzeitstrahl.fmp12` |
| npm-Paketname | `projektzeitstrahl` |
| Widget-ID | `projektzeitstrahl` |
| Hauptrichtung der Daten | FileMaker schiebt den aktuellen Projektkontext ins Widget |
| Bearbeitbar | Nein |
| Rückmeldung | Auswahlereignis mit Projekt- und Datensatz-ID |

Die FileMaker-Datei wird nicht automatisch vom Assistenten verändert. Die konkrete Reaktion auf eine Auswahl — zum Beispiel Navigation, Öffnen eines Popovers oder Setzen eines Filters — wird im FileMaker-Projekt festgelegt.

## Fachlicher Ablauf

1. Der Benutzer öffnet oder wechselt in FileMaker zu einem Projekt.
2. FileMaker baut die JSON-Nutzlast für dieses Projekt auf.
3. FileMaker übergibt die Nutzlast mit `fmWidgetLoad(payload)` an den Web Viewer.
4. Das Widget berechnet aus Datumswerten Position und Breite der Balken.
5. Der Benutzer wechselt bei Bedarf zwischen Tages- und Wochenansicht.
6. Der Benutzer klickt eine Aufgabe oder einen Meilenstein an.
7. Das Widget markiert die Auswahl und sendet `select-record` an `fmWidget_handleEvent`.
8. FileMaker führt die projektspezifische Aktion aus und bestätigt oder verwirft den Aufruf.

## Datenfluss

```text
Aktueller Projektkontext in FileMaker
                │
                │ fmWidgetLoad(payload)
                ▼
        Interaktiver Zeitstrahl
                │
                │ Benutzer wählt einen Balken
                ▼
 fmWidget_handleEvent: select-record
                │
                ▼
 Projektspezifische FileMaker-Aktion
```

## Repräsentative Beispieldaten

```json
{
  "meta": {
    "title": "Website Relaunch",
    "projectId": "P-1007",
    "rangeStart": "2026-09-01",
    "rangeEnd": "2026-10-15",
    "locale": "de-DE"
  },
  "data": [
    {
      "recordId": "A-101",
      "title": "Anforderungen abstimmen",
      "phase": "Planung",
      "startDate": "2026-09-01",
      "endDate": "2026-09-05",
      "status": "done",
      "type": "task"
    },
    {
      "recordId": "A-102",
      "title": "Gestaltung freigeben",
      "phase": "Konzeption",
      "startDate": "2026-09-08",
      "endDate": "2026-09-16",
      "status": "active",
      "type": "task"
    },
    {
      "recordId": "A-103",
      "title": "Widget umsetzen",
      "phase": "Umsetzung",
      "startDate": "2026-09-17",
      "endDate": "2026-10-02",
      "status": "open",
      "type": "task"
    },
    {
      "recordId": "A-104",
      "title": "Abnahme durchführen",
      "phase": "Abschluss",
      "startDate": "2026-10-05",
      "endDate": "2026-10-09",
      "status": "open",
      "type": "task"
    },
    {
      "recordId": "M-105",
      "title": "Go-live",
      "phase": "Abschluss",
      "startDate": "2026-10-12",
      "endDate": "2026-10-12",
      "status": "open",
      "type": "milestone"
    }
  ]
}
```

Wichtige Regeln für die Daten:

- `projectId` und `recordId` sind stabile FileMaker-IDs.
- Datumswerte werden im ISO-Format `YYYY-MM-DD` übertragen.
- `rangeStart` und `rangeEnd` bestimmen den sichtbaren Gesamtzeitraum.
- `startDate` darf nicht nach `endDate` liegen.
- Ein Meilenstein hat in diesem Beispiel dasselbe Start- und Enddatum.
- Mögliche Statuswerte sind `open`, `active` und `done`.
- Die Phasenbezeichnung dient der Gruppierung; ihre Reihenfolge entspricht zunächst dem Auftreten in den Daten.

## Ereignis bei Auswahl

Beim Klick auf einen Balken sendet das Widget:

```json
{
  "event": "select-record",
  "data": {
    "projectId": "P-1007",
    "recordId": "A-103"
  }
}
```

Die konkrete FileMaker-Aktion ist bewusst nicht Teil des allgemeinen Widget-Vertrags. Im Demo-Projekt kann `fmWidget_handleEvent` zum Beispiel zu einem passenden Datensatz navigieren. Es kann ebenso ein Popover öffnen oder einen Filter setzen. Diese Entscheidung gehört zur FileMaker-Anwendung.

## Fertiger Start-Prompt für den KI-Assistenten

Den folgenden Text kann ich zusammen mit den Beispieldaten als fachlichen Auftrag übergeben.

```text
Ich möchte ein FileMaker-Web-Viewer-Widget mit dem Namen „Projektzeitstrahl“ entwickeln.

FileMaker übergibt dem Widget jeweils den aktuellen Projektkontext. Das Widget soll die Aufgaben eines kleinen Projekts mit maximal etwa 30 Einträgen als horizontalen Zeitstrahl darstellen. Die Aufgaben werden nach Phase gruppiert. Ihre horizontale Position und Breite ergeben sich aus Start- und Enddatum innerhalb des übergebenen Gesamtzeitraums. Meilensteine sollen als einzelner Zeitpunkt erkennbar sein.

Der Benutzer kann zwischen einer Tages- und einer Wochenansicht wechseln und horizontal scrollen. Beim Klick auf eine Aufgabe oder einen Meilenstein wird der Eintrag hervorgehoben und ein Ereignis „select-record“ mit projectId und recordId an FileMaker gesendet. Das Widget bearbeitet keine fachlichen Daten, besitzt keinen Speichern-Button und bleibt deshalb immer dirty = false.

Bitte plane außerdem einen Ladezustand, einen Leerzustand und eine verständliche Validierungsanzeige für ungültige oder außerhalb des Bereichs liegende Datumswerte. Verwende die bestehenden Schnittstellen und festen Skriptnamen des Templates. Erfinde keine zusätzliche FileMaker-Brücke. Beginne mit React, CSS Grid und nativer Scrollfunktion und füge keine Zeitstrahlbibliothek hinzu, solange diese Anforderungen damit übersichtlich umgesetzt werden können.

Bitte beginne mit Rückfragen, falls Informationen fehlen. Erstelle danach einen konkreten Plan und warte auf meine ausdrückliche Freigabe, bevor du Dateien änderst. Lege den freigegebenen Plan unter docs/PLAN.md ab und dokumentiere die FileMaker-Integration unter docs/FM-INTEGRATION.md. Die FileMaker-Datei selbst soll nicht automatisch bearbeitet werden.
```

## Erwartete Rückfragen des Assistenten

Der Assistent sollte mindestens klären:

- Wie viele Aufgaben werden typischerweise angezeigt?
- Welche zeitlichen Auflösungen werden benötigt?
- Wie werden Aufgaben, Meilensteine und Status visuell unterschieden?
- Soll die Auswahl nur markiert oder direkt von FileMaker verarbeitet werden?
- Wie behandelt die Oberfläche ungültige und außerhalb des Bereichs liegende Daten?
- Soll die Phase aus den Daten übernommen oder separat sortiert werden?
- Was soll beim Laden eines neuen Projekts mit der bisherigen Auswahl geschehen?

Empfohlene Entscheidungen für diese Demo:

- maximal etwa 30 Einträge,
- Tages- und Wochenansicht,
- Auswahl wird beim Laden eines neuen Projekts zurückgesetzt,
- außerhalb des Zeitraums liegende Balken werden am Rand abgeschnitten,
- vollständig ungültige Datensätze werden nicht gezeichnet und in einer verständlichen Meldung aufgeführt,
- keine externe Zeitstrahlbibliothek.

## UI-Anforderungen

### Normalzustand

- Projekttitel und sichtbarer Datumsbereich
- Umschalter **Tag / Woche**
- feste linke Spalte mit Titel beziehungsweise Phase
- horizontal scrollbarer Zeitbereich
- Aufgaben als Balken, Meilensteine als deutlich unterscheidbare Markierung
- Statusunterschiede über Farbe und zusätzlich über Text oder Symbol erkennbar

### Auswahl

- angeklickter Eintrag ist eindeutig hervorgehoben
- Hervorhebung ist nicht ausschließlich von einer Farbänderung abhängig
- die Auswahl selbst ist keine fachliche Datenänderung

### Ladezustand

- ruhige Ladeanzeige, solange noch kein Projekt übergeben wurde

### Leerzustand

- Text wie: **Für dieses Projekt sind keine Aufgaben vorhanden.**

### Fehler- und Validierungszustand

- verständlicher Hinweis bei ungültigem Gesamtzeitraum
- ungültige Einzelaufgaben werden benannt, ohne die gesamte Oberfläche abstürzen zu lassen
- technische Details bleiben in der Entwicklungskonsole, nicht in der Benutzeroberfläche

## Zustandsregeln

Das Widget bearbeitet keine fachlichen Daten.

- `dirty` bleibt immer `false`.
- Ein Wechsel von Tag zu Woche ist nur ein Anzeigezustand.
- Horizontales Scrollen ist nur ein Anzeigezustand.
- Die Auswahl eines Balkens ist ein Interaktionszustand, aber keine ungespeicherte Änderung.
- Nach `fmWidgetLoad(payload)` wird die bisherige Auswahl zurückgesetzt.

Falls FileMaker den Zustand abfragt, meldet `fmWidget_reportState`:

```json
{
  "dirty": false
}
```

## Technischer Rahmen

- Ausgangspunkt ist das Repository `fm-starter-ai` beziehungsweise ein daraus erstelltes Projekt.
- Projektanlage mit dem Skill `aga-fm-start`.
- Planung und Umsetzung im Projekt mit dem dort enthaltenen Skill `aga-fm-widget`.
- React und CSS Grid für die Darstellung verwenden.
- Native horizontale Scrollfunktion verwenden.
- Für maximal etwa 30 Einträge keine Zeitstrahl-, Gantt- oder State-Management-Bibliothek hinzufügen.
- Entwicklungsmodus mit Beispieldaten: `http://localhost:5173/?data=test`
- Beispieldaten dürfen nicht im Produktions-Build enthalten sein.

## Verbindliche Schnittstelle zum Template

### Skripte in FileMaker

- `fmWidget_getData`
- `fmWidget_handleEvent`
- `fmWidget_upload`
- `fmWidget_reportData`
- `fmWidget_reportState`

In diesem Beispiel schiebt FileMaker die Daten aktiv in das Widget. Deshalb wird `fmWidget_getData` für den Hauptablauf nicht benötigt. `fmWidget_handleEvent` verarbeitet die Auswahl. `fmWidget_reportState` wird nur benötigt, falls FileMaker den Zustand ausdrücklich abfragt.

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

Für die Übergabe eines neuen Projektkontexts verwendet FileMaker `fmWidgetLoad(payload)`.

### FMGofer-Hülle

Der Auswahlaufruf des Widgets an FileMaker enthält:

- `parameter`
- `callbackName`
- `promiseID`

FileMaker beendet einen erfolgreichen Aufruf über den übergebenen Callback mit `$promiseID`, Nutzlast und `False`. Ein Fehler wird mit `$promiseID`, Fehlermeldung und `True` zurückgegeben.

## FileMaker-Integration

### Vorbereitung

1. Web Viewer auf dem Projektlayout anlegen.
2. Option **„JavaScript darf FileMaker-Skripte ausführen“** aktivieren.
3. Produktionsdatei des Widgets in den vom Template vorgesehenen Ablageort übernehmen.
4. Die vom Projekt erzeugte Datei `docs/FM-INTEGRATION.md` als konkrete Integrationsanleitung verwenden.

### Projektkontext an das Widget übergeben

Ein projektspezifisches FileMaker-Skript:

1. ermittelt das aktuelle Projekt,
2. baut die vereinbarte JSON-Nutzlast,
3. ruft im Web Viewer `fmWidgetLoad` mit dieser Nutzlast auf,
4. wird bei Projektwechsel, Layoutöffnung oder einer fachlich sinnvollen Aktualisierung erneut ausgeführt.

Der Name dieses projektspezifischen Skripts kann in der eigenen FileMaker-Lösung passend gewählt werden. `fmWidgetLoad` selbst bleibt als globaler Adapter unverändert.

### `fmWidget_handleEvent`

Das feste Skript:

1. liest `parameter.event` und `parameter.data`,
2. behandelt im Beispiel `select-record`,
3. validiert `projectId` und `recordId`,
4. führt die im Demo-Projekt gewählte FileMaker-Aktion aus,
5. bestätigt den Aufruf über den Callback oder gibt eine verständliche Ablehnung zurück.

Die Navigation zu einem Datensatz ist eine mögliche Demo-Implementierung, aber keine allgemeine Pflicht des Templates.

## Arbeitsablauf zur Vorbereitung

### 1. Projekt erzeugen

Den installierten Skill verwenden:

```text
Verwende den Skill aga-fm-start, um ein FileMaker-Web-Viewer-Widget zu erstellen.
```

Als Projektname **Projektzeitstrahl** und als Ziel einen neuen, eindeutig benannten Ordner wählen.

### 2. Projekt öffnen und lokalen Skill verwenden

Im erzeugten Projekt den enthaltenen Skill `aga-fm-widget` für Planung und Entwicklung verwenden. Start-Prompt und Beispieldaten aus dieser Datei übergeben.

### 3. Plan prüfen

Vor der Freigabe kontrollieren:

- Ist die Datenrichtung korrekt: FileMaker schiebt den Projektkontext?
- Ist das Widget ausdrücklich nicht editierbar?
- Wird `dirty` immer als `false` behandelt?
- Werden Tages- und Wochenansicht sowie horizontales Scrollen geplant?
- Werden stabile Projekt- und Datensatz-IDs übertragen?
- Ist die Reaktion in FileMaker als projektspezifische Entscheidung beschrieben?
- Sind Lade-, Leer- und Validierungszustände geplant?
- Wird auf eine unnötige Zeitstrahlbibliothek verzichtet?
- Bleiben die festen Skript- und Adapter-Namen unverändert?
- Werden Beispieldaten vom Produktions-Build ausgeschlossen?

Erst danach die Umsetzung ausdrücklich freigeben.

### 4. Im Browser prüfen

Mit den Beispieldaten testen:

1. Start im Mock-Modus.
2. Phasengruppierung und zeitliche Positionierung prüfen.
3. Zwischen Tag und Woche wechseln.
4. Horizontal scrollen.
5. Aufgabe und Meilenstein auswählen.
6. Ereignisinhalt mit `projectId` und `recordId` prüfen.
7. Leere Datenmenge testen.
8. ungültiges Datum und ungültigen Gesamtzeitraum testen.
9. sicherstellen, dass alle Interaktionen `dirty = false` lassen.

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

Die projektbezogene `docs/FM-INTEGRATION.md` Schritt für Schritt umsetzen. Danach Projektwechsel und Auswahlereignis mit echten FileMaker-Daten testen.

## Abnahmekriterien

- `fmWidgetLoad(payload)` ersetzt den angezeigten Projektkontext vollständig.
- Beim Laden eines neuen Projekts wird die bisherige Auswahl zurückgesetzt.
- Aufgaben werden nach Phase gruppiert.
- Position und Breite der Aufgaben ergeben sich nachvollziehbar aus den Datumswerten.
- Ein Eintrag mit gleichem Start- und Enddatum wird als Meilenstein dargestellt.
- Tag/Woche-Umschaltung und horizontales Scrollen funktionieren.
- Ein Klick hebt den Eintrag sichtbar hervor.
- Das Ereignis enthält stabile `projectId`- und `recordId`-Werte.
- Die ausgewählte FileMaker-Aktion wird ausgeführt oder ein Fehler verständlich zurückgemeldet.
- Ungültige Daten führen nicht zu einem Absturz der gesamten Ansicht.
- Lade- und Leerzustand sind eindeutig.
- Das Widget besitzt keinen Speichern-Button und bleibt `dirty = false`.
- Der Produktions-Build enthält keine Beispieldaten.
- Type-Check, Lint, Tests und Build laufen erfolgreich durch.

## Ablauf der Live-Demo

Dieses Beispiel ist als kurze, bereits vorbereitete Kontrastdemo gedacht. Es sollte nicht dieselbe Entwicklungsstrecke wie das Rechnungsbeispiel noch einmal vollständig wiederholen.

### Kurz-Demo — etwa 4 bis 6 Minuten

1. Mit einem Satz an das Rechnungsbeispiel anknüpfen: Hier gibt FileMaker die Datenrichtung vor.
2. Den Start-Prompt oder den relevanten Ausschnitt aus `docs/PLAN.md` zeigen.
3. Im Browser den fertigen Zeitstrahl mit Mock-Daten öffnen.
4. Zwischen Tag und Woche wechseln.
5. Horizontal scrollen und einen Meilenstein zeigen.
6. Eine Aufgabe anklicken und die stabile ID im Ereignis erläutern.
7. Zu FileMaker wechseln.
8. Zwischen zwei vorbereiteten Projekten wechseln und zeigen, dass FileMaker jeweils neue Daten mit `fmWidgetLoad` übergibt.
9. Einen Balken anklicken und die projektspezifische FileMaker-Reaktion zeigen.
10. Abschließend darauf hinweisen, dass keine Daten im Widget verändert wurden und deshalb kein Dirty-State entsteht.

Passender Sprechsatz:

> Beim Rechnungswidget fragt die Oberfläche Daten an und gibt Änderungen zurück. Hier schiebt FileMaker den aktuellen Kontext in eine interaktive Visualisierung — dieselbe Brücke, aber ein anderes fachliches Muster.

## Demo-Vorbereitung und Reserve

Vor dem Vortrag:

- zwei kleine Projekte mit deutlich unterschiedlichen Zeiträumen vorbereiten,
- Browser-URL mit `?data=test` als Lesezeichen bereithalten,
- Tages- und Wochenansicht auf dem Beamer prüfen,
- horizontale Breite so wählen, dass Scrollen sichtbar, aber nicht mühsam ist,
- einen gut sichtbaren Meilenstein vorbereiten,
- eine FileMaker-Aktion für `select-record` festlegen und testen,
- Schriftgrößen und Kontrast für helle Projektion prüfen,
- fertigen Produktions-Build vorab erzeugen.

Wenn die FileMaker-Reaktion nicht funktioniert:

1. Browser-Demo vollständig zeigen.
2. Das erwartete `select-record`-JSON öffnen.
3. Anhand der stabilen IDs erklären, welche Information FileMaker erhält.
4. Den vorbereiteten Plan oder die Integrationsdokumentation zeigen.

## Persönliche Abschlussfragen

Nach der Vorbereitung sollte ich diese Fragen ohne Nachschlagen beantworten können:

- Warum schiebt FileMaker in diesem Beispiel die Daten aktiv in das Widget?
- Warum ist ein Wechsel zwischen Tag und Woche kein Dirty-State?
- Wie werden Position und Breite eines Balkens aus Datumswerten bestimmt?
- Warum übermittelt die Auswahl sowohl `projectId` als auch `recordId`?
- Welche Reaktion gehört zum allgemeinen Widget und welche zur konkreten FileMaker-Lösung?
- Warum ist für dieses kleine Beispiel keine Gantt- oder Zeitstrahlbibliothek nötig?

