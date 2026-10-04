# Check des Feind(t)es

Auswertungsseite für die Lehrkraft: <https://check.janrickmer.de>

Eine einzige HTML-Datei (`index.html`) ohne externe Bibliotheken. Sie ersetzt die frühere
Lehrkraft-Ansicht des Feiditors und liest alle Feiditor-Dateien direkt im Browser aus:
kombinierte Dateien aus den Escape-Rooms („Abgeschlossener Escape-Room von …“, vor dem 04.10.2026
„Escape Room Daten von …“), Escape-Room-Überblicke
(„Escape-Room-Überblick_von_…“), Dateien aus dem normalen Feiditor („Text von …“, mit oder ohne
eingetragene Aufgabenstellungen), Lerntagebuch-Einträge („Lerntagebucheintrag_…“) und
Zwischenstände („Zwischenstand vom … von …“). Auch die „Auswertungsdatei_….html“, die ältere
Feiditor-Versionen (Juni bis August 2026) zusätzlich ausgegeben haben, wird gelesen – nur die darin
eingebetteten verschlüsselten Daten, nichts daraus wird ausgeführt. Sie zeigt eine Tabelle:

| Spalte | Inhalt |
|---|---|
| Nachname / Vorname | aus den verschlüsselten Feiditor-Daten (sonst aus dem Überblick) |
| Code passt? | **ja** (grün) / **nein** (rot, fett): Bestätigungscode ↔ Geburtsdatum des Escape-Room-Überblicks, Konstante des jeweiligen Escape-Rooms, Namensabgleich Feiditor-Daten ↔ Überblick. „–“ = Datei enthält keinen Code (reiner Feiditor). |
| Rote Buchstaben | Rot-Anteil wie in der Feiditor-Lehrkraft-Auswertung (eingefügt oder ≤ 0,015 s getippt); ab 50 % rot und fett |
| Feiditor-Bewertung | je Aufgabe „Feiditor-Nr. N: sehr gut … ungenügend“, bewertet von einem Sprachmodell über die API des gewählten Anbieters (voreingestellt OpenRouter mit kostenlosen Modellen; wahlweise Groq, Google Gemini oder ein anderer OpenAI-kompatibler Dienst; Modell wählbar). Das Notenwort ist farbig: sehr gut hellgrün, gut dunkelgrün, befriedigend gelb, ausreichend orange, mangelhaft und ungenügend rot. |

Ein Klick auf eine Zeile zeigt Details (Dateiart, Kurs, Geburtsdatum, Code, Abschlusszeit, Gründe
der Code-Prüfung, Begründungen der KI mit farbigem Notenwort). Ein Klick auf die Prozentzahl öffnet
die Feiditor-Ansicht: der entschlüsselte Text mit jedem Zeichen so gefärbt wie in der
Lehrkraft-Auswertung des Feiditors (rot = eingefügt oder ≤ 0,015 s, orange = 0,015–0,07 s,
grün = langsamer getippt, schwarz = vorgegebener Aufgabentext, nicht gewertet), samt Legende und
Statistik. Ein Klick auf einen Spaltenkopf sortiert die Tabelle, zum Beispiel nach Rot-Anteil
(höchster zuerst, wie in der Tabellenansicht des Feiditors) oder nach Durchschnittsnote; der Browser
merkt sich die zuletzt gewählte Sortierung. Haben fast alle Zeichen einer Datei einen Tippabstand
von genau 0 ms, ohne als eingefügt markiert zu sein, nennt die Detailzeile mögliche Ursachen
(Rückgängig/Wiederholen nach dem Einfügen, Zwischenablage-Leiste einer Handytastatur, Drag-and-drop,
Diktierfunktion, Autovervollständigung, Datenverlust älterer Feiditor-Versionen) – ohne sie zu werten.
Enthält eine Datei aus einer Feiditor-Version vor der Korrektur vom 29.09.2026 ein Emoji, weist die
Detailzeile darauf hin, dass der Rot-Anteil zu niedrig sein kann: Diese Versionen verloren nach einem
Emoji bei einem Teil der später getippten Zeichen das Tipp-Tempo (siehe unten). Dateien der
korrigierten Version (Datenversion 2) sind davon nicht betroffen und bekommen keinen Hinweis.

