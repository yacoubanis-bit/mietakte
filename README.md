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

### 1. Dropbox App-Key
1. <https://www.dropbox.com/developers/apps> → **Create app** → *Scoped access* →
   *App folder* (Belege landen dann in `Apps/<Appname>/…`) oder *Full Dropbox*.
2. Reiter **Permissions** aktivieren: `account_info.read`, `files.metadata.read`,
   `files.metadata.write`, `files.content.read`, `files.content.write`,
   `sharing.read`, `sharing.write` → **Submit**.
3. Reiter **Settings** → **Redirect URIs**: genau die Adresse eintragen, die die App unter
   Einstellungen anzeigt (z. B. `https://<host>/MietAkte/index.html` oder
   `http://localhost:8777/MietAkte/index.html`). Dropbox akzeptiert nur `https://`
   oder `http://localhost` – **kein `file://`**.
4. **App key** kopieren, in der App eintragen → **Mit Dropbox verbinden**.
   Die Anmeldung läuft per OAuth 2 mit PKCE (kein App-Secret nötig) und bleibt über
   ein Refresh-Token dauerhaft bestehen.

### 2. Texterkennung
Keine Einrichtung nötig. Unter Einstellungen kann die Sprache (Deutsch bzw. Deutsch + Englisch)
gewählt und das Erkennungsmodul vorab geladen werden, damit der erste Beleg schneller geht.

## Bedienung
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

## Hosting fürs Handy
Kamera-Zugriff und Dropbox-Anmeldung setzen `https://` voraus. Beispiel GitHub Pages:
Repo mit `MietAkte/index.html` + `MietAkte/Logo/` veröffentlichen, dann
`https://<user>.github.io/<repo>/MietAkte/index.html` als Redirect-URI eintragen und auf
dem Handy als Startbildschirm-Verknüpfung ablegen.

Lokaler Test am PC:
```bash
python -m http.server 8777
```
→ <http://localhost:8777/MietAkte/index.html>
