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

## Von überall nutzen – für mehrere Personen (Android / iPhone / Desktop)

MietAkte ist eine Webseite. Sobald sie unter einer `https://`-Adresse liegt, kann sie jede
Person von überall im Browser öffnen und wie eine App auf dem Handy installieren.
Das eigene, private SYNIUM-Repo kann dafür nicht dienen (GitHub Pages ist im Free-Plan nur für
**öffentliche** Repos verfügbar). Deshalb bekommt MietAkte ein eigenes öffentliches Repo, das nur
die App-Dateien enthält (`index.html`, `Logo/`, `icons/`, `manifest.webmanifest`, `README.md`) –
keine Schlüssel, keine Belege, keine Nutzerdaten.

### Einrichtung (einmalig, Betreiber)
1. Auf GitHub ein **öffentliches** Repo `mietakte` anlegen (leer, ohne README).
2. Inhalt des Ordners `MietAkte/` als Wurzel in dieses Repo pushen (Branch `main`).
3. Repo → **Settings → Pages** → *Deploy from a branch* → `main` / `/ (root)` → Save.
   Nach 1–2 Minuten ist die App erreichbar unter `https://<user>.github.io/mietakte/`.
4. In der **Dropbox App Console** die App auf **Production** stellen, sobald mehr als
   50 Personen sie nutzen sollen (im Status „Development" sind 50 verbundene Dropbox-Konten erlaubt).
5. Optional: den Dropbox-App-Key fest in `index.html` eintragen
   (`const DEFAULT_DBX_APP_KEY = '…'`), damit Nutzer ihn nicht selbst eingeben müssen.
   Der App-Key ist kein Geheimnis (kein App-Secret nötig, Anmeldung per PKCE).

### Nutzung (jede Person)
1. Link `https://<user>.github.io/mietakte/` im Handy-Browser öffnen.
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