Zusammenführen: Eine Feiditor-Datei mit Escape-Room-Aufgaben, aber ohne Code (vor dem 04.10.2026:
Standalone-Feiditor im Escape-Modus, Pulsar-Raum, fortgesetzter Zwischenstand – seitdem enthält jede fertige
Abgabe die Überblicksseiten) und der Überblick derselben Person aus
demselben Escape-Room (Abgleich der Aufgabentexte) werden zu einer Zeile zusammengeführt; fertige
Abgaben haben dabei Vorrang vor Zwischenständen. Wird zur kombinierten Datei zusätzlich der separate
Überblick mit demselben Code hochgeladen, entsteht keine zweite Zeile. Dieselbe Abgabe doppelt (etwa
PDF und ältere Auswertungsdatei) erscheint und zählt nur einmal. Normale Feiditor-Dateien und
Lerntagebuch-Einträge werden nie mit einem Überblick zusammengeführt. Zwischenstände sind in der
Tabelle markiert.

**Zwischenstand oder fertige Abgabe:** Seit dem 29.09.2026 schreiben alle Feiditoren (Standalone,
Lerntagebuch, alle Escape-Rooms) eine versteckte Kennung in die verschlüsselten Daten der PDF:
„Zwischenstand herunterladen und speichern“ → Zwischenstand, „Text in PDF-Datei umwandeln“ → fertige
Abgabe. Diese Kennung ist maßgeblich; die Details nennen sie („laut verschlüsselter Kennung …“).
Passt der Dateiname nicht dazu, wurde die Datei umbenannt – die Details weisen darauf hin. Ältere
Dateien ohne Kennung werden weiterhin am Dateinamen („Zwischenstand vom …“) erkannt.
Zur Sicherheit: Die Kennung steckt mit allen anderen Tipp-Daten im selben AES-GCM-verschlüsselten
Block. Wer nur die PDF bearbeitet, kann sie nicht ändern, ohne dass die Datei unlesbar wird. Weil
der Feiditor aber im Browser läuft und sein Quelltext öffentlich ist, könnte jemand mit
Programmierkenntnissen die Daten entschlüsseln und neu verschlüsseln – derselbe Schutz wie bei
allen übrigen Tipp-Daten (Rot-Anteil, Einfüge-Markierungen).

**Geladene Spielstände im Escape-Room:** Seit dem 29.09.2026 lässt sich ein Escape-Room als
Zwischenstand-Datei (.txt) speichern und später weiterspielen. Der Überblick vermerkt am Ende der
Lösungserläuterungen unter „Verwendung von Zwischenstand-Dateien“, ob und wann eine solche Datei
geladen wurde und ab welchem Raum es weiterging. Die Details zeigen diesen Vermerk
(„Zwischenstand-Dateien im Escape-Room“); an die KI geht er nicht.

Gelöschte und ergänzte Aufgaben: Im neuen Feiditor lassen sich Aufgabenblöcke mit dem roten ×
entfernen und eigene ergänzen. Fehlt in einer Escape-Room-Datei eine Aufgabennummer oder im Lerntagebuch
eine der fünf Fragen, erscheint sie als „im Feiditor gelöscht“ und wird ohne KI „ungenügend“. Selbst
ergänzte Aufgaben bekommen die nächste freie Nummer, stehen als „(ergänzt)“ in der Tabelle und zählen
nicht zur Durchschnittsnote. Selbst eingetragene Aufgabentexte färbt der Feiditor nicht ein; ist einer
ungewöhnlich lang, weist die Detailzeile darauf hin.

