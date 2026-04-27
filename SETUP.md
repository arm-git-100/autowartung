# Autowartung – Einrichtungsdokumentation

## Inhaltsverzeichnis
1. [Projektübersicht](#1-projektübersicht)
2. [Firebase Projekt einrichten](#2-firebase-projekt-einrichten)
3. [Firestore Datenbank](#3-firestore-datenbank)
4. [Google Authentifizierung](#4-google-authentifizierung)
5. [Firestore Security Rules](#5-firestore-security-rules)
6. [Firebase Konfiguration in der App](#6-firebase-konfiguration-in-der-app)
7. [GitHub Repository & Deployment](#7-github-repository--deployment)
8. [Firebase Authorized Domain](#8-firebase-authorized-domain)
9. [App aktualisieren](#9-app-aktualisieren)
10. [PWA Installation](#10-pwa-installation)
11. [Datensicherung](#11-datensicherung)
12. [Zugangsdaten & URLs](#12-zugangsdaten--urls)

---

## 1. Projektübersicht

**Autowartung** ist eine Progressive Web App zur Verwaltung von Fahrzeugwartungen.

| Technologie | Verwendung |
|-------------|-----------|
| HTML/CSS/JS | Frontend (eine einzelne `index.html`) |
| Firebase Auth | Google Login, Zugangskontrolle |
| Firebase Firestore | Datenbank (Echtzeit-Sync) |
| GitHub Pages | Hosting (kostenlos, HTTPS) |
| PWA / Service Worker | Offline-Fähigkeit, Homescreen-Installation |

**Live-URL:** `https://arm-git-100.github.io/autowartung`

---

## 2. Firebase Projekt einrichten

1. Öffne [console.firebase.google.com](https://console.firebase.google.com)
2. Klick **"Projekt hinzufügen"**
3. Name: `autowartung`
4. Google Analytics: deaktivieren
5. Klick **"Projekt erstellen"**

---

## 3. Firestore Datenbank

1. Im linken Menü: **"Firestore Database"**
2. Klick **"Datenbank erstellen"**
3. Modus: **"Testmodus"**
4. Standort: **`europe-west3`** (Frankfurt)
5. Klick **"Weiter"** → **"Fertig"**

**Datenstruktur (Collections):**

| Collection | Inhalt |
|------------|--------|
| `vehicles` | Fahrzeuge (Typ, Kennzeichen, Nutzer, Km) |
| `maintenanceTypes` | Wartungstypen (Name, Icon, Intervalle) |
| `maintenanceRecords` | Wartungseinträge (Datum, Km, Intervall, Kommentar) |
| `repairs` | Reparatureinträge (Beschreibung, Datum, Km) |

---

## 4. Google Authentifizierung

1. Im linken Menü: **"Authentication"** → **"Loslegen"**
2. Tab **"Sign-in-Methode"**
3. Klick auf **"Google"**
4. Toggle auf **aktiviert** schalten
5. Projekt-Support-E-Mail auswählen
6. Klick **"Speichern"**

---

## 5. Firestore Security Rules

Nur autorisierte E-Mail-Adressen erhalten Datenzugriff.

1. **"Firestore Database"** → Tab **"Regeln"**
2. Code ersetzen mit:

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

3. E-Mail-Adressen durch die echten Google-Konten ersetzen
4. Klick **"Veröffentlichen"**

> Nur Nutzer deren E-Mail in der Liste steht können sich anmelden und Daten sehen/bearbeiten.

---

## 6. Firebase Konfiguration in der App

Die Konfiguration steht in `index.html` im `<script type="module">` Block:

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
**Zahnrad ⚙️ → Projekteinstellungen → Deine Apps → `</>`**

---

## 7. GitHub Repository & Deployment

### Repository erstellen
1. [github.com](https://github.com) → **"+"** → **"New repository"**
2. Name: `autowartung`, Sichtbarkeit: **Public**
3. Kein Häkchen bei "Add a README"
4. Klick **"Create repository"**

### Dateien hochladen (einmalig)
```bash
cd C:\ai\projekte\Autowartung
git init
git add .
git commit -m "Autowartung App - erste Version"
git remote add origin https://github.com/arm-git-100/autowartung.git
git branch -M main
git push -u origin main
```

### GitHub Pages aktivieren
1. Repository → **"Settings"** → **"Pages"**
2. Source: Branch **`main`**, Ordner **`/ (root)`**
3. Klick **"Save"**
4. Nach 1-2 Minuten erreichbar unter: `https://arm-git-100.github.io/autowartung`

---

## 8. Firebase Authorized Domain

Damit der Google-Login auf GitHub Pages funktioniert:

1. Firebase Console → **"Authentication"** → Tab **"Settings"**
2. Scrolle zu **"Autorisierte Domains"**
3. Klick **"Domain hinzufügen"**
4. Eintragen: `arm-git-100.github.io`
5. Klick **"Hinzufügen"**

---

## 9. App aktualisieren

Nach Änderungen an `index.html` (oder anderen Dateien):

```bash
cd C:\ai\projekte\Autowartung
git add .
git commit -m "Beschreibung der Änderung"
git push
```

GitHub Pages aktualisiert sich automatisch nach 1-2 Minuten.

---

## 10. PWA Installation

### Android (Chrome)
1. App im Browser öffnen
2. Menü (⋮) → **"Zum Startbildschirm hinzufügen"**

### iPhone (Safari)
1. App in Safari öffnen
2. Teilen-Button → **"Zum Home-Bildschirm"**

### Windows/Mac (Chrome/Edge)
1. App im Browser öffnen
2. Adressleiste → Installations-Icon klicken

---

## 11. Datensicherung

### Manuelles Backup (Export)
- In der App: Dashboard → **"📥 Export"**
- Lädt eine JSON-Datei herunter: `autowartung_backup_DATUM.json`

### Wiederherstellung (Import)
- In der App: Dashboard → **"📤 Import"**
- JSON-Backup-Datei auswählen
- Bestätigen → alle Daten werden wiederhergestellt

### Lokale Backup-Dateien
Backups der `index.html` liegen im Projektordner:
```
C:\ai\projekte\Autowartung\
├── index_backup_20260427.html
└── index_backup_20260427_1025.html
```

---

## 12. Zugangsdaten & URLs

| Was | URL / Wert |
|-----|-----------|
| App (live) | `https://arm-git-100.github.io/autowartung` |
| GitHub Repo | `https://github.com/arm-git-100/autowartung` |
| Firebase Console | `https://console.firebase.google.com` |
| Firebase Projekt-ID | `autowartung-app` |
| Firestore Region | `europe-west3` (Frankfurt) |
