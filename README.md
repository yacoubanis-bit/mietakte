# MietAkte — Belegverwaltung für vermietete Wohnungen (SYNIUM)

Web-App zum Erfassen von Renovierungs- und Instandhaltungsbelegen: Beleg fotografieren,
Betrag/MwSt/Datum/Händler per Vision-Modell (Anthropic API) erkennen lassen, prüfen,
einer Wohnung zuordnen und in Dropbox ablegen. Übersicht mit Filtern, Summen je Wohnung
und Excel-Export fürs Finanzamt.

> **Dateien:** `index.html` (die App) + Ordner `Logo/` (muss neben `index.html` liegen).
> Einzeldatei-App ohne Build. Die Excel-Bibliothek (SheetJS) wird beim Export von cdnjs geladen.

## Erste Einrichtung

Die App braucht zwei Zugänge, die unter **Einstellungen** eingetragen werden und nur im
Browser (localStorage) gespeichert bleiben:

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

### 2. Anthropic API-Key
Unter <https://console.anthropic.com> einen API-Key erzeugen und in der App eintragen.
Standardmodell ist Claude Opus 5; Sonnet 5 / Haiku 4.5 sind als günstigere Varianten wählbar.
**Verbindung testen** prüft den Schlüssel.

## Bedienung
- **Wohnungen**: Mietobjekte anlegen (Name, Adresse, Dropbox-Ordnername). Ordner werden
  unterhalb des Basisordners (Standard `/MietAkte`) angelegt.
- **Erfassen**: Foto aufnehmen oder Datei (JPG/PNG/PDF) wählen → Werte werden erkannt und
  blau markiert → prüfen/korrigieren → Wohnung und Kategorie wählen → **Speichern & hochladen**.
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
  ein Login für weitere Nutzer ergänzt werden kann. Für Mehrbenutzerbetrieb sollte der
  Anthropic-Aufruf dann über einen kleinen Server laufen, damit der API-Key nicht im Browser liegt.

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
