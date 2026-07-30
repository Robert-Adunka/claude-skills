---
name: vorgespraech
description: Verarbeitet Transkripte von Vorgesprächen (Discovery-/Verkaufsgespräche) zu Schulungen, Workshops, Projekten oder Mitgliedschaften mit Interessenten. Triggert auf "/vorgespraech", "Vorgespräch verarbeiten", "Discovery Call", "Erstgespräch", "Akquisegespräch", oder wenn Robert ein Transkript oder eine Audioaufnahme (.m4a, .mp3, .wav) eines Vor-/Verkaufsgesprächs mit einem Interessenten übergibt. NICHT für bestehende TRIZ-Mastery-Hub-Mitglieder verwenden — dafür ist der Skill member-interview zuständig.
version: 1.0.0
---

# Vorgespräch Processor

Dieser Skill verarbeitet das Transkript eines Vorgesprächs (Discovery- oder Verkaufsgespräch) mit einem Interessenten zu Schulungen, Workshops, Projekten oder einer Mitgliedschaft. Transkripte können deutsch oder englisch sein. Führe die Schritte ohne Rückfrage nacheinander aus — bei einer Audiodatei als Input beginnend mit Schritt 0, sonst mit Schritt 1.

**Abgrenzung:** Dieser Skill ist für Interessenten VOR einem Abschluss/Beitritt. Bestehende TRIZ-Mastery-Hub-Mitglieder werden mit dem Skill `member-interview` verarbeitet (der hat einen zusätzlichen Circle-Schritt). Wichtige Unterschiede zu member-interview: (1) kein Circle-Schritt, (2) eigene Frontmatter-Felder, (3) eigener Ablageort.

## Input

Drei Formen:

**Dateipfad** — Robert gibt einen Pfad zum Transkript an:
```
/vorgespraech /path/to/transcript.txt
```

**Inline** — Robert fügt das Transkript direkt in den Chat ein (mit oder ohne `/vorgespraech`). Wenn im Text oder Kontext kein Datum steht, heutiges Datum für den Dateinamen verwenden.

**Audiodatei** — Robert gibt einen Pfad zu einer Aufnahme statt zu einem Transkript an (`.m4a`, `.mp3`, `.wav`, `.mp4`):
```
/vorgespraech /path/to/AndreasNeufing.m4a
```
Dann zuerst Schritt 0 ausführen, danach normal weiter mit Schritt 1.

## Schritt 0 — Audio transkribieren (nur bei Audio-Input)

Diesen Schritt überspringen, wenn bereits ein Transkript vorliegt.

**Erst prüfen, ob es eine Aufzeichnung der Konferenzplattform gibt.** Teams und Zoom zeichnen pro Teilnehmer eine eigene Tonspur auf, ihre Transkripte haben deshalb eine exakte Sprecherzuordnung statt einer geschätzten. Wenn ein solches Transkript vorliegt, ist es der bessere Ausgangspunkt — dann Schritt 0 überspringen. Robert nutzt es selbst, wenn er es hat; meist hat er nur eine Audiodatei.

Transkription läuft lokal mit Whisper, es wird nichts hochgeladen:

```bash
whisper "<Audiodatei>" --model large-v3-turbo --language de --output_format srt --output_dir "/Users/robert/Documents/TCG-Ordner/_Adunka/Vorgespraeche/"
```