Drucken: Die Tabelle lässt sich direkt aus dem Browser drucken (A4 hoch); Kopfleiste, Upload-Feld
und Schaltflächen werden dabei ausgeblendet, die Notenfarben bleiben erhalten.

**Kontext für Dateien ohne Escape-Room:** Normale Feiditor-Dateien nennen weder Fach noch Jahrgang. Unter dem Feld zum Hochladen lässt sich dafür ein Kontext eintragen (etwa
„Ethik, Klasse 8 – Thema Gewissen“); die KI erhält ihn für diese Dateien. Ohne Angabe schließt sie
das Niveau aus Aufgaben und Antworten und bewertet im Zweifel nach Sekundarstufe I.

**Jahrgangsstufe und Inhalte aus dem Escape-Room:** Jeder Escape-Room hat einen Steckbrief (Fach,
Jahrgangsstufe, Thema, Name des Escape-Rooms und 8 im Escape-Room erarbeitete Inhalte). Der Feiditor
übernimmt ihn beim Öffnen aus dem Escape-Room und speichert ihn verschlüsselt in der PDF – auch in
Zwischenständen, beim Fortsetzen und im normalen Feiditor, wenn dort die Überblick-PDF hochgeladen wird
(Escape-Rooms ab 30.09.2026). Die KI erhält Jahrgangsstufe, Fach, Escape-Room und die Inhalte und nutzt sie
als Erwartungshorizont: Fehlen zentrale passende Inhalte oder sind sie falsch wiedergegeben, führt das zu
Abzug; richtiges Wissen darüber hinaus wertet auf; der Anspruch richtet sich nach der Jahrgangsstufe (beim
Pulsar-Raum, Jahrgangsstufen 9–12, im Zweifel nach der niedrigsten). Ältere Dateien ohne Steckbrief erkennt
die Seite an ihren Feiditor-Aufgaben und ergänzt den Steckbrief aus ihrer eigenen Liste (`ESCAPE_ROOMS`) –
auch bei älteren Dateien des Pulsar-Raums (vor dem 04.10.2026), deren Feiditor-Datei keine Überblicksseiten hat. Was die KI erhalten hat und woher
es stammt, steht in den Details unter „Angaben für die KI“.

| Escape-Room | Fach | Jahrgangsstufe |
|---|---|---|
| Lumo und die innere Stimme | Ethik | 6 |
| Das Zeitportal – Die drei Schlüssel der Würde | Ethik | 7 |
| Der Sneaker vor der Haustür | Politik und Wirtschaft | 7 |
| Der Fall @lichtenberg.leaks | Politik und Wirtschaft | 8 |
| Europäischer Jugendgipfel | Politik und Wirtschaft | 9 |
| EU-Klimagipfel | Politik und Wirtschaft | 11 (Einführungsphase, E2) |
| Das Gutachten | Evangelische Religion | 11 (Einführungsphase, E1) |
| Stunde Null | Politik und Wirtschaft | 12 (Qualifikationsphase, Q2) |
| Das Vermögensgeheimnis (Pulsar, Projektwoche Finanzielle Bildung) | Politik und Wirtschaft | 9–12 (gemischte Gruppe) |

Neue Escape-Rooms müssen genauso aufgebaut sein; die Checkliste steht in `CLAUDE.md`.

**Lerntagebücher einer Klasse:** Alle Lerntagebuch-Einträge einer Klasse (etwa am Schuljahresende)
lassen sich auf einmal hochladen. Sie erscheinen nicht in der Haupttabelle, sondern in einer eigenen
Tabelle „Lerntagebücher“, je Person eine Zeile:

| Spalte | Inhalt |
|---|---|
| Nachname / Vorname | aus den verschlüsselten Feiditor-Daten; zugeordnet wird über den vollen Namen (Groß-/Kleinschreibung und Leerzeichen spielen keine Rolle, ältere Dateien mit nur einem Gesamtnamen passen dazu), angezeigt wird die häufigste Schreibweise |
| Rote Buchstaben (alle Einträge) | rot markierte Zeichen aller Einträge dieser Person geteilt durch alle ihre Zeichen (ohne Leerzeichen und ohne die vorgegebenen Fragen) – längere Einträge zählen entsprechend mehr; ab 50 % rot und fett |
| Eingereichte Dateien | Anzahl der Einträge; Zwischenstände zählen wie fertige Abgaben, dieselbe Abgabe doppelt (etwa PDF und ältere Auswertungsdatei) zählt einmal |

Ein Klick auf eine Person zeigt ihre Sitzungen nach Datum sortiert: Datum der Sitzung, Rot-Anteil
dieses Eintrags (ab 50 % rot und fett; ein Klick öffnet die Feiditor-Ansicht mit dem farbig
markierten Text) und die Zahl der Zeichen ohne Aufgabenstellungen. Liegen mehrere Dateien zum selben
Datum vor, steht das dabei. Hinweise zu einzelnen Einträgen (Emoji in älteren Versionen, fast alles mit
0 ms Abstand, veränderter Rot-Anteil) stehen in der aufgeklappten Sitzung; ein „!“ neben dem Gesamtwert
zeigt, dass es bei dieser Person solche Hinweise gibt. Die Spaltenköpfe sortieren auch hier. Lerntagebücher
werden nicht von der KI bewertet.

Selbst eingefügte Aufgabenblöcke: Ältere Lerntagebuch-Versionen hatten den Button „Neuen Aufgabentext
eintragen“. Text darin gilt im Feiditor als vorgegebener Aufgabentext (schwarz, nicht gezählt), und der
Feiditor unterscheidet dort nicht zwischen Tippen und Einfügen. Damit darüber kein eingefügter Text
verschwindet, zählt diese Seite solchen Text (ganze Zeilen aus Systemtext, die weder eine der fünf Fragen
noch das Datum sind) als eingefügt, also rot, und nennt das in der Sitzung. Im aktuellen
Lerntagebuch-Feiditor gibt es den Button nicht mehr.

Datum der Sitzung:
1. Seit dem 30.09.2026 wählen die Schüler:innen das Datum bei Frage 1 im Lerntagebuch-Feiditor über
   ein Kalenderfeld aus (kein Eintippen). Es steht als zweite Zeile im festen Block von Frage 1
   („Dienstag, 29.09.2026“), erscheint so auch in der PDF und wird hier ausgelesen. Die fertige PDF
   lässt sich erst mit Datum erzeugen; Zwischenstände jederzeit.
2. Ältere Einträge: Die Seite liest das Datum aus der getippten Antwort auf Frage 1 („29.09.2026“,
   „29.9.26“, „29. September 2026“, „29.09.“ …; fehlt die Jahreszahl, gilt das letzte passende Datum bis
   zum Tag der PDF-Erstellung; Uhrzeiten wie „10.30 Uhr“ werden nicht als Datum gelesen).
3. Sonst aus dem Dateinamen („Lerntagebucheintrag_…_TTMMJJJJ.pdf“, „Textdatei_…“, „Zwischenstand vom
   TT.MM.JJJJ …“, auch mit Moodle-Präfix) – das ist der Tag, an dem die PDF erstellt wurde. Die PDFs des
   Feiditors enthalten keine Metadaten mit Datum.

Woher das Datum stammt, steht in der aufgeklappten Zeile, wenn es nicht aus dem Kalenderfeld kommt.
Liegt ein getipptes Datum nach dem Tag der PDF-Erstellung oder mehr als 200 Tage davor, weist die Zeile
darauf hin. 600 Dateien werden in wenigen Sekunden eingelesen.

