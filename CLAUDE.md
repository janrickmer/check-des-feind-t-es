# Check des Feind(t)es – Hinweise für die Entwicklung

Auswertungsseite für Lehrkräfte (check.janrickmer.de): eine einzige Datei `index.html` ohne externe
Bibliotheken, PIN-geschützt, veröffentlicht über GitHub Pages aus dem Branch
`claude/check-des-feindes-site-njd9ja`. Sie liest die PDFs aus dem Feiditor (janrickmer/feiditor), dem
Lerntagebuch-Feiditor (janrickmer/lerntagebuch-feiditor) und den Escape-Rooms (janrickmer/escape-room-*),
zeigt den Rot-Anteil und lässt Escape-Room- und Feiditor-Antworten von einer KI einordnen. Oberfläche
und Dokumentation sind deutsch.

## Architektur-Vorgaben für Escape-Rooms (Pflicht – auch für jeden neuen Escape-Room)

Damit der Check die richtigen Angaben an die KI weitergeben kann, muss jeder Escape-Room so aufgebaut sein
wie die bestehenden (Vorlage: ein aktueller Raum, z. B. janrickmer/escape-room-jg6-Gewissen-und-Identitaet):

1. **Steckbrief `ROOM_CONTEXT`** im Escape-Room: `{id, fach, stufe, thema, raum, inhalte}`.
   - `stufe`: ausdrückliche Jahrgangsstufe („Jahrgangsstufe 8“, „Jahrgangsstufe 11 (Einführungsphase, E1)“,
     „Jahrgangsstufe 12 (Qualifikationsphase, Q2)“; Schule in Hessen, G9). Ist sie unklar, die Lehrkraft fragen.
   - `raum`: Name des Escape-Rooms, wie ihn die Schüler:innen sehen; `thema` in wenigen Worten.
   - `inhalte`: 5–8 kurze Punkte (je etwa ≤ 200 Zeichen) mit den im Escape-Room erarbeiteten Inhalten – nur aus
     den Räumen und Lösungserläuterungen (`SOLUTION_NOTES`) dieses Escape-Rooms, ausgerichtet auf die
     Feiditor-Aufgaben (`REFLECT_TASKS`). Jede Liste unabhängig gegen die Datei auf Faktentreue prüfen und der
     Lehrkraft vor dem Veröffentlichen zur Freigabe vorlegen.
2. **Übergabe an den eingebetteten Feiditor:** `openFeiditor()` setzt `window.__ESCAPE_DATA__` mit
   `firstName, lastName, birthDate, code, tasks: REFLECT_TASKS.slice(0,6), redFields: RED_FIELDS,
   ctx: ROOM_CONTEXT` (und `overview` für die Übersichtsseiten der kombinierten PDF). Der Feiditor speichert
   `ctx` verschlüsselt in der PDF (`e.ctx`), auch in Zwischenständen und beim Fortsetzen. Die fertige Abgabe
   heißt „Abgeschlossener Escape-Room von Vorname Nachname.pdf“ (Engine, `makePdfName()`; bis 03.10.2026
   „Escape Room Daten von …“), der Zwischenstand „Zwischenstand vom … von ….pdf“; `firstNameFromFile()` kennt
   beide Namen.
3. **Überblick-PDF:** Seiten „Überblick deiner Antworten & Erläuterungen“ und „Lösungserläuterungen“,
   Labels `NAME` (oder `SCHÜLERIN / SCHÜLER`), `GEBURTSDATUM`, `BESTÄTIGUNGSCODE`, `ABGESCHLOSSEN AM`,
   `ÜBERSICHT`, Fußzeile „Bestätigungscode für Moodle: …“, Kurszeile „Fach · Jahrgang · Escape-Room „Name““,
   Abschnitt „Verwendung von Zwischenstand-Dateien“. Als Kommentarzeile direkt nach dem PDF-Kopf (bis 04.10.2026
   hinter `%%EOF`): `%ESCAPEDATA` mit Base64-JSON
   `{v, n, t: REFLECT_TASKS.slice(0,6), r: RED_FIELDS, c: ROOM_CONTEXT}` (der Standalone-Feiditor übernimmt
   daraus im Escape-Modus Aufgaben und Steckbrief).
4. **Eingebetteter Feiditor:** die gemeinsame Engine unverändert (siehe unten) als
   `<script type="text/template" id="feiditor-src">` mit `<\/script>`; geöffnet per `document.write`
   (nicht über eine Blob-URL, sonst fehlt `crypto.subtle`).