- **Ein Lauf, nicht zwei.** Das blanke `whisper` transkribiert vollständig, kann aber keine Sprecher unterscheiden. Die Zuordnung wird in Schritt 1 aus dem Inhalt erschlossen — bei einem Vorgespräch ist das gut möglich, weil Robert als TRIZ-Experte und Anbieter inhaltlich klar vom Interessenten zu trennen ist. Das ist Roberts ausdrückliche Entscheidung: Sprechererkennung ist den Mehraufwand hier nicht wert.
- **Modell:** immer `large-v3-turbo`.
- **Zeitstempel:** `--output_format srt`. Die Zeitmarken werden gebraucht, weil die `_corrected`-Dateien das Format `Sprechername / mm:ss / Text` haben.
- **Sprache:** `--language de` bei deutschen Gesprächen, `--language en` bei englischen. Wenn unklar, den Parameter weglassen.
- **Laufzeit:** grob die Länge der Aufnahme bis zum Doppelten davon auf CPU. Immer im Hintergrund starten und nicht blockierend warten. Vorab mit `ffprobe` die Dauer ermitteln und Robert eine Zeitschätzung nennen. Whisper puffert seine Ausgabe, wenn sie nicht auf ein Terminal geht — eine leere Logdatei heißt nicht, dass der Lauf hängt.
- **Ergebnis:** Whisper legt `<Basisname>.srt` im Zielordner ab. Diese Datei ist das **Roh-Transkript** und geht so in Schritt 1 und 2. In Schritt 2 wird sie auf `..._raw.txt` umbenannt (Inhalt unverändert, inklusive Zeitmarken); die von Whisper erzeugte Datei danach entfernen, damit im Ordner keine Dublette liegt.
- **Datum:** Audiodateinamen enthalten meist kein Datum. Dann das Änderungsdatum der Audiodatei als Gesprächsdatum verwenden, sonst heute.
- **Plausibilitätsprüfung:** grob 130 bis 140 Wörter pro Gesprächsminute sind normal. Deutlich weniger heißt, dass etwas schiefgelaufen ist.
- **Halluzinierte Schlusssätze:** Whisper erfindet auf Stille am Dateiende gern einen Satz ("Und diese Show wird schon toll sein.", "Vielen Dank fürs Zuschauen."). Steht am Ende ein Satz, der nicht zum Gespräch passt, ersatzlos streichen — das ist kein Hörfehler, sondern Erfindung.

Whisper liefert keine Sprecherkennung, der Text läuft als ein Block durch. Aus den SRT-Zeitmarken den **Startzeitpunkt jedes Sprecherblocks als `mm:ss`** übernehmen, damit das bereinigte Transkript dasselbe Format hat wie die von Transkriptionsdiensten erzeugten Dateien:

```
Robert Adunka
00:19
So, wunderschönen guten Morgen.
```

### Sprechererkennung mit WhisperX — nur auf Anforderung

WhisperX ist eingerichtet (`~/.venvs/whisperx/`) und kann Sprecher trennen, wird aber **standardmäßig nicht verwendet**. Es lohnt sich nur, wenn die Zuordnung aus dem Inhalt schwerfällt — etwa bei mehr als zwei Teilnehmern oder wenn sich die Rollen inhaltlich nicht klar trennen lassen. Einsatz nur, wenn Robert es verlangt.

Wenn es zum Einsatz kommt, ist das Wichtigste: **WhisperX allein liefert kein vollständiges Transkript.** Seine Sprachaktivitätserkennung schneidet Text weg, gemessen an derselben 26-Minuten-Aufnahme (blankes `whisper` = 3528 Wörter als Referenz):

- pyannote (Voreinstellung): 2738 Wörter, 22 % fehlen
- pyannote mit abgesenkten `--vad_onset`/`--vad_offset`: bringt nichts
- `--vad_method silero`: 3065 Wörter, 13 % fehlen

Die Lücken sind im Fließtext nicht zu sehen. Deshalb in dem Fall **den Text aus dem Whisper-Lauf behalten** und aus WhisperX nur Sprecherlabels und Zeitmarken übernehmen:

```bash
/Users/robert/.venvs/whisperx/bin/whisperx "<Audiodatei>" \
  --model large-v3-turbo --language de \
  --device cpu --compute_type int8 \
  --vad_method silero \
  --diarize --min_speakers 2 --max_speakers 2 \
  -f srt --output_dir "<Zielordner>"
```

