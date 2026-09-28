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
| Feiditor-Bewertung | je Aufgabe „Feiditor-Nr. N: sehr gut … ungenügend“, bewertet von Google Gemini über die Gemini-API (Modell wählbar; voreingestellt das neueste stabile Flash-Modell) |

Ein Klick auf eine Zeile zeigt Details (Kurs, Geburtsdatum, Code, Abschlusszeit, Gründe der
Code-Prüfung, Begründungen der KI). Ein Klick auf die Prozentzahl öffnet die Feiditor-Ansicht:
der entschlüsselte Text mit jedem Zeichen so gefärbt wie in der Lehrkraft-Auswertung des Feiditors
(rot = eingefügt oder ≤ 0,015 s, orange = 0,015–0,07 s, grün = langsamer getippt, grau =
vorbefüllter Aufgabentext), samt Legende und Statistik. Eine reine Feiditor-Datei und der
Überblick derselben Person werden automatisch zu einer Zeile zusammengeführt.

## Einrichtung

1. **GitHub Pages**: Repository → Settings → Pages → Branch `main`, Ordner `/ (root)`.
   Die Datei `CNAME` enthält bereits `check.janrickmer.de`.
2. **DNS** bei der Domain: `check` als `CNAME` auf `janrickmer.github.io` zeigen lassen
   (wie bei den anderen Subdomains). Danach in den Pages-Einstellungen „Enforce HTTPS“ aktivieren.
   Die Seite funktioniert nur über `https://` (sonst fehlen dem Browser die Verschlüsselungsfunktionen;
   die Seite zeigt dann einen entsprechenden Hinweis).
3. **PIN**: Die Seite fragt beim Öffnen eine PIN ab. Sie ist nur als SHA-256-Hash im Quelltext
   hinterlegt (Konstante `PIN_HASH`, Salt `PIN_SALT`). Neue PIN setzen:
   `sha256("check-des-feindtes|" + PIN)` als Hex eintragen. Achtung: Eine achtstellige Zahl ist
   kein echter Schutz gegen jemanden, der den Quelltext liest – die PIN hält nur Zufallsbesucher fern.
4. **Gemini-Verbindung** (für die Spalte „Feiditor-Bewertung“): Oben rechts auf „Gemini“ klicken
   und den Google-API-Schlüssel aus Google AI Studio eintragen
   (<https://aistudio.google.com/app/apikey>). Der Schlüssel wird ausschließlich im
   `localStorage` des jeweiligen Browsers gespeichert – mit der PIN verschlüsselt (PBKDF2 mit
   200 000 Runden, AES-GCM) – und geht nur an `generativelanguage.googleapis.com`. Entschlüsselt
   liegt er nur im Arbeitsspeicher der geöffneten Seite; deshalb fragt die Seite bei jedem Öffnen die
   PIN ab. Wird die PIN im Quelltext geändert, muss der Schlüssel einmal neu eingetragen werden.
   Nach dem Speichern fragt die Seite die verfügbaren Modelle bei Google ab und bietet sie zur Auswahl an.

## KI-Anbindung über die Gemini-API

Die Seite ruft `generateContent` der Gemini-API direkt aus dem Browser auf (Header `x-goog-api-key`,
strukturierte JSON-Antwort über `responseSchema`). Ein claude.ai-Abo lässt sich für solche Aufrufe
nicht nutzen, und die Claude-API braucht Console-Guthaben; deshalb wurde auf Gemini umgestellt.
Die frühere Claude-Anbindung liegt in der Git-Historie (Commit `519b889`).

Was du zum Google-Kontingent wissen solltest (Stand September 2026, bitte in AI Studio prüfen):

* Google AI Studio stellt ein kostenloses Kontingent mit Tages- und Minutenlimits bereit
  (je nach Modell etwa 5–15 Anfragen pro Minute und 100–1500 pro Tag). Die Seite sendet deshalb
  höchstens eine Anfrage alle vier Sekunden und wiederholt bei „429“ automatisch.
* Für Nutzer:innen im EWR, in der Schweiz und im Vereinigten Königreich verlangt Google für die
  Gemini-API ein hinterlegtes Abrechnungskonto; dafür gelten dort die Datenbedingungen der
  bezahlten Dienste.
* Im unbezahlten Kontingent darf Google Eingaben und Antworten zur Produktverbesserung nutzen und von
  Menschen prüfen lassen; im bezahlten Kontingent nicht. Vor dem Senden von Schülertexten prüfen,
  welche Bedingungen für das eigene Projekt gelten.
* Kosten im bezahlten Kontingent: je Schülerdatei grob 3–5 Tsd. Eingabe- und unter 1 Tsd. Ausgabe-Token,
  bei Flash-Modellen also Bruchteile eines Cents.

## Datenschutz

* Alle PDFs werden lokal im Browser gelesen und entschlüsselt; nichts wird hochgeladen.
* An die KI gehen nur: Kurszeile, Aufgabenstellungen, Antworttexte und die
  Lösungserläuterungen aus dem Überblick. Keine Namen, Geburtsdaten oder Codes.
* Leere Antworten werden ohne KI als „ungenügend“ eingeordnet.
* Der System-Prompt kennzeichnet Aufgaben, Antworten und Lösungserläuterungen als Daten aus einer
  Schülerdatei, nicht als Anweisungen.

## Technischer Hintergrund

* Die Feiditor-Tracking-Daten stehen als `%TRACKDATA`-Kommentar in der PDF, AES-GCM-verschlüsselt
  mit dem Feiditor-Lehrkraft-Code (Buchstabenzahl des Vornamens × 1104). Die Seite probiert die
  Buchstabenzahlen 0–60 durch – genau wie die Feiditor-Tabellenansicht.
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