**Feiditor:** Der normale Feiditor (feiditor.janrickmer.de) und der Lerntagebuch-Feiditor haben keinen
Lehrkraft-Zugang mehr in der Fußzeile; die Auswertung läuft vollständig über diese Seite. Die in die
Escape-Rooms eingebetteten Feiditoren haben ihn noch.

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
* An die KI gehen nur: Art der Datei, Kurszeile bzw. (nur bei Dateien ohne Escape-Room) der Kontext
  der Lehrkraft, bei Escape-Room-Dateien der Steckbrief des Escape-Rooms (Jahrgangsstufe, Fach, Thema,
  erarbeitete Inhalte), Aufgabenstellungen, Antworttexte, ein eventueller Text vor der ersten Aufgabe (als
  eigener Abschnitt) und die Lösungserläuterungen aus dem Überblick. Keine Geburtsdaten oder Codes.
  Schreibt eine Schülerin oder ein Schüler den eigenen Namen in den Text, wird er vor dem Senden durch
  „[Name]“ ersetzt: der volle Name überall, der Vorname als eigenes Wort, der Nachname allein nur im
  Text vor der ersten Aufgabe (sonst könnte er ein normales Wort wie „klein“ oder „Wolf“ sein).
* Leere Antworten werden ohne KI als „ungenügend“ eingeordnet.
* Der System-Prompt kennzeichnet Aufgaben, Antworten und Lösungserläuterungen als Daten aus einer
  Schülerdatei, nicht als Anweisungen. Aufgabenstellung und Antwort stehen jeweils zwischen `<<<` und
  `>>>`; gleichlautende Zeichenfolgen im Schülertext werden vorher entschärft, damit niemand über eine
  selbst eingetragene Aufgabenstellung einen falschen Antwortblock unterschieben kann.

## Technischer Hintergrund

* Die Feiditor-Tracking-Daten stehen als `%TRACKDATA`-Kommentar in der PDF, AES-GCM-verschlüsselt.
  Die Seite entschlüsselt sie selbst, wie früher die Feiditor-Tabellenansicht (siehe `autoDecrypt`).
* Der Überblick wird aus den unkomprimierten Textoperatoren der PDF gelesen
  (Labels `NAME`/`SCHÜLERIN / SCHÜLER`, `GEBURTSDATUM`, `BESTÄTIGUNGSCODE`, Kurszeile,
  „Lösungserläuterungen“).
* Der Bestätigungscode hängt vom Geburtsdatum und von einer Prüfkonstante des jeweiligen
  Escape-Rooms ab (Berechnung: `computeCode()` im Escape-Room, `checkCode()` hier).
  Die Konstanten der bekannten Escape-Rooms stehen in `ROOM_MAGIC`; bei einem neuen Escape-Room
  dort eine Zeile ergänzen (Kurszeile → Konstante), ebenso in `KNOWN_MAGIC` und – mit Aufgaben und
  Steckbrief – in `ESCAPE_ROOMS` (siehe `CLAUDE.md`). Unbekannte Konstanten werden als Hinweis
  gemeldet, nicht als „nein“.
* Gemeinsame Feiditor-Engine (ab 28.09.2026, Standalone, Lerntagebuch und Escape-Rooms): Die
  Nutzdaten enthalten die Blockstruktur `k` (je Block Art 0 = Zeile / 1 = Aufgabenblock,
  Zeichenzahl, Labellänge) und das Format-Bit 16 für Aufgabentext. Aufgabenblöcke sind fest
  eingefügte Aufgabentexte: im Escape-Room mit dem Label „Aufgabe N“, im normalen Feiditor
  (Button „Neuen Aufgabentext eintragen“) und im Lerntagebuch ohne Label. Alle Zeilen bis zum
  nächsten Aufgabenblock sind die Antwort; Text vor der ersten Aufgabe (etwa eine Überschrift)
  geht als eigener Abschnitt an die KI. `k` zählt seit dem 29.09.2026 (Datenversion `v: 2`) wie der
  Text in UTF-16-Einheiten, davor in Unicode-Zeichen (Codepunkten), in denen ein Emoji eins statt zwei
  belegt; die Seite liest beide Zählweisen. Beschriftete Aufgaben behalten ihre
  Nummer, selbst ergänzte bekommen die nächste freie Nummer, sonst wird fortlaufend nummeriert.
  Nach dem Fortsetzen eines alten Zwischenstands kann ein Aufgabenblock mehrere frühere Aufgaben
  enthalten; er wird an „Aufgabe N“-Zeilen bzw. an den Lerntagebuch-Fragen geteilt. Passt `k` nicht
  zum Text, gelten die älteren Regeln unten.
