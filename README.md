# MietAkte — Belegverwaltung für vermietete Wohnungen (SYNIUM)

Web-App zum Erfassen von Renovierungs- und Instandhaltungsbelegen: Beleg fotografieren,
Betrag/MwSt/Datum/Händler auslesen lassen – wahlweise über **ChatGPT** (OpenAI-API, deutlich
treffsicherer) oder **lokal auf dem Gerät** (Tesseract-OCR, ohne Internet) –, prüfen, einer
Wohnung zuordnen und in Dropbox ablegen. Für eine Person auf dem eigenen Handy; Schlüssel
bleiben nur auf dem Gerät. Übersicht mit Filtern, Summen je Wohnung
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

### 2. Belegerkennung über ChatGPT (empfohlen)
1. Unter <https://platform.openai.com/api-keys> einen API-Key erzeugen (OpenAI-Konto mit
   Guthaben; abgerechnet wird pro Bild, wenige Cent oder weniger).
2. In MietAkte → Einstellungen → **OpenAI API-Key** eintragen, Modell prüfen (Standard
   `gpt-5.6`, änderbar) → **Verbindung testen**.
3. Erkennung „ChatGPT, bei Fehler lokal": Das Foto wird an OpenAI geschickt, das Modell liest
   Händler, Datum, Brutto, MwSt und Belegnummer aus, die App füllt die Felder. Ohne Internet
   oder bei einem Fehler springt automatisch die lokale Texterkennung ein.

Der Schlüssel liegt nur im Browser dieses Geräts (localStorage) und wird direkt an
`api.openai.com` gesendet. Die App ist für **eine Person** gedacht – wer den Schlüssel in eine
öffentlich verteilte App legt, gibt ihn allen Nutzern preis. Hinweis: Bei einem ungültigen
Schlüssel meldet der Browser nur „Keine Verbindung", weil OpenAI Fehlerantworten ohne
CORS-Freigabe liefert – dann Schlüssel prüfen.

### 3. Lokale Texterkennung (Rückfall, offline)
Keine Einrichtung nötig. Unter Einstellungen kann die Sprache (Deutsch bzw. Deutsch + Englisch)
gewählt und das Erkennungsmodul vorab geladen werden (einmalig ca. 15 MB). Mit „Nur lokal"
verlässt kein Beleg das Gerät, die Trefferquote ist aber geringer als mit ChatGPT.

## Bedienung
Die Oberfläche passt sich der Bildschirmgröße an: auf dem Handy mit unterer Tab-Leiste, am
Desktop (ab 1100 px Breite) mit linker Navigationsleiste und mehrspaltigen Ansichten; ab 1700 px
(23-Zoll-Monitore und größer) mit größerer Schrift, drei Spalten in den Einstellungen und
bis zu 2100 px Inhaltsbreite.

- **Wohnungen**: Mietobjekte anlegen (Name, Adresse, Dropbox-Ordnername). Ordner werden
  unterhalb des Basisordners (Standard `/MietAkte`) angelegt. Je Kaufjahr entsteht darin
  automatisch ein Unterordner `Steuer JJJJ` (Jahr aus dem Belegdatum), z. B.
  `/MietAkte/Wohnung_Hauptstrasse_12/Steuer 2025/2025-03-04_Bauhaus_43,89EUR.jpg`.
  Die Excel-Jahresaufstellungen liegen davon getrennt unter `/MietAkte/Steuer/`.
- **Erfassen**: Foto aufnehmen oder Datei (JPG/PNG/PDF) wählen → Beleg wird über ChatGPT bzw.
  lokal ausgewertet, die Werte werden blau markiert → prüfen/korrigieren → Wohnung und
  Kategorie wählen → **Speichern & hochladen**. Der Hinweis unter dem Foto nennt, welche
  Erkennung gearbeitet hat; bei lokaler Erkennung zeigt „Erkannten Text anzeigen" den Rohtext. Tipp: Beleg gerade, hell und
  scharf fotografieren; bereits erfasste Händlernamen werden beim nächsten Mal wiedererkannt.
  Das Bild wird auf max. 1800 px verkleinert und als
  `JJJJ-MM-TT_Händler_Betrag,xxEUR.jpg` im Wohnungsordner abgelegt.
- **Übersicht**: Dashboard nach Steuerjahr – je Jahr eine Karte mit Jahressumme (Brutto, MwSt,
  Netto), Summen je Kategorie und einer Tabelle aller Belege des Jahres (Datum, Händler,
  Brutto, MwSt) samt Summenzeile; filterbar nach Wohnung und Kategorie, Tipp auf
  eine Zeile öffnet den Beleg, „Excel JJJJ" exportiert das Jahr.
- **Belege**: Liste mit Filter (Wohnung, Kategorie, Zeitraum/Jahr, Suche), Summen je Wohnung,
  Bearbeiten/Löschen per Tipp auf einen Beleg.