- **Sprecherzahl:** Robert fragen, wie viele Personen auf der Aufnahme sprechen — er kann das nach dem Gespräch sagen. Die Zahl in `--min_speakers`/`--max_speakers` eintragen. Bei Unsicherheit beide weglassen und die gefundene Zahl auf Plausibilität prüfen. **Nicht** auf kurze Ausschnitte anwenden, in denen eine Person kaum spricht: das Clustering teilt dann den dominanten Sprecher in zwei auf.
- **Zugang:** lädt `pyannote/speaker-diarization-community-1`, ein zugangsbeschränktes Modell. Roberts HuggingFace-Token liegt in `~/.cache/huggingface/`. Die Warnung `No --hf_token provided` ist irreführend und kann ignoriert werden.
- **Laufzeit:** mit Silero rund 36 Minuten für 26 Minuten Audio, also länger als der reine Whisper-Lauf. Fortschrittsbalken per `grep -viE "%\||MB/s"` filtern.
- **Güte:** lange Redebeiträge sitzen zuverlässig, die Label-Zuordnung bleibt über die ganze Aufnahme stabil. Kurze Einwürfe ("Ah, okay", "Ja, genau", "Mhm") landen oft beim falschen Sprecher — rund 8 Prozent der Segmente. Die kurzen also gegenlesen.
- **Zusammenführen:** Blöcke des bereinigten Textes der Reihe nach gegen die WhisperX-Segmente matchen; für Blöcke ohne Treffer die Zeit zwischen den benachbarten Treffern **interpolieren**, nicht hochzählen, sonst läuft das Ende über die Aufnahmelänge hinaus. Vor dem Speichern prüfen, dass der Text Wort für Wort unverändert ist und nur Zeitmarken dazugekommen sind.

## Schritt 1 — Transkript bereinigen

Lies das Roh-Transkript. Wende **nur die minimal nötigen Korrekturen** für Speech-to-Text-Fehler an. Nicht paraphrasieren, keine Grammatik glätten, keine Füllwörter entfernen, Stil nicht ändern.

**TRIZ-Begriffe** (nur wenn der Kontext eindeutig die Methodik meint):
- "Tris", "trees", "Chris", "Pris", "twist" → TRIZ
- "Arize", "arise", "a rise", "a riz" → ARIZ

**Personennamen** (phonetische Varianten korrigieren):
- Varianten → Genrikh Altshuller, Boris Zlotin, Alla Zusman, Valeri Souchkov, Sergei Ikovenko, Zion Bar-El, Nikolai Khomenko, Denis Cavallucci, Oleg Feygenson, Darrell Mann, Karen Gadd, Pavel Livotov, Christian Thurnes, Oliver Mayer
- Typische Verhörer: "Lieferthoff", "Livotoff", "Liwotow" → Pavel Livotov; "Thunis", "Thurnis", "Turnes" → Christian Thurnes
- Firmen-/Methodennamen: "Trismastery Hub"-Varianten → TRIZ Mastery Hub; "Ideasio/Ideation Workbench" → Ideation Workbench; "Kepner Trigo" etc. → Kepner-Tregoe

**Standardfall ist die leichte Korrektur oben.** Nur wenn das Transkript es klar erfordert, zusätzlich:
- **Sprecher-Rekonstruktion aus dem Kontext**, falls die Transkriptionssoftware die Sprecher vertauscht oder innerhalb der Blöcke zusammengemischt hat. Dann anhand des Inhalts zuordnen (wer ist Robert = TRIZ-Experte/Gastgeber, wer der Interessent) und zusammengefallene Redebeiträge trennen. Bei Audio-Input aus Schritt 0 ist das kein Sonderfall, sondern immer fällig, weil Whisper keine Sprecher kennzeichnet: Der Text kommt als durchlaufender Block, und die Redebeiträge müssen inhaltlich aufgeteilt und zugeordnet werden. Anhaltspunkte: Robert stellt die Fragen zu Bedarf und Zielsetzung, nennt Preise, Kursaufbau und Termine; der Interessent beschreibt Firma, Aufgabe und Randbedingungen. Kurze Einwürfe ("Mhm", "Ja, genau") gehören meist dem jeweils Zuhörenden. Wurde ausnahmsweise WhisperX benutzt, stattdessen dessen `SPEAKER_0x`-Labels übersetzen und nur die kurzen Einwürfe gegenlesen.
- **Fremdsprachige oder irrelevante Teile am Ende entfernen** (z. B. ein angehängtes internes Gespräch mit der Assistentin), wenn Robert das anmerkt oder es offensichtlich nicht zum Gespräch gehört.

