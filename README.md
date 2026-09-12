# MietAkte — Belegverwaltung für vermietete Wohnungen (SYNIUM)

Web-App zum Erfassen von Renovierungs- und Instandhaltungsbelegen: Beleg fotografieren,
Betrag/MwSt/Datum/Händler **lokal auf dem Gerät** per Texterkennung (Tesseract-OCR als
WebAssembly) auslesen, prüfen, einer Wohnung zuordnen und in Dropbox ablegen. Es wird
**kein KI-Dienst** angebunden – Belege verlassen das Gerät nur Richtung eigene Dropbox. Übersicht mit Filtern, Summen je Wohnung
und Excel-Export fürs Finanzamt.

> **Dateien:** `index.html` (die App) + Ordner `Logo/` (muss neben `index.html` liegen).
> Einzeldatei-App ohne Build. Von cdnjs/jsdelivr werden nachgeladen: SheetJS (Excel-Export),
> Tesseract.js samt Sprachdaten (Texterkennung, einmalig ca. 15 MB, danach im Browser
> zwischengespeichert) und pdf.js (nur bei PDF-Belegen).

## Erste Einrichtung

Die App braucht nur den Dropbox-Zugang, der unter **Einstellungen** eingetragen wird und nur
im Browser (localStorage) gespeichert bleibt:

### 1. Dropbox App-Key (einmalig, ca. 3 Minuten)
1. <https://www.dropbox.com/developers/apps> → **Create app** → *Scoped access* →
   *App folder* (Belege landen dann in `Apps/<Appname>/…`) oder *Full Dropbox* → Name vergeben.
2. Reiter **Permissions** aktivieren: `account_info.read`, `files.metadata.read`,
   `files.metadata.write`, `files.content.read`, `files.content.write`,
   `sharing.read`, `sharing.write` → unten **Submit** drücken (wird leicht vergessen).
3. Reiter **Settings** → **App key** kopieren und in MietAkte unter Einstellungen eintragen.
4. **Verbinden (Code eingeben)** drücken → „Bei Dropbox anmelden" → Zugriff erlauben →
   Dropbox zeigt einen Code → Code in MietAkte einfügen → **Verbinden**.
   Dieses Verfahren braucht **keine Redirect-URI** und funktioniert auf dem PC (auch bei
   Doppelklick auf `index.html`) und auf dem Handy.

Alternative „Verbinden (Weiterleitung)": nur wenn die App über `https://` oder
`http://localhost` läuft **und** in der App Console unter **Settings → Redirect URIs** genau
die Adresse eingetragen ist, die MietAkte in den Einstellungen anzeigt.

**Typische Fehlerbilder**
| Meldung | Ursache / Abhilfe |
|---|---|
| „Die Weiterleitung funktioniert nur über https://…" | App per Datei geöffnet → Code-Verfahren nutzen |
| Dropbox-Seite „Invalid redirect_uri" | Adresse in der App Console fehlt/abweichend → Code-Verfahren nutzen oder URI exakt eintragen |
| „App-Key unbekannt" | Falscher/unvollständiger App key (nicht das App secret) |
| „Code ist ungültig oder abgelaufen" | Anmeldung erneut öffnen, neuen Code innerhalb weniger Minuten einfügen |
| Upload-Fehler „missing_scope" | Permissions in der App Console nicht aktiviert oder ohne **Submit** → nachholen, dann Trennen und neu verbinden |

Die Anmeldung bleibt über ein Refresh-Token dauerhaft bestehen (OAuth 2 mit PKCE, kein App-Secret).

### 2. Texterkennung
Keine Einrichtung nötig. Unter Einstellungen kann die Sprache (Deutsch bzw. Deutsch + Englisch)
gewählt und das Erkennungsmodul vorab geladen werden, damit der erste Beleg schneller geht.

## Bedienung
Die Oberfläche passt sich der Bildschirmgröße an: auf dem Handy mit unterer Tab-Leiste, am
Desktop (ab 1100 px Breite) mit linker Navigationsleiste und mehrspaltigen Ansichten; ab 1700 px
(23-Zoll-Monitore und größer) mit größerer Schrift, drei Spalten in den Einstellungen und
bis zu 2100 px Inhaltsbreite.