- **Excel Steuerjahr**: Erzeugt je Steuerjahr **eine** Datei `Steuer_JJJJ.xlsx` mit **allen** Belegen
  des Jahres über alle Wohnungen (ungefiltert): Blatt „Steuer JJJJ" mit Nr., Wohnung, Datum,
  Händler (Link zum Beleg), Kategorie, Brutto, MwSt, Bemerkung und Summenzeile; dazu die Blätter
  „Je Wohnung" und „Nach Kategorie". Die Datei wird lokal heruntergeladen **und** in Dropbox unter
  `<Basisordner>/Steuer/Steuer_JJJJ.xlsx` abgelegt – beim erneuten Erstellen wird die Datei des
  Jahres überschrieben. Aufruf über „Excel JJJJ" in der Übersicht oder „Excel Steuerjahr" in der
  Belegliste (Jahr aus dem Jahresfilter, sonst das jüngste Jahr mit Belegen).
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

## Von überall nutzen – für mehrere Personen (Android / iPhone / Desktop)

MietAkte ist eine Webseite. Sobald sie unter einer `https://`-Adresse liegt, kann sie jede
Person von überall im Browser öffnen und wie eine App auf dem Handy installieren.
Das eigene, private SYNIUM-Repo kann dafür nicht dienen (GitHub Pages ist im Free-Plan nur für
**öffentliche** Repos verfügbar). Deshalb bekommt MietAkte ein eigenes öffentliches Repo, das nur
die App-Dateien enthält (`index.html`, `Logo/`, `icons/`, `manifest.webmanifest`, `README.md`) –
keine Schlüssel, keine Belege, keine Nutzerdaten.

### Veröffentlichung (eingerichtet am 15.09.2026)
Die App ist unter **<https://yacoubanis-bit.github.io/mietakte/>** erreichbar (GitHub Pages aus dem
öffentlichen Repo `yacoubanis-bit/mietakte`, das nur die App-Dateien enthält). Aktualisieren nach
Änderungen im SYNIUM-Repo:
```bash
git subtree split --prefix=MietAkte -b mietakte-pages && git push -f https://github.com/yacoubanis-bit/mietakte.git mietakte-pages:main
```
4. In der **Dropbox App Console** die App auf **Production** stellen, sobald mehr als
   50 Personen sie nutzen sollen (im Status „Development" sind 50 verbundene Dropbox-Konten erlaubt).
5. Optional: den Dropbox-App-Key fest in `index.html` eintragen
   (`const DEFAULT_DBX_APP_KEY = '…'`), damit Nutzer ihn nicht selbst eingeben müssen.
   Der App-Key ist kein Geheimnis (kein App-Secret nötig, Anmeldung per PKCE).

### Nutzung (jede Person)
1. Link <https://yacoubanis-bit.github.io/mietakte/> im Handy-Browser öffnen.
2. Installieren: **Android/Chrome** → Menü ⋮ → „App installieren" bzw. „Zum Startbildschirm";
   **iPhone/Safari** → Teilen-Symbol → „Zum Home-Bildschirm". Danach startet MietAkte
   als eigene App mit SYNIUM-Symbol.
3. Einstellungen → **Verbinden (Code eingeben)** → bei Dropbox anmelden → Code einfügen.
   Jede Person meldet sich mit **ihrem eigenen Dropbox-Konto** an; Belege und die Belegtabelle
   landen in ihrer Dropbox. Nichts wird zwischen Personen geteilt.
4. Wohnungen anlegen, Belege fotografieren – fertig. Beim ersten Beleg lädt das Erkennungsmodul
   einmalig ca. 15 MB (danach offline nutzbar, nur der Upload braucht Internet).

### Alle Belege in einer gemeinsamen Dropbox?
Sollen mehrere Personen in **dieselbe** Dropbox schreiben (z. B. Verwalter und Helfer), gibt es
zwei Wege:
- **Gemeinsames Konto:** alle melden sich mit demselben Dropbox-Konto an (einfach, aber kein
  Nachvollziehen, wer was hochgeladen hat).
- **Freigegebener Ordner:** Der Eigentümer gibt den Ordner `/MietAkte` in Dropbox für die anderen
  frei; jede Person verbindet ihr eigenes Konto und trägt als Basisordner den Pfad des
  freigegebenen Ordners ein. Dafür muss die Dropbox-App den Zugriffstyp **Full Dropbox** haben
  (nicht *App folder*). Die Belegtabelle `mietakte-daten.json` wird dann von allen gemeinsam
  genutzt und beim Start zusammengeführt.

Lokaler Test am PC: Doppelklick auf `index.html` genügt (Dropbox per Code verbinden), oder
```bash
python -m http.server 8777
```
→ <http://localhost:8777/MietAkte/index.html>
