# EcoRide Baseline Fahrer-PWA + Mitfahrer-PWA

Stand: 2026-09-16

Diese Datei dokumentiert den verifizierten EcoRide-Baseline-Stand fuer Fahrer-PWA und Mitfahrer-PWA. Sie ist die kanonische Produkt-Baseline fuer neue Arbeiten ab diesem Datum.

## Team und Arbeitsablauf

- Morris: Produkt, Smartphone-Realtests, Freigaben.
- GPT: Architektur, UX, Logik, Auftragsdefinition.
- Vero: Orchestrierung, Verifikation, Handover.
- Orion: Implementierung.

Verbindliches Produktprinzip:

> Die Einfachheit gehoert zum Fahrer und Mitfahrer. Die Komplexitaet gehoert ins System.

Ergaenzende Leitlinien:

> Sehen -> verstehen -> antippen -> EcoRide erledigt den Rest.

> Nicht mehr Daten zeigen, sondern bessere Entscheidungen ermoeglichen.

Verbindlicher Arbeitsablauf:

1. Morris
2. GPT
3. Vero
4. echter Orion
5. Vero
6. Morris

Produktcode wird ausschliesslich ueber echten Orion bearbeitet. Vorgeschriebener Handover:

```bash
sudo -n -u orion /usr/local/sbin/nuc15-orion-handoff <AUFTRAGSDATEI>
```

Vor Commit und Push gilt: echte lokale Verifikation durch Orion, anschliessend Vero-Verifikation und Morris-Smartphone-Test.

## Aktuelle Baseline-Commits

### Fahrer-PWA

- Repository: `/srv/shared/projects/ecoride/ecoride-gps-client`
- Remote: `git@github.com:EcoRide-Bremen/ecoride-gps-client.git`
- Commit: `f436b92aa8fe4221580216b32d209569eb417474`
- Commit-Message: `Finalize driver route UI and date-aware departure scheduling`
- Status zur Baseline-Aufnahme: `HEAD == origin/main`, Working Tree clean.

### Mitfahrer-PWA

- Repository: `/srv/shared/projects/ecoride/ecoride-map`
- Remote: `git@github.com:EcoRide-Bremen/ecoride-map.git`
- Commit: `0a0d4813a91d9409f98e9d4ed901c8d85e290470`
- Commit-Message: `Finalize passenger ride UI and date-aware matching`
- Status zur Baseline-Aufnahme: `HEAD == origin/main`, Working Tree clean.

## Fahrer-PWA

### Fahrer-Marker

Die Fahrer-/Live-Position verwendet den Mapbox-Standardmarker.

- Farbe: `#DC2626`
- Mapbox-Standardform.
- Weisser Innenkreis.
- Standardproportionen.
- Standard-Schatten.
- Standard-Anchor und Standard-Offset.
- Keine eigene Sonder-SVG.

### Fahrer-Zielmarker

Das Fahrerziel verwendet ebenfalls den Mapbox-Standardmarker.

- Farbe: `#16A34A`
- Gleiche Markerfamilie wie der Fahrer-Marker.
- Gleiche Groesse.
- Gleiche Form.
- Gleicher weisser Innenkreis.
- Gleicher Anchor und Offset.

### Routenvorschlaege

Vor Auswahl verwenden alle Routenvorschlaege kraeftiges Indigo:

- Schnellste Route: `#4F46E5`
- Kuerzeste Route: `#4F46E5`
- Alternative Route: `#4F46E5`

Die gewaehlte Vorschlagsroute bleibt `#4F46E5`.

Nicht gewaehlte Vorschlagsroute nach Auswahl:

- Light-Mode: `#615AC7`
- Dark-Mode: `#726AEA`

Bedeutung: zurueckgesetzt, nicht gewaehlt und visuell schwaecher als die aktive Route.

### Finale aktive Route

Nach `Fahrt anbieten` verwendet die finale aktive Route in beiden Themes `#4F46E5`. Die fruehere Anthrazit-/Off-White-Loesung ist verworfen.

### Routenbreiten

- Routenvorschlaege: `2.55`
- Finale aktive Route: `2.925`
- Unsichtbare Hit-Zone: `40px`

Die Hit-Zone bleibt unabhaengig von der sichtbaren Linienbreite.

### Routenwahl-Logik

Vor `Fahrt anbieten` ist die Routenwahl frei. Der Fahrer kann zwischen schnellster Route, kuerzester Route und Alternativroute wechseln. Es darf immer nur eine Route-Card visuell aktiv sein.

Nach `Fahrt anbieten` ist die veroeffentlichte Route verbindlich:

- Routenwahl gesperrt.
- Gewaehlte Route bleibt aktiv.
- Andere Route-Cards bleiben neutral.
- Andere Route-Cards duerfen keine zweite EcoRide-Hervorhebung bekommen.
- `selectRoute()` blockiert Wechsel.
- Map-Hit-Layer darf keinen Routenwechsel mehr ausloesen.

Zentrale Sperre:

```js
isRouteSelectionLocked()
```

Beruecksichtigt:

- `activeRideId`
- Status `angeboten`
- Status `aktiv`

Nach `Fahrt stornieren` wird die Routenwahl wieder freigegeben.

### Route-Card UI-State

Korrigierter UI-State:

- Gesperrte, nicht gewaehlte Route-Cards bekommen kein EcoRide-Gruen.
- Kein Sticky-Hover.
- Kein falscher Active-State.
- Nur die tatsaechlich gewaehlte Route bleibt visuell aktiv.
- Gesperrte Card bekommt `.locked`.
- Hover fuer gesperrte, nicht aktive Cards bleibt neutral.

### Datum + Startzeit

Vorhanden:

- natives Datumsfeld.
- bestehendes Zeitfeld `HH:MM`.
- Standarddatum = heute.

Datum + Uhrzeit ergeben zusammen den eindeutigen Startzeitpunkt.

### `departure_time`

Datum + Uhrzeit werden lokal zusammengefuehrt. Danach wird daraus ueber `toISOString()` der bestehende Supabase-`timestamptz`-Pfad bedient. Es gibt keine Schemaaenderung.

### Zeitvalidierung

Schutzebene 1 direkt bei Eingabe, Change und Blur:

- Vergangenes Datum = ungueltig.
- Heute + vergangene Uhrzeit = ungueltig.
- Heute + aktuelle Minute = gueltig.
- Heute + zukuenftige Uhrzeit = gueltig.
- Zukunftsdatum + Uhrzeit = gueltig.

Schutzebene 2 direkt vor `Fahrt anbieten`:

- Datum + Uhrzeit werden erneut geprueft.
- Eine inzwischen vergangene Zeit kann nicht angeboten werden.

Meldung:

```text
Bitte ein gültiges Datum und eine zukünftige Startzeit wählen.
```

## Mitfahrer-PWA

### Eigene Marker

Treffpunkt / Start:

- Mapbox-Standardmarker.
- Farbe: `#2563EB`
- Kraeftiges Blau.

Eigenes Ziel:

- Mapbox-Standardmarker.
- Farbe: `#F97316`
- Kraeftiges Orange.

### Fahrer-Welt

Die visuelle Fahrer-Sprache wurde bewusst aus der Fahrer-PWA uebernommen.

Fahrerposition:

- Mapbox-Standardmarker.
- Farbe: `#DC2626`

Fahrerziel:

- Mapbox-Standardmarker.
- Farbe: `#16A34A`

Fahrerroute:

- Oeffentliche Fahrerroute.
- Farbe: `#4F46E5`

Damit sieht dieselbe Fahrerfahrt in beiden PWAs gleich aus.

### Markergroessen

Alle relevanten Mapbox-Pins haben dieselbe Standarddimension:

- Fahrer rot.
- Fahrerziel gruen.
- Mitfahrer-Treffpunkt blau.
- Mitfahrer-Ziel orange.

Gleiche Markerform, Proportion, weisser Innenkreis, Anchor, Offset und visuelle Groesse.

### Fahrer-Live-Puls

Der rote Fahrer-Marker pulsiert am Fuss beziehungsweise Anchorpunkt.

Der Puls sitzt:

- auf der Kartenoberflaeche.
- exakt auf der Fahrer-GPS-Koordinate.
- unter dem roten Pin.
- nicht um den Marker-Kopf.

Kurz:

> Der rote Mapbox-Pin steht auf dem Pulsring.

### Dark-Mode-Puls

Der Dark-Mode-Puls wurde technisch repariert. Der Pulse-Layer wird direkt ueber Paint-Werte animiert:

- `circle-radius`
- `circle-stroke-width`
- `circle-opacity`
- `circle-stroke-color`

Der Timer laeuft etwa alle `180ms`. Der Puls laeuft auch bei stillstehender Karte; Zoom oder Pan sind nicht noetig.

### Weisser Zusatzring entfernt

Frueher lagen zwei Ring-Layer uebereinander:

- `${LAYER_POINTS}-pulse`
- `${LAYER_POINTS}-ring`

Der zusaetzliche Ring wurde fuer Live-Fahrer entfernt.

Aktueller Zustand:

- Live-Fahrer: roter Puls.
- Stale Fahrer: Ring-Layer darf weiterhin verwendet werden.

Kein weisser statischer Grundring mehr unter dem roten Puls.

### Gruener Zielmarker

Der gruene Fahrer-Zielmarker `#16A34A` hat:

- weissen Innenkreis.
- keinen zusaetzlichen weissen Aussenring.
- keinen Puls.

Light-Mode und Dark-Mode sind konsistent.

### Blauer Empfehlungsring

Der blaue Ring bleibt bewusst bestehen.

Geplante Bedeutung:

> EcoRide-Empfehlungsindikator

Noch keine echte Empfehlungslogik umgesetzt.

Spaeter moegliche Kategorien:

- Beste Empfehlung.
- Sehr passend.
- Weitere passende Fahrt.

### Trefferanzeige / Accordion V1

Die fruehere grosse Trefferkarte wurde strukturell umgebaut.

Neue Architektur:

- kompakte Trefferzeile.
- ganze Zeile antippbar.
- Chevron.
- Detailbereich aufklappbar.
- maximal ein Treffer gleichzeitig offen.
- erneutes Antippen schliesst denselben Treffer.
- Oeffnen eines zweiten Treffers schliesst den ersten.

Neue Struktur:

- `paxRideResults`
- `openPaxResultDriverKey`
- `buildPaxRideResult(...)`
- `renderPaxResults(...)`
- `selectPaxResult(...)`
- `togglePaxResultDetails(...)`
- `clearPaxResults()`

Geschlossene Trefferzeile zeigt:

- Fahrername.
- Chevron.

Nicht mehr geschlossen sichtbar:

- Entfernung.
- Plaetze.
- Preis.

Geoeffneter Detailbereich enthaelt vorhandene und verstaendlich darstellbare Informationen wie:

- Entfernung zum Treffpunkt.
- freie Plaetze.
- Preis pro Platz.
- Abfahrtszeit.
- Fahrerziel.
- Ride-Status.
- vorhandene Routendaten.

Es werden keine technischen Rohdaten angezeigt.

### Matching-Sektor

Matching ist ein eigener Sektor beziehungsweise Baustein.

Aufbau:

```text
Matching
1 Fahrer gefunden (2 km Umkreis)
Trefferzeile / Accordion
```

Der Sektor verwendet bestehende EcoRide-Proportionen.

### Statuszeile

Die fruehere gruene Erfolgsschrift wurde angepasst:

- Text neutral/dunkel statt gruen.
- Gruener Haken entfernt.
- Keine Success-Optik.
- Statuszeile gehoert zum Matching-Sektor.

### Trefferbereich / Hintergrund

Der alte schwere Anthrazit-Look wurde angepasst.

Ziel:

- EcoRide-nahe Gestaltung.
- Indigo-orientierte Designsprache.
- Chevron Indigo.
- Kein schwerer fremder Anthrazitblock.

### Startzeit-/Suchsektor

Der Startzeit-/Suchbereich wurde zu einem zusammenhaengenden Baustein erweitert.

Im selben Sektor liegen:

- Datum.
- Startzeit.
- Eingabefeld.
- Zeitfenster.
- `Fahrer finden`.
- kleiner Hinweistext.

Der Matching-Sektor darunter bleibt separat.

### Datum + Startzeit

Vorhanden:

- Datumsfeld `paxDateInput`.
- Zeitfeld `paxTimeInput`.
- Datum + Startzeit gemeinsam im Startzeit-Sektor.
- Standarddatum = heute.

### Zeitlogik

`paxTimeMin` wird aus Datum + Uhrzeit als Abstand zum aktuellen Zeitpunkt berechnet.

Schutzebene 1:

- `commitPaxTimeInput(...)` prueft Datum, Uhrzeit, Vergangenheit und Zukunft.

Schutzebene 2:

- Direkt vor `Fahrer finden` ruft `runPaxSearch()` erneut `commitPaxTimeInput(true)` auf.
- Dadurch wird verhindert, dass eine inzwischen vergangene Suchzeit weiterverwendet wird.

### Vollstaendiges Datum+Uhrzeit-Matching

Wichtiger Bugfix:

Frueher konnte folgender Fall matchen:

- Mitfahrer: `13.09.2026 16:00`
- Fahrer: `14.09.2026 16:00`

Das war falsch.

Jetzt wird verglichen:

```text
vollstaendiger Mitfahrer-Suchzeitpunkt gegen vollstaendiges ride.departure_time
```

Regel:

- Die Zeitdifferenz muss innerhalb `PAX_MATCH_TIME_SLACK` liegen.

Beispiele:

- Mitfahrer `13.09.2026 16:00`, Fahrer `14.09.2026 16:00` -> kein Match.
- Mitfahrer `14.09.2026 16:00`, Fahrer `14.09.2026 16:00` -> Match moeglich.

## EcoRide Port-Mapping

Read-only gepruefter Status: korrekt, kein Aenderungsbedarf.

### Mitfahrer-PWA Routing

- Oeffentlicher Pfad: `/map/`
- Tailscale Serve Ziel: `http://127.0.0.1:8081`
- Lokaler App-Pfad: `/`
- Repo: `/srv/shared/projects/ecoride/ecoride-map`

### Fahrer-PWA Routing

- Oeffentlicher Pfad: `/gps/`
- Tailscale Serve Ziel: `http://127.0.0.1:8082`
- Lokaler App-Pfad: `/`
- Repo: `/srv/shared/projects/ecoride/ecoride-gps-client`

Wichtig:

- Direkt lokal `8081/map/` oder `8082/gps/` kann `404` liefern.
- Das ist korrekt.
- Die Prefixe `/map/` und `/gps/` werden durch Tailscale Serve erzeugt.

## Konzepte und offene Themen

### Preislogik gemeinsame Strecke

Konzept, noch nicht umgesetzt:

1. Mitfahrer gibt gewuenschten Ausstiegspunkt an.
2. EcoRide berechnet die gemeinsame Strecke.
3. EcoRide berechnet daraus einen Preisvorschlag.
4. Preis wird auf den naechsten `0,10 EUR`-Schritt zugunsten des Fahrers aufgerundet.
5. Fahrer kann akzeptieren, Gegenvorschlag machen oder ablehnen.
6. Mitfahrer kann Gegenvorschlag akzeptieren oder ablehnen.

### Empfehlungslogik

Noch nicht implementiert.

Spaeter moegliche Faktoren:

- Entfernung zum Treffpunkt.
- Zeitabweichung.
- gemeinsame Strecke.
- Routennaehe.
- Umweg.
- freie Plaetze.
- Zielrichtung.
- Fahrerbewertung.
- Zuverlaessigkeit.

Die Komplexitaet bleibt im System. Nutzer sehen nur verstaendliche Kategorien.

### Match-Lifecycle nach Ablauf der Suchzeit

Read-only Diagnose abgeschlossen, aber noch nicht umgesetzt.

Aktueller Stand:

- `runPaxSearch()` erzeugt einen Match-Snapshot.
- Es gibt keinen automatischen Recheck nach Ablauf.
- `1 Fahrer gefunden` kann stehen bleiben.
- `paxRideResults` wird nicht automatisch invalidiert.
- Blauer Ring kann verschwinden.
- Fahrerroute und Ziel koennen weiter sichtbar bleiben.
- Marker und Puls koennen durch Fahrer-Refresh wieder erscheinen.

Festgestellt:

> Live-Fahrer, oeffentliche Ride und Match-Gueltigkeit sind aktuell getrennte Zustaende.

Noch nicht umgesetzt:

- Lifecycle-Watcher.
- automatischer Match-Recheck.
- automatischer Statuswechsel.

### Verspaeteter Fahrtstart

Beispiel:

- geplant `19:00`
- `19:05` noch okay.
- `19:10` keine Bewegung.
- `19:20` Fahrer startet moeglicherweise trotzdem.

Noch keine harte Auto-Storno-Regel beschlossen.

Aktuelle Richtung:

- nicht automatisch stornieren.
- moeglicher spaeterer Zwischenstatus `verspaetet`.
- moeglicher spaeterer Status `abgelaufen`.

### Auto-Storno

Noch nicht umgesetzt. Keine automatische Stornierung nur aufgrund Zeitueberschreitung.

### Fahrer-PWA Designangleichung

Mittelfristiges Ziel:

> Fahrer-PWA gestalterisch an Mitfahrer-PWA angleichen.

Die Mitfahrer-PWA ist visuelle Referenz. Zukuenftige Fahrer-UI soll dieselbe Sektor-/Baustein-Sprache verwenden.

## Dauerregel fuer neue Arbeiten

Jede neue Arbeit startet von diesen verifizierten Baselines:

- Fahrer-PWA: `f436b92aa8fe4221580216b32d209569eb417474`
- Mitfahrer-PWA: `0a0d4813a91d9409f98e9d4ed901c8d85e290470`

Vor jeder Umsetzung:

1. echten Dateistand pruefen.
2. kleinen Scope definieren.
3. echten Orion verwenden.
4. Vero verifiziert.
5. Morris Smartphone-Test.
6. erst danach Commit / Push.