- **Wohnungen**: Mietobjekte anlegen (Name, Adresse, Dropbox-Ordnername). Ordner werden
  unterhalb des Basisordners (Standard `/MietAkte`) angelegt.
- **Erfassen**: Foto aufnehmen oder Datei (JPG/PNG/PDF) wählen → Text wird auf dem Gerät erkannt,
  daraus werden Summe, MwSt, Datum, Händler und Belegnummer regelbasiert ermittelt und blau
  markiert → prüfen/korrigieren → Wohnung und Kategorie wählen → **Speichern & hochladen**.
  „Erkannten Text anzeigen" zeigt den Rohtext zum Gegenprüfen. Tipp: Beleg gerade, hell und
  scharf fotografieren; bereits erfasste Händlernamen werden beim nächsten Mal wiedererkannt.
  Das Bild wird auf max. 1800 px verkleinert und als
  `JJJJ-MM-TT_Händler_Betrag,xxEUR.jpg` im Wohnungsordner abgelegt.
- **Belege**: Liste mit Filter (Wohnung, Kategorie, Zeitraum/Jahr, Suche), Summen je Wohnung,
  Bearbeiten/Löschen per Tipp auf einen Beleg. **Excel-Export** erzeugt eine Aufstellung
  (bei „Alle Wohnungen": ein Blatt je Wohnung + Gesamtblatt + Kategorien-Summen).
- Schlägt ein Upload fehl (offline, Dropbox getrennt), bleibt der Beleg mit dem Bild lokal
  gespeichert (IndexedDB) und kann über **Ausstehende hochladen** nachgeholt werden.

## Datenhaltung
- Belegtabelle und Wohnungen liegen im localStorage des Browsers und werden bei verbundener
  Dropbox zusätzlich als `mietakte-daten.json` im Basisordner gespiegelt (Abgleich beim Start,
  beim Zurückkehren in die App und nach jeder Änderung). So funktioniert die App auf mehreren
  Geräten mit derselben Dropbox.
- **Sicherung (JSON)** / **Wiederherstellen** unter Einstellungen.
- Die Speicherschlüssel sind nach Benutzer-ID gekapselt (`Session.userId`), damit später
  ein Login für weitere Nutzer ergänzt werden kann.

## Auf dem Handy nutzen (Android / iPhone)
Am besten läuft MietAkte als Webseite über `https://`; dann kann sie auf Android („Zum
Startbildschirm hinzufügen") und iPhone (Teilen → „Zum Home-Bildschirm") wie eine App abgelegt
werden. Kamera, Texterkennung und Dropbox funktionieren dort ohne weitere Einrichtung.

**Option A – GitHub Pages (kostenlos, empfohlen):** Das SYNIUM-Repo ist privat, dort ist
Pages nicht verfügbar. Stattdessen ein eigenes, öffentliches Repo (z. B. `mietakte`) nur mit
`index.html` + `Logo/` anlegen, unter *Settings → Pages* den Branch `main` veröffentlichen →
Adresse `https://<user>.github.io/mietakte/`. Die Dateien enthalten keine Schlüssel;
alle Zugänge liegen nur im Browser des Handys.

**Option B – Android ohne Hosting:** Ordner `MietAkte` (mit `Logo/`) auf das Handy kopieren
(z. B. über Dropbox → „Verfügbar offline" oder USB) und `index.html` in Chrome öffnen.
Kamera-Aufnahme und Dropbox-Anmeldung per Code funktionieren auch so; beim ersten Beleg
wird das Erkennungsmodul aus dem Internet geladen. Auf dem iPhone ist Option B nicht möglich.

Lokaler Test am PC: Doppelklick auf `index.html` genügt (Dropbox per Code verbinden), oder
```bash
python -m http.server 8777
```
→ <http://localhost:8777/MietAkte/index.html>