Diese beiden Zusatzschritte sind die Ausnahme, nicht die Regel. Halte das bereinigte Transkript für Schritt 2 bereit.

## Schritt 2 — Roh- und bereinigte Datei speichern

**Zielordner:** `/Users/robert/Documents/TCG-Ordner/_Adunka/Vorgespraeche/`

**Dateinamen:** `yymmdd_Vorgespraech_VornameNachname_raw.txt` und `_corrected.txt`
- Datum: aus dem Originaldateinamen/Kontext, sonst heute (`yymmdd`)
- Name: der Interessent (der Nicht-Robert-Sprecher). Voller Name in CamelCase ohne Leerzeichen, z. B. `ChrisGray`

Speichere:
1. Eine Kopie des **Original-Roh-Transkripts** als `..._raw.txt`
2. Das **bereinigte Transkript** als `..._corrected.txt`

## Schritt 3 — Obsidian-Notiz anlegen

**Vorlage:** `/Users/robert/Library/Mobile Documents/iCloud~md~obsidian/Documents/Arbeit-ObsidianVault/Vorgespräche/_Vorlage Vorgespräch.md`

**Zieldatei:** `/Users/robert/Library/Mobile Documents/iCloud~md~obsidian/Documents/Arbeit-ObsidianVault/Vorgespräche/<Vorname Nachname>.md`

Lies die Vorlage, lege eine neue Datei nach dem Interessenten an und fülle alle Abschnitte **auf Deutsch** basierend auf dem bereinigten Transkript.

**YAML-Frontmatter (eigene Felder, nicht die von member-interview):**
- `Name:` — voller Name
- `Datum:` — Gesprächsdatum `YYYY-MM-DD` (wenn eine Uhrzeit bekannt ist, gehört sie in "Anlass / Kontext", nicht ins Frontmatter)
- `Sprache:` — DE oder EN
- `Art:` — Schulung / Workshop / Projekt / Kurs / Membership / Sonstiges (ggf. kombiniert, plus "(Interessent)")
- `Firma:` — Arbeitgeber, oder "Nicht genannt"
- `Rolle:` — Funktion/Position
- `Land:` — Standort
- `Status:` — offen / Angebot / gewonnen / verloren

**Abschnitte (auf Deutsch, nach dem tatsächlich Gesagten):**
- `## Anlass / Kontext` — wie kam der Kontakt zustande, worum ging es (Datum + ggf. Uhrzeit, Verweis auf die Transkript-Dateien im TCG-Ordner)
- `## Hintergrund Gesprächspartner` — beruflicher Hintergrund
- `## Bedarf / Ziel` — was sucht die Person, welches Problem
- `## Empfehlung / Angebot` — was wurde empfohlen/angeboten
- `## Offene Punkte` — Fragen, Einwände, Unklarheiten
- `## Follow-up / Nächster Schritt` — konkreter nächster Schritt, Wiedervorlage
- `## Notizen` — freie Notizen, Zitate, Eindrücke

Kurz aber vollständig schreiben. Wenn etwas nicht besprochen wurde, einen kurzen neutralen Platzhalter setzen statt den Abschnitt leer zu lassen.

**Kein Circle-Schritt.** Interessenten sind noch keine Mitglieder, es wird nichts in Circle eingetragen. Falls der Interessent später beitritt, kann er ins member-interview-System übernommen und hier der Status auf "gewonnen" gesetzt werden.

## Zusammenfassung

Nach den Schritten kurz in einem Absatz berichten:
- Gespeicherte Dateien (mit Dateinamen)
- Angelegte Obsidian-Notiz (Dateiname)
- Aufgetretene Besonderheiten (z. B. Audio transkribiert und mit welchem Modell, Sprecher-Rekonstruktion nötig, Teile entfernt)