5. **Eintrag im Check (`index.html` hier):**
   - `ESCAPE_ROOMS`: die normalisierten Anfänge (60 Zeichen) der ersten sechs `REFLECT_TASKS` und derselbe
     Steckbrief. Daran erkennt der Check Dateien ohne mitgespeicherten Steckbrief. Die Anfänge müssen über
     alle Escape-Rooms eindeutig sein (prüfen).
   - `KNOWN_MAGIC` und `ROOM_MAGIC` (Kurszeile → Prüfkonstante `MAGIC` des Raums) für die Code-Prüfung.
     Der Bestätigungscode entsteht im Escape-Room wie in den bestehenden Räumen über `computeCode()` mit einer
     eigenen, noch nicht vergebenen Prüfkonstante `MAGIC`.
6. **Testen vor dem Veröffentlichen:** echte Dateien aus dem echten Escape-Room erzeugen (Playwright,
   Headless-Chromium), im Check laden und prüfen: Code-Prüfung „ja“, `roomContextFor(row).src === "feiditor"`,
   die KI-Nachricht (`buildUserMessage`) enthält Jahrgangsstufe, Fach und alle Inhalte.

Der Check verwendet den Steckbrief in dieser Reihenfolge: verschlüsselte Feiditor-Daten (`e.ctx`) →
`ESCAPE_ROOMS` (Erkennung an den Aufgaben) → unverschlüsselter Überblick (`%ESCAPEDATA` c). Im System-Prompt
dienen die Inhalte als Erwartungshorizont; der Anspruch richtet sich nach der Jahrgangsstufe (bei einer
Spanne nach der niedrigsten).

## Gemeinsame Feiditor-Engine

- Eine Engine für elf Varianten: normaler Feiditor, Lerntagebuch-Feiditor und die in die neun Escape-Rooms
  eingebetteten Feiditoren. Sie bleibt in allen Varianten byte-identisch; Änderungen als exakte Ersetzungen,
  die in jeder Datei genau einmal vorkommen, in alle elf Dateien einspielen und prüfen.
- Unterschiede nur über `window.FEIDITOR_CONFIG` (vor der Engine) oder eigene Skripte danach
  (z. B. das Kalenderfeld bei Frage 1 im Lerntagebuch).
- Nutzdaten (`%TRACKDATA`, AES-GCM): `t` Text, `m` Quelle + Tipp-Abstand je Zeichen, `f` Format-Bits,
  `k` Blockstruktur, `v: 2` (alle Längen in UTF-16-Einheiten), `z` (1 = Zwischenstand, 0 = fertige Abgabe),
  `e` = `{esc, red, ctx}` im Escape-Modus.
- Die fertige Abgabe im Escape-Modus enthält auf jedem Weg die Überblicksseiten: im eingebetteten Feiditor neu
  erzeugt, im eigenständigen Feiditor aus der hochgeladenen Überblick- bzw. Zwischenstand-PDF übernommen (Content-
  Streams, in Feiditor-PDFs mit der Kommentarzeile `%ESCAPE-OVERVIEW` markiert). Die Zusammenführung zweier Dateien
  im Check bleibt für ältere Dateien (vor dem 04.10.2026) nötig.

## Weitere Regeln

- Nie Namen, Geburtsdaten oder Codes an die KI senden (`anonymize`/`safeData`); Schülertext steht zwischen
  `<<<` und `>>>`.
- API-Schlüssel nur PIN-verschlüsselt im localStorage dieses Browsers; nie im Chat erfragen oder anzeigen.
- Rot-Anteil ab 50 % rot und fett.
- Lerntagebücher: eigene Klassenansicht, keine KI-Bewertung; Datum aus dem Kalenderfeld, sonst aus der Antwort
  auf Frage 1, sonst aus dem Dateinamen.
- Escape-Rooms und Feiditoren sind für Schüler:innen live (Branch `main`): erst nach bestandenen Tests pushen.
- Diese Datei, README und `index.html` sind über check.janrickmer.de öffentlich abrufbar, die Repositories
  auf GitHub ebenfalls: keine Geheimnisse und keine Anleitungen zum Umgehen der Prüfungen hineinschreiben.
- Wie der Bestätigungscode und der Schlüssel der Feiditor-Daten gebildet werden, nie ausschreiben – weder in
  README/CLAUDE.md noch in erklärenden Code-Kommentaren oder Commit-Nachrichten. Nur auf die Funktionen
  verweisen; dort steht die Berechnung: `computeCode()` und `MAGIC` im Escape-Room, `checkCode()` /
  `CODE_FACTOR` und `autoDecrypt()` / `kfN()` im Check, `_kf()` / `_kfN()` in der Feiditor-Engine.