* Ältere Dateien: Die Aufgaben werden anhand der vorbefüllten Systemzeilen („Aufgabe N“ +
  Aufgabentext) getrennt; Systemtext ohne „Aufgabe N“ (etwa Lerntagebuch-Fragen mit Marke `q`)
  bildet je zusammenhängendem Abschnitt eine Aufgabe; Texte ganz ohne Aufgaben werden als eine
  Aufgabe bewertet. Nummeriert eine Aufgabenstellung sich selbst („1) …“ im Lerntagebuch, „Aufgabe 3 …“),
  gilt diese Nummer; „9. November …“ ist ein Datum und keine Nummer. Ältere Dateien speichern nur den Gesamtnamen; bei Doppel-Vornamen hilft der Dateiname
  („Lerntagebucheintrag_Vorname_Nachname_TTMMJJJJ.pdf“) bei der Aufteilung. Der ältere Standalone-Feiditor
  verliert im Escape-Modus ab der ersten Eingabe die Systemmarkierung des nachfolgenden
  vorbefüllten Textes (die Aufgabentexte zählen dort als „eingefügt“). Die Seite erkennt die
  Struktur trotzdem, rechnet die Aufgabentexte aus dem Rot-Anteil heraus und nennt in den Details
  den ursprünglichen Wert der Datei.
* Namen werden tolerant verglichen: Buchstaben außerhalb von Windows-1252 (ş, ł, ć, ő …) stehen
  auf den PDF-Seiten als „?“ und gelten beim Abgleich mit den Feiditor-Daten nicht als Abweichung.
* `%TRACKDATA` wird nur im Kopf der PDF (vor dem ersten Objekt) und `%ESCAPEDATA` nur hinter dem
  letzten `%%EOF` gelesen – genau dort schreiben sie Feiditor und Escape-Room hin. Schülertext steht in
  der PDF immer in Klammern innerhalb eines Textstroms davor, deshalb kann niemand eigene
  „Escape-Daten“ in seinen Text tippen oder einfügen (auch nicht mit einem Wagenrücklauf).
* Emoji-Fehler der gemeinsamen Feiditor-Engine (28.09.–29.09.2026, seit 29.09.2026 in allen elf
  Feiditoren behoben): `serializeEditor` zählte Zeichen als Unicode-Codepunkte (`for (const ch of s)`),
  der Text `t`, die Tipp-Daten und `buildPayload` arbeiten aber mit UTF-16-Einheiten. Nach einem Emoji
  (zwei UTF-16-Einheiten) waren Format-Bits, Aufgaben-Markierung und Blockstruktur verschoben;
  `applySystemFlags` setzte dadurch bei späteren Zeichen das Tipp-Tempo zurück (im Test fiel der
  Rot-Anteil einer komplett getippten Escape-Room-Antwort mit einem Emoji von 97 % auf 30 %), und beim
  Fortsetzen eines solchen Zwischenstands gerieten Aufgabenblöcke durcheinander. Die korrigierte Engine
  zählt je UTF-16-Einheit, schreibt `v: 2` und rechnet beim Fortsetzen älterer Dateien die
  Blockstruktur um. Diese Seite liest die Blockstruktur in beiden Zählweisen.
* PDFs, die von einem Viewer neu gespeichert oder gedruckt wurden, verlieren die Kommentarzeilen
  und werden als „nicht lesbar“ gemeldet (gleiche Einschränkung wie im Feiditor).
