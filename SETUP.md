# Autowartung – Vollständige Einrichtungsdokumentation

Stand: 22. Juni 2026 · App-Cache-Version: `v13`

---

## Inhaltsverzeichnis

1. [Projektübersicht](#1-projektübersicht)
2. [Funktionsumfang](#2-funktionsumfang)
3. [Projekt-Dateien](#3-projekt-dateien)
4. [Firebase-Projekt einrichten](#4-firebase-projekt-einrichten)
5. [Firestore-Datenbank](#5-firestore-datenbank)
6. [Datenmodell (Collections & Felder)](#6-datenmodell-collections--felder)
7. [Google-Authentifizierung](#7-google-authentifizierung)
8. [Firestore Security Rules](#8-firestore-security-rules)
9. [Firebase-Konfiguration in der App](#9-firebase-konfiguration-in-der-app)
10. [GitHub Repository & GitHub Pages](#10-github-repository--github-pages)
11. [Firebase Authorized Domain](#11-firebase-authorized-domain)
12. [App aktualisieren (Workflow)](#12-app-aktualisieren-workflow)
13. [PWA-Installation auf dem Handy](#13-pwa-installation-auf-dem-handy)
14. [Datensicherung](#14-datensicherung)
15. [Service Worker & Cache-Versionen](#15-service-worker--cache-versionen)
16. [Zugangsdaten & URLs](#16-zugangsdaten--urls)
17. [Troubleshooting](#17-troubleshooting)

---

## 1. Projektübersicht

**Autowartung** ist eine Progressive Web App (PWA) zur Verwaltung mehrerer Fahrzeuge inkl. Wartung, Reparaturen, Kundendienst, Tankungen, Ladungen (Elektro) und Zubehör.

| Technologie | Zweck |
|---|---|
| HTML / CSS / Vanilla JS | Frontend (eine einzelne `index.html`) |
| Firebase Authentication | Google-Login mit Whitelist |
| Firebase Firestore | Datenbank mit Echtzeit-Sync zwischen Geräten |
| Firestore Persistent Cache | Offline-Fähigkeit der Daten |
| Service Worker (PWA) | App-Shell-Cache, Installation auf Homescreen |
| GitHub Pages | Hosting (kostenlos, HTTPS) |

**Live-URL:** `https://arm-git-100.github.io/autowartung`

Es gibt keinen Build-Prozess und kein npm — du editierst direkt `index.html`, pushst zu GitHub, und nach 1–2 Minuten ist der neue Stand online.

---

## 2. Funktionsumfang

### Fahrzeug-Übersicht (Dashboard)
- Liste aller Fahrzeuge mit Typ, Kennzeichen, Nutzer, Antriebsart, aktueller Kilometerstand
- Farbiger Status-Streifen am linken Rand (grün/gelb/rot/grau) je nach Wartungslage
- Status-Badges: Anzahl OK / bald fällig / überfällig
- Bei Verbrennern: Ø-Verbrauch in l/100 km
- Bei Elektro: Ø-Verbrauch in kWh/100 km
- Buttons: 📥 Export · 📤 Import · ➕ Fahrzeug

### Fahrzeug-Detailansicht
**Action-Leiste:**
- 📍 Km aktualisieren
- ⛽ Tanken (nur Benziner/Diesel/Hybrid)
- 🔌 Laden (nur Elektro/Hybrid)
- ✏️ Bearbeiten
- 📋 Stammdaten
- 📊 Statistik
- 📄 PDF
- 🗑️ Löschen

**Vier Tabs:**

1. **🔧 Wartungen**
   - Liste aller erfassten Wartungstypen mit Status (OK / bald fällig / überfällig)
   - Pro Wartungstyp: letzte Wartung, nächste Fälligkeit (km + Datum), Fortschrittsbalken (km + Zeit)
   - Historie aller Einträge mit Kommentar
   - Verwaltung der **Wartungstypen** (Name, Icon, Intervalle km/Monate)
   - 22 vordefinierte Typen werden beim ersten Start automatisch angelegt
   - Eigene Typen können hinzugefügt und nur eigene wieder gelöscht werden

2. **🔨 Reparaturen**
   - Beschreibung (mehrzeilig), Datum, Km, Werkstatt, Kosten

3. **🛠️ Kundendienste**
   - Datum, Km, Werkstatt, Beschreibung (mehrzeilig), Kosten

4. **🛍️ Zubehör**
   - Datum, Beschreibung (mehrzeilig), Preis (optional)
   - Summen-Banner oberhalb der Liste

### Tanken (Modal)
- 🏠 **Voll-Betankung**: Datum, Km, Liter, **Gesamtpreis** → Literpreis berechnet, aktualisiert Fahrzeug-Km automatisch wenn höher
- 🔹 **Teil-Betankung**: Datum, Liter, Gesamtpreis (kein Km, keine Verbrauchsberechnung)
- Ø-Verbrauch (l/100 km) ab 2 Voll-Betankungen
- Ø-Literpreis, Gesamtkosten

### Laden (Modal)
- **Heim-Preis** €/kWh pro Fahrzeug (Default 0,300 €/kWh, dreistellig, im Modal anpassbar; zuletzt verwendeter Wert wird in `localStorage` gemerkt)
- 🏠 **Heim-Monatsladung**: Monat, kWh, **Km am Monatsende** (Pflicht) → Kosten = kWh × Heim-Preis
- 🔌 **Öffentliche Ladung**: Datum, kWh, Gesamtpreis, Km (optional) → €/kWh berechnet
- Ø-Verbrauch (kWh/100 km) ab 2 Einträgen mit Km

### Stammdaten (Modal)
Modell, Fahrgestellnummer, Erstzulassung, Farbcode, Schlüssel 2.1/2.2, Kaufdatum, Kaufpreis, **Km bei Kauf**, Gekauft von (Händler/Privat), Zubehör bei Kauf (Freitext), Öl Norm/Menge, Reifengröße Sommer/Winter, **Infos / Notizen** (Freitext), Batterie Nettokapazität, **SoH-Historie** (mehrere Messungen mit Datum)

### Statistik (Modal)
Fahrleistungs-Auswertung, vollständig **automatisch** aus allen vorhandenen km-Einträgen berechnet (Wartungen, Reparaturen, Kundendienste, Tankungen, Ladungen, aktueller Stand, Km bei Kauf). Pro Datum wird der höchste km-Stand genommen, Zwischenwerte werden linear interpoliert.

**Einstellungen (pro Fahrzeug, gespeichert im `vehicles`-Dokument):**
- **Berechnung ab** – Startdatum der Auswertung (z. B. `15.06.2026`)
- **Geplante Fahrleistung pro Jahr** – Liste „Jahr → km" (= Fahrleistungsangabe der Versicherung), pro Jahr einstellbar

**Auswertung (live, aktualisiert sich sofort bei Änderung):**
- **📊 Gesamtkilometer pro Kalenderjahr** – Balkendiagramm; unvollständige Jahre (laufend bzw. ab Startdatum) mit `*` markiert
- **📅 Versicherungsjahr (ab Startdatum)** – rollende 12-Monats-Periode (z. B. 15.06.2026–14.06.2027) mit: gefahren bisher, Ø pro Monat, Hochrechnung aufs Jahr, Plan-Vergleich (Fortschrittsbalken mit Soll-Marke, grün ≤ Plan / rot > Plan, Klartext „voraussichtlich X km über/unter Plan") sowie einer Liste abgeschlossener Perioden mit Ist vs. Plan

### PDF-Export
Komplette Wartungshistorie inkl. Stammdaten, Wartungen, Reparaturen, Kundendienste, Zubehör, Tankungen, Ladungen — eine HTML-Seite, die Druck-Dialog automatisch öffnet (→ als PDF speicherbar).

### Datensicherung
- **Export**: JSON-Datei mit allen 8 Collections (Stand 22.06.2026 vollständig; Statistik-Einstellungen sind Felder in `vehicles` und damit enthalten)
- **Import**: Komplettes Überschreiben der Cloud-Daten
- **Eingabe-Komfort**: Km-Felder akzeptieren Punkt als Tausender-Trenner (`100.000` = `100000`)

---

## 3. Projekt-Dateien

```
C:\ai\projekte\Autowartung\
├── index.html                       ← Die ganze App (HTML+CSS+JS in einer Datei)
├── sw.js                            ← Service Worker für PWA / Offline
├── manifest.json                    ← PWA-Manifest
├── icon-192.svg                     ← App-Icon (192×192)
├── icon-512.svg                     ← App-Icon (512×512)
├── README.md                        ← Originale Projektidee
├── SETUP.md                         ← Diese Dokumentation
├── screenshot1.png                  ← Screenshot
└── index_backup_*.html              ← Versions-Backups (manuell erstellt)
```

Eine **Strategie für lokale Backups** ist hilfreich, weil Cache-Probleme oder fehlerhafte Edits am `index.html` selten sind, aber vorkommen können. Üblich war zuletzt:

- Vor jedem größeren Feature: `index_backup_YYYYMMDD_HHMM.html` anlegen
- Stabile Stände zusätzlich als `index_backup_ready_vNNN.html` markieren

---

## 4. Firebase-Projekt einrichten

1. [console.firebase.google.com](https://console.firebase.google.com) öffnen
2. **„Projekt hinzufügen"**
3. Name: `autowartung`
4. Google Analytics: **deaktivieren** (nicht nötig)
5. **„Projekt erstellen"**

---

## 5. Firestore-Datenbank

1. Linkes Menü: **„Firestore Database"** → **„Datenbank erstellen"**
2. Modus: **„Testmodus"** (Regeln werden gleich gehärtet)
3. Standort: **`europe-west3`** (Frankfurt)
4. **„Weiter"** → **„Fertig"**

Es müssen **keine Collections von Hand angelegt** werden — die App erzeugt sie beim ersten Schreiben.

---

## 6. Datenmodell (Collections & Felder)

Die App nutzt **8 Top-Level-Collections**. Alle Dokument-IDs werden von Firestore automatisch vergeben (außer `maintenanceTypes`, dort verwendet die App sprechende IDs für die 22 Default-Typen wie `oil`, `brakes_front` und `custom_<timestamp>` für eigene).

### `vehicles`
```
type, plate, user, drivetrain ('Benziner'|'Diesel'|'Elektro'|'Hybrid'|null),
mileage (number|null), mileageDate (YYYY-MM-DD),
kwhPrice (number, nur Elektro),
// Stammdaten:
model, vin, firstReg, colorCode, key21, key22,
purchaseDate, purchasePrice, purchaseMileage, purchaseSource ('Händler'|'Privat'),
accessories (string, "Zubehör bei Kauf"),
oilSpec, tireSummer, tireWinter, batteryCap,
notes (string|null, "Infos / Notizen"),
batterySohHistory: [{value, date}, ...],
// Statistik:
statsStartDate (YYYY-MM-DD|null, Startdatum der Auswertung),
plannedKm: [{year (number), km (number)}, ...] | null
```
> **Hinweis (km = null):** Ein leerer/`null` Kilometerstand wird in der App als **0 km** behandelt (Auslieferungs-/Neuwagenfall). Die Statistik interpoliert nur über Datenpunkte mit `mileage > 0`.

### `maintenanceTypes`
```
name, icon (Emoji), intervalKm (number|null), intervalMonths (number|null)
```
Beim ersten Start werden 22 Default-Typen ohne Intervalle angelegt (Ölwechsel, Bremsen, Zahnriemen, TÜV, …). Eigene haben IDs mit Präfix `custom_`.

### `maintenanceRecords`
```
vehicleId, typeId, date (YYYY-MM-DD), mileage (number|null),
intervalKm (number|null, optional pro Eintrag),
intervalMonths (number|null, optional pro Eintrag),
comment (string|null)
```
Ein Eintrag kann eigene Intervalle haben, die das Default des Typs überschreiben (z. B. wenn Ölwechsel mal früher).

### `repairs`
```
vehicleId, description (string, mehrzeilig), date, mileage, workshop, cost
```

### `services`
```
vehicleId, date, mileage, workshop, description (mehrzeilig), cost
```

### `fuelings`
```
vehicleId, date, isFull (boolean), mileage (number|null bei Teil),
liters (number), totalCost (number)
```
**Literpreis** wird live berechnet (totalCost ÷ liters). **Verbrauch (l/100 km)** wird zwischen aufeinanderfolgenden Voll-Betankungen berechnet — Teil-Betankungen dazwischen werden zur Liter-Summe addiert.

### `charges`
```
vehicleId, kind ('home'|'public'),
// kind='home':
month (YYYY-MM), kwh, mileage (Pflicht!),
// kind='public':
date (YYYY-MM-DD), kwh, totalCost, mileage (optional)
```
**Kosten** bei `home` werden live mit `vehicles.kwhPrice` berechnet (Variante: Preis pro Fahrzeug, retroaktiv). **Verbrauch (kWh/100 km)** zwischen Einträgen mit Km-Stand.

### `accessories`
```
vehicleId, date, description (mehrzeilig), cost (number|null)
```

---

## 7. Google-Authentifizierung

1. Linkes Menü: **„Authentication"** → **„Loslegen"**
2. Tab **„Sign-in-Methode"**
3. Anbieter **„Google"** anklicken
4. Toggle auf **aktiviert**
5. Projekt-Support-E-Mail wählen
6. **„Speichern"**

---

## 8. Firestore Security Rules

Nur explizit aufgeführte Google-Konten dürfen lesen/schreiben:

1. **„Firestore Database"** → Tab **„Regeln"**
2. Code ersetzen:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /{document=**} {
      allow read, write: if request.auth != null
        && request.auth.token.email in [
          'armin@beispiel.de',
          'heike@beispiel.de'
        ];
    }
  }
}
```

3. Echte E-Mail-Adressen einsetzen
4. **„Veröffentlichen"**

> Nicht-autorisierte Konten landen beim Login auf dem Fehler-Screen („Kein Zugriff – Dein Konto ist nicht für diese App autorisiert").

---

## 9. Firebase-Konfiguration in der App

In `index.html` im `<script type="module">`-Block ganz oben:

```js
const firebaseConfig = {
  apiKey:            "AIzaSyD3YSbcAw3qakXlVQcPfnXI_g48FNWH3ss",
  authDomain:        "autowartung-app.firebaseapp.com",
  projectId:         "autowartung-app",
  storageBucket:     "autowartung-app.firebasestorage.app",
  messagingSenderId: "275183006005",
  appId:             "1:275183006005:web:939d19a65d6e6efb89dc7c"
};
```

Diese Werte findest du in der Firebase Console:

⚙️ **Projekteinstellungen → Deine Apps → `</>` Web-App registrieren** (falls noch nicht geschehen).

---

## 10. GitHub Repository & GitHub Pages

### Repository erstellen
1. [github.com](https://github.com) → **„+" → „New repository"**
2. Name: `autowartung`, Sichtbarkeit: **Public** (für kostenloses GitHub Pages)
3. Kein Häkchen bei „Add a README"
4. **„Create repository"**

### Dateien hochladen (einmalig)
```bash
cd C:\ai\projekte\Autowartung
git init
git add index.html sw.js manifest.json icon-192.svg icon-512.svg README.md SETUP.md
git commit -m "Autowartung App – initial commit"
git remote add origin https://github.com/arm-git-100/autowartung.git
git branch -M main
git push -u origin main
```

### GitHub Pages aktivieren
1. Repository → **„Settings"** → **„Pages"**
2. Source: Branch **`main`**, Ordner **`/ (root)`**
3. **„Save"**
4. Nach 1–2 Minuten erreichbar unter: `https://<USER>.github.io/autowartung`

---

## 11. Firebase Authorized Domain

Damit der Google-Login auf der GitHub-Pages-Domain funktioniert:

1. Firebase Console → **„Authentication"** → Tab **„Settings"**
2. Abschnitt **„Autorisierte Domains"**
3. **„Domain hinzufügen"**
4. Wert: `arm-git-100.github.io` (bzw. die eigene GH-Pages-Domain)
5. **„Hinzufügen"**

`localhost` ist standardmäßig schon authorisiert (für lokale Tests).

---

## 12. App aktualisieren (Workflow)

Nach Änderungen an `index.html` oder anderen Dateien:

```bash
cd C:\ai\projekte\Autowartung
git add index.html sw.js
git commit -m "Beschreibung der Änderung"
git push
```

GitHub Pages liefert nach 1–2 Minuten den neuen Stand aus.

> **Wichtig bei UI/JS-Änderungen:** **Service-Worker-Cache-Version in `sw.js` hochzählen** (`const CACHE = 'autowartung-vNN'`). Sonst sehen installierte PWAs noch die alte Version. Details: → Abschnitt 15.

---

## 13. PWA-Installation auf dem Handy

### Android (Chrome)
1. App im Browser öffnen (`https://<USER>.github.io/autowartung`)
2. ⋮ → **„Zum Startbildschirm hinzufügen"**

### iPhone (Safari)
1. App in Safari öffnen
2. Teilen-Button → **„Zum Home-Bildschirm"**

### Windows / Mac (Chrome/Edge)
1. App im Browser öffnen
2. Installations-Icon in der Adressleiste klicken

Die App läuft danach im Vollbild ohne Browser-Chrome.

---

## 14. Datensicherung

### Cloud-Backup (in der App)
- **Export**: Dashboard → **📥 Export** → Datei `autowartung_backup_YYYY-MM-DD.json`
- **Import**: Dashboard → **📤 Import** → JSON-Datei wählen → bestätigen
- Import **ersetzt vollständig** alle Cloud-Daten (alle 8 Collections werden geleert und neu geschrieben)
- Alte Backups (vor Tanken/Laden/Zubehör) sind kompatibel — fehlende Arrays werden als leer behandelt

**Inhalt des Backups:** alle Felder aller Dokumente aus den 8 Collections. **Nicht** im Backup: `lastKwhPrice` aus `localStorage` (das ist nur ein lokaler UI-Default, kein Datum — pro Fahrzeug ist der Preis in `vehicles.kwhPrice` gespeichert und damit im Backup).

### Lokale Code-Backups
Snapshots der `index.html` werden manuell angelegt:

```powershell
$stamp = Get-Date -Format "yyyyMMdd_HHmm"
Copy-Item index.html "index_backup_$stamp.html"
```

Für stabile Stände zusätzlich z. B. `index_backup_ready_v001.html`.

---

## 15. Service Worker & Cache-Versionen

`sw.js` cached die App-Shell (`index.html`, Manifest, Icons), damit die App offline lädt.

```js
const CACHE = 'autowartung-vNN';
const SHELL = ['./', './index.html', './manifest.json', './icon-192.svg', './icon-512.svg'];
```

**Regel:** Jede Code-Änderung an `index.html` → `vNN` in `sw.js` um eins erhöhen (aktuell `v13`). Sonst zeigt eine installierte PWA noch den alten Stand, weil der Service Worker die alte App ausliefert.

Beim ersten Besuch nach Update einmal `Strg + F5` (Desktop) oder Tab neuladen (Mobile) — danach übernimmt die neue Version.

---

## 16. Zugangsdaten & URLs

| Was | Wert |
|---|---|
| App (live) | `https://arm-git-100.github.io/autowartung` |
| GitHub Repository | `https://github.com/arm-git-100/autowartung` |
| Lokaler Pfad | `C:\ai\projekte\Autowartung` |
| Firebase Console | `https://console.firebase.google.com` |
| Firebase Projekt-ID | `autowartung-app` |
| Firestore Region | `europe-west3` (Frankfurt) |
| Default Heim-Preis €/kWh | `0,300` |
| Aktuelle Cache-Version | `autowartung-v13` |

---

## 17. Troubleshooting

| Symptom | Ursache | Lösung |
|---|---|---|
| Nach Update sehe ich alte Version | Service Worker liefert alten Cache | In `sw.js` Version hochzählen, `Strg+F5`, evtl. Browser-Cache leeren |
| „Kein Zugriff – Dein Konto ist nicht autorisiert" | E-Mail nicht in den Firestore-Regeln | E-Mail in den Rules ergänzen + veröffentlichen |
| Google-Login funktioniert auf GitHub Pages nicht | Domain fehlt in Authorized Domains | In Firebase → Authentication → Settings → Domain hinzufügen |
| App lädt unendlich / „Daten werden geladen…" | Firestore unreachable oder Rules zu eng | Browser-Konsole prüfen, Firestore-Regeln testen |
| Datum springt ins Jahr 0026 statt 2026 | Browser interpretiert zweistelliges Jahr | App korrigiert automatisch (Funktion `fixYear`) — falls nicht: manuell 4-stellig eingeben |
| Voll-Betankung trotz höherem Km ändert Fahrzeug-Km nicht | Eingegebener Km ≤ aktueller Fahrzeug-Km | Stimmt so — Fahrzeug-Km wird nur erhöht, nicht gesenkt |
| Verbrauch wird nicht angezeigt | Weniger als 2 Voll-Betankungen / 2 Lade-Einträge mit Km | Mehr Einträge erfassen |
