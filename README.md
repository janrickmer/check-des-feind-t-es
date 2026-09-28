# Check des Feind(t)es

Auswertungsseite für die Lehrkraft: <https://check.janrickmer.de>

Eine einzige HTML-Datei (`index.html`) ohne externe Bibliotheken. Sie liest die PDF-Dateien
aus den Escape-Rooms (kombinierte Datei „Escape Room Daten von …“), aus dem alleinigen
Feiditor („Text von …“) und die Escape-Room-Überblicke („Escape-Room-Überblick_von_…“)
direkt im Browser aus und zeigt eine Tabelle:

| Spalte | Inhalt |
|---|---|
| Nachname / Vorname | aus den verschlüsselten Feiditor-Daten (sonst aus dem Überblick) |
| Code passt? | **ja** (grün) / **nein** (rot, fett): Bestätigungscode ↔ Geburtsdatum des Escape-Room-Überblicks, Konstante des jeweiligen Escape-Rooms, Namensabgleich Feiditor-Daten ↔ Überblick. „–“ = Datei enthält keinen Code (reiner Feiditor). |
| Rote Buchstaben | Rot-Anteil wie in der Feiditor-Lehrkraft-Auswertung (eingefügt oder ≤ 0,015 s getippt); über 50 % rot hervorgehoben |
| Feiditor-Bewertung | je Aufgabe „Feiditor-Nr. N: sehr gut … ungenügend“, bewertet von Claude (Modell `claude-opus-5-5`) |

Ein Klick auf eine Zeile zeigt Details (Kurs, Geburtsdatum, Code, Abschlusszeit, Gründe der
Code-Prüfung, Begründungen der KI). Eine reine Feiditor-Datei und der Überblick derselben
Person werden automatisch zu einer Zeile zusammengeführt.

## Einrichtung

1. **GitHub Pages**: Repository → Settings → Pages → Branch `main`, Ordner `/ (root)`.
   Die Datei `CNAME` enthält bereits `check.janrickmer.de`.
2. **DNS** bei der Domain: `check` als `CNAME` auf `janrickmer.github.io` zeigen lassen
   (wie bei den anderen Subdomains). Danach in den Pages-Einstellungen „Enforce HTTPS“ aktivieren.
3. **PIN**: Die Seite fragt beim Öffnen eine PIN ab. Sie ist nur als SHA-256-Hash im Quelltext
   hinterlegt (Konstante `PIN_HASH`, Salt `PIN_SALT`). Neue PIN setzen:
   `sha256("check-des-feindtes|" + PIN)` als Hex eintragen. Achtung: Eine achtstellige Zahl ist
   kein echter Schutz gegen jemanden, der den Quelltext liest – die PIN hält nur Zufallsbesucher fern.
4. **Claude-Verbindung** (für die Spalte „Feiditor-Bewertung“): Oben rechts auf „Claude“ klicken
   und einen API-Schlüssel aus der Claude Console eintragen
   (<https://platform.claude.com/settings/keys>). Der Schlüssel wird ausschließlich im
   `localStorage` des jeweiligen Browsers gespeichert und geht nur an `api.anthropic.com`.

## Warum ein API-Schlüssel und nicht das claude.ai-Konto?

Eine statische Seite auf GitHub Pages kann sich nicht in ein claude.ai-Konto einloggen:
Anthropic erlaubt Drittanbieter-Apps keine Anmeldung über claude.ai-Konten und keine Nutzung
des Pro/Max-Abos. Der vorgesehene Weg ist ein API-Schlüssel aus der Claude Console
(Abrechnung nach Verbrauch über das Console-Guthaben). Die Seite ruft die Messages-API direkt
aus dem Browser auf (Header `anthropic-dangerous-direct-browser-access`). Daraus ergibt sich
genau die gewünschte Bindung an das Gerät: Wer die Seite woanders öffnet, findet keinen
Schlüssel vor und muss einen eigenen hinterlegen.

Richtwert Kosten: ca. 3–5 Tsd. Eingabe- und 2–4 Tsd. Ausgabe-Token je Schüler:in,
also grob 0,05–0,10 € pro Datei bei den aktuellen Opus-5.5-Preisen.

## Datenschutz

* Alle PDFs werden lokal im Browser gelesen und entschlüsselt; nichts wird hochgeladen.
* An die KI gehen nur: Kurszeile, Aufgabenstellungen, Antworttexte und die
  Lösungserläuterungen aus dem Überblick. Keine Namen, Geburtsdaten oder Codes.
* Leere Antworten werden ohne KI als „ungenügend“ eingeordnet.

## Technischer Hintergrund

* Die Feiditor-Tracking-Daten stehen als `%TRACKDATA`-Kommentar in der PDF, AES-GCM-verschlüsselt
  mit dem Feiditor-Lehrkraft-Code (Buchstabenzahl des Vornamens × 1104). Die Seite probiert die
  Buchstabenzahlen 1–60 durch – genau wie die Feiditor-Tabellenansicht.
* Der Überblick wird aus den unkomprimierten Textoperatoren der PDF gelesen
  (Labels `NAME`/`SCHÜLERIN / SCHÜLER`, `GEBURTSDATUM`, `BESTÄTIGUNGSCODE`, Kurszeile,
  „Lösungserläuterungen“).
* Bestätigungscode = Geburtsdatum als Zahl (TTMMJJJJ) × 1104 + Konstante des Escape-Rooms.
  Die Konstanten der bekannten Escape-Rooms stehen in `ROOM_MAGIC`; bei einem neuen Escape-Room
  dort eine Zeile ergänzen (Kurszeile → Konstante). Unbekannte Konstanten werden als Hinweis
  gemeldet, nicht als „nein“.
* Die Aufgaben werden anhand der vorbefüllten Systemzeilen („Aufgabe N“ + Aufgabentext) getrennt;
  reine Feiditor-Texte ohne Aufgaben werden als eine Aufgabe bewertet. Der Standalone-Feiditor
  verliert im Escape-Modus ab der ersten Eingabe die Systemmarkierung des nachfolgenden
  vorbefüllten Textes (die Aufgabentexte zählen dort als „eingefügt“). Die Seite erkennt die
  Struktur trotzdem, rechnet die Aufgabentexte aus dem Rot-Anteil heraus und nennt in den Details
  den ursprünglichen Wert der Datei.
* Namen werden tolerant verglichen: Buchstaben außerhalb von Windows-1252 (ş, ł, ć, ő …) stehen
  auf den PDF-Seiten als „?“ und gelten beim Abgleich mit den Feiditor-Daten nicht als Abweichung.
* PDFs, die von einem Viewer neu gespeichert oder gedruckt wurden, verlieren die Kommentarzeilen
  und werden als „nicht lesbar“ gemeldet (gleiche Einschränkung wie im Feiditor).
