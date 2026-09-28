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
| Feiditor-Bewertung | je Aufgabe „Feiditor-Nr. N: sehr gut … ungenügend“, bewertet von einem Sprachmodell über die API des gewählten Anbieters (voreingestellt OpenRouter mit kostenlosen Modellen; wahlweise Groq, Google Gemini oder ein anderer OpenAI-kompatibler Dienst; Modell wählbar) |

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
4. **KI-Verbindung** (für die Spalte „Feiditor-Bewertung“): Oben rechts auf „KI: nicht verbunden“
   klicken, den Anbieter wählen (Vorgabe: OpenRouter) und den API-Schlüssel eintragen. Jeder Schlüssel
   wird ausschließlich im `localStorage` des jeweiligen Browsers gespeichert – mit der PIN verschlüsselt
   (PBKDF2 mit 200 000 Runden, AES-GCM) – und geht nur an den gewählten Anbieter. Entschlüsselt
   liegt er nur im Arbeitsspeicher der geöffneten Seite; deshalb fragt die Seite bei jedem Öffnen die
   PIN ab. Wird die PIN im Quelltext geändert, müssen die Schlüssel einmal neu eingetragen werden.
   Nach dem Speichern fragt die Seite die verfügbaren Modelle beim Anbieter ab, schlägt das
   leistungsfähigste vor und bietet die übrigen zur Auswahl an.

## KI-Anbieter

Die Seite spricht die APIs direkt aus dem Browser an (kein eigener Server). Ein claude.ai-Abo lässt
sich dafür nicht nutzen, und die Claude-API braucht Console-Guthaben; die frühere Claude-Anbindung
liegt in der Git-Historie (Commit `519b889`), die reine Gemini-Fassung in Commit `005422a`.

**Kein kostenloses Kontingent ist unbegrenzt.** Alle Anbieter begrenzen Anfragen pro Minute und pro
Tag, manche zusätzlich Token pro Tag. Die Seite braucht genau eine Anfrage je Schülerdatei
(grob 3–5 Tsd. Eingabe- und unter 1 Tsd. Ausgabe-Token), hält je Anbieter einen Mindestabstand
zwischen den Anfragen ein, wiederholt bei „429“ nach der vom Anbieter genannten Wartezeit und weicht
bei erschöpftem oder ausgefallenem Modell automatisch auf das nächste der Rangliste aus (nur für die
laufende Sitzung; die Auswahl im Panel zeigt es als „Ausweichmodell“ bzw. „erschöpft“).

| Anbieter | Schlüssel | Limits (Stand September 2026, bitte beim Anbieter prüfen) | Hinweise |
|---|---|---|---|
| **OpenRouter** (Vorgabe) | kostenloses Konto, <https://openrouter.ai/settings/keys> | kostenlose Modelle (`:free`): 20 Anfragen/Minute, 50 Anfragen/Tag; nach einem einmaligen Guthabenkauf von 10 US-$ 1000 Anfragen/Tag. Gezählt werden Anfragen, keine Token. | Browser-Aufrufe (CORS) werden unterstützt. Die kostenlosen Modelle laufen bei Drittanbietern, die Eingaben protokollieren oder zum Training nutzen dürfen; das muss unter Settings → Privacy erlaubt sein, sonst antwortet die API „No endpoints found matching your data policy“. Vorgabe-Modell: das leistungsfähigste kostenlose (z. B. gpt-oss-120b). Ist das Tageslimit erreicht, meldet die Seite das ohne Durchprobieren anderer Modelle. |
| **Groq** | kostenloses Konto ohne Kreditkarte, <https://console.groq.com/keys> | je nach Modell etwa 30 Anfragen/Minute und 1000/Tag, dazu Token-Limits pro Minute und Tag (z. B. 200 000 Token/Tag bei gpt-oss-120b ≈ 40 Schülerdateien; 100 000 bei Llama 3.3 70B). Tageslimits gelten je Modell, daher wechselt die Seite bei erschöpftem Modell. | Sehr schnell. Ob Groq Aufrufe direkt aus dem Browser zulässt, konnte nicht geprüft werden – falls nicht, meldet der Verbindungstest „erlaubt keine Aufrufe direkt aus dem Browser (CORS)“; dann OpenRouter nutzen. Datenschutzbedingungen von Groq vor dem Senden von Schülertexten prüfen. |
| **Google Gemini** | Google AI Studio, <https://aistudio.google.com/app/apikey> | kostenloses Kontingent mit Minuten- und Tageslimits je Modell. Für viele Schlüssel liegt das Tageslimit der neuesten Modelle bei 0 („limit: 0“ noch vor der ersten Bewertung) – die Seite weicht dann auf das nächste Modell aus, für das der Schlüssel ein Kontingent hat. | Im kostenlosen Kontingent darf Google Eingaben zur Produktverbesserung nutzen und von Menschen prüfen lassen; im bezahlten nicht. Für den EWR, die Schweiz und das Vereinigte Königreich verlangt Google ein hinterlegtes Abrechnungskonto (dann gelten die Datenbedingungen der bezahlten Dienste). |
| **Anderer OpenAI-kompatibler Dienst** | je nach Dienst | je nach Dienst | Basis-URL ohne Schrägstrich am Ende (z. B. `https://api.cerebras.ai/v1`); der Dienst muss CORS erlauben. Ohne `GET …/models` wird die Modell-ID von Hand eingetragen. |

Technisch: OpenAI-kompatible Dienste werden über `POST {Basis-URL}/chat/completions` mit
`Authorization: Bearer …`, System- und Nutzer-Nachricht, `temperature 0.2` und
`response_format: {"type":"json_object"}` aufgerufen (lehnt ein Modell den JSON-Modus mit HTTP 400 ab,
wird ohne wiederholt); die Antwort wird tolerant gelesen (Markdown-Zäune, `<think>`-Blöcke, Text
drumherum, Noten in Groß-/Kleinschreibung). Gemini läuft weiter über `generateContent` mit
`responseSchema`. Modelllisten kommen von `GET …/models` (OpenRouter zusätzlich `GET …/auth/key`
zur Schlüsselprüfung, weil `/models` dort öffentlich ist); Nicht-Textmodelle (Whisper, TTS,
Embeddings, Bild, Guard) werden ausgeblendet, kleine oder reine Reasoning-Modelle ans Ende sortiert.

## Datenschutz

* Alle PDFs werden lokal im Browser gelesen und entschlüsselt; nichts wird hochgeladen. Schlüssel werden je Anbieter getrennt gespeichert (`cdf_<anbieter>_api_key_enc`), der gewählte Anbieter unter `cdf_provider`.
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
