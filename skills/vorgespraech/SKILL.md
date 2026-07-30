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

Transkription läuft lokal mit WhisperX, es wird nichts hochgeladen. WhisperX liegt in einer eigenen virtuellen Umgebung:

```bash
/Users/robert/.venvs/whisperx/bin/whisperx "<Audiodatei>" \
  --model large-v3-turbo --language de \
  --device cpu --compute_type int8 \
  --vad_method silero \
  --diarize --min_speakers 2 --max_speakers 2 \
  -f srt --output_dir "/Users/robert/Documents/TCG-Ordner/_Adunka/Vorgespraeche/"
```

- **Warum WhisperX statt `whisper`:** es liefert Sprechertrennung (Diarisierung über pyannote) und satzgenaue Zeitmarken durch Alignment. Das blanke `whisper`-CLI kann beides nicht.
- **`--vad_method silero` ist Pflicht, nicht Geschmackssache.** WhisperX transkribiert nur, was seine Sprachaktivitätserkennung als Sprache markiert. Die voreingestellte pyannote-VAD schneidet auf Roberts Aufnahmen ganze Passagen weg — bei einer geprüften 26-Minuten-Datei fehlten **22 Prozent des Textes** ersatzlos, darunter inhaltlich wichtige Stellen wie der Name der Geschäftseinheit des Interessenten. Das Absenken von `--vad_onset`/`--vad_offset` behebt es **nicht**; Silero schon. Ohne diesen Schalter ist das Transkript unvollständig, und zwar unauffällig: die Lücken sind im Fließtext nicht zu sehen.
- **Plausibilitätsprüfung nach dem Lauf:** grob 130 bis 140 Wörter pro Gesprächsminute sind normal. Deutlich weniger heißt, dass die VAD Text verschluckt hat. Im Zweifel gegen einen Lauf des blanken `whisper` gegenprüfen, das keine VAD-Vorfilterung macht.
- **Modell:** immer `large-v3-turbo`.
- **Sprecherzahl:** **Robert vor dem Lauf fragen, wie viele Personen auf der Aufnahme sprechen.** Er kann das nach dem Gespräch sagen, auch wenn es vorher nicht feststand. Regelfall sind zwei, es kommen aber Gespräche mit mehr Teilnehmern vor. Die genannte Zahl in `--min_speakers` und `--max_speakers` eintragen — eine feste Vorgabe verbessert das Ergebnis deutlich. Antwortet er nicht oder ist er unsicher, beide Parameter weglassen und pyannote schätzen lassen; dann aber im Ergebnis prüfen, ob die Zahl der gefundenen Sprecher plausibel ist. **Nicht** auf einen kurzen Ausschnitt anwenden, in dem eine Person kaum spricht: das Clustering teilt dann den dominanten Sprecher in zwei auf, statt den stillen zu finden.
- **Zugang:** die Diarisierung lädt `pyannote/speaker-diarization-community-1`, ein zugangsbeschränktes Modell. Roberts HuggingFace-Token liegt in `~/.cache/huggingface/`, WhisperX findet ihn selbst. Die Warnung `No --hf_token provided` im Log ist irreführend und kann ignoriert werden, solange der Lauf weiterläuft.
- **Sprache:** `--language de` bei deutschen Gesprächen, `--language en` bei englischen. Wenn unklar, den Parameter weglassen.
- **Laufzeit:** rund drei Viertel der Aufnahmedauer auf CPU (26 Minuten Audio ≈ 19 Minuten Rechenzeit). Immer im Hintergrund starten und nicht blockierend warten. Vorab mit `ffprobe` die Dauer ermitteln und Robert eine Zeitschätzung nennen. Die Ausgabe wird gepuffert, wenn sie nicht auf ein Terminal geht — eine leere Logdatei heißt nicht, dass der Lauf hängt. Fortschrittsbalken beim Filtern per `grep -viE "%\||MB/s"` unterdrücken, sonst ist das Log unlesbar.
- **Ergebnis:** WhisperX legt `<Basisname>.srt` im Zielordner ab, mit `[SPEAKER_00]`-Präfix je Segment. Diese Datei ist das **Roh-Transkript** und geht so in Schritt 1 und 2. In Schritt 2 wird sie auf `..._raw.txt` umbenannt (Inhalt unverändert, inklusive Zeitmarken und Sprecherlabels); die von WhisperX erzeugte Datei danach entfernen, damit im Ordner keine Dublette liegt.
- **Datum:** Audiodateinamen enthalten meist kein Datum. Dann das Änderungsdatum der Audiodatei als Gesprächsdatum verwenden, sonst heute.
- **Halluzinierte Schlusssätze:** Whisper erfindet auf Stille am Dateiende gern einen Satz ("Und diese Show wird schon toll sein.", "Vielen Dank fürs Zuschauen."). Steht am Ende ein Satz, der nicht zum Gespräch passt, ersatzlos streichen — das ist kein Hörfehler, sondern Erfindung.

### Was die Diarisierung leistet und was nicht

Sie liefert `SPEAKER_00` und `SPEAKER_01`, keine Namen. Welches Label Robert ist, einmal am Gesprächsanfang aus dem Inhalt bestimmen und dann durchziehen — die Zuordnung bleibt über die ganze Aufnahme stabil.

**Verlässlich** bei längeren Redebeiträgen. **Unzuverlässig bei kurzen Einwürfen** ("Ah, okay", "Ja, genau", "Mhm") — die landen oft beim falschen Sprecher. Bei einer geprüften 26-Minuten-Aufnahme waren rund 8 Prozent der Segmente falsch zugeordnet, fast ausschließlich solche Kurzeinwürfe. In Schritt 1 deshalb die kurzen Segmente inhaltlich gegenlesen und korrigieren; die langen Blöcke kann man übernehmen.

Aus den SRT-Zeitmarken den **Startzeitpunkt jedes Sprecherblocks als `mm:ss`** übernehmen, damit das bereinigte Transkript dasselbe Format hat wie die von Transkriptionsdiensten erzeugten Dateien:

```
Robert Adunka
00:19
So, wunderschönen guten Morgen.
```

**Falls WhisperX fehlt** (venv gelöscht o. ä.): Robert darauf hinweisen statt zu improvisieren. Notbehelf ist das blanke `whisper`-CLI ohne Sprechertrennung — das braucht dann aber `SSL_CERT_FILE`, weil die Modell-Downloads sonst an der Zertifikatsprüfung scheitern (`CERTIFICATE_VERIFY_FAILED`, selbstsigniertes Zertifikat in der Kette):

```bash
export SSL_CERT_FILE=$(/Library/Frameworks/Python.framework/Versions/3.14/bin/python3 -c "import certifi;print(certifi.where())")
```

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
- **Sprecher-Rekonstruktion aus dem Kontext**, falls die Transkriptionssoftware die Sprecher vertauscht oder innerhalb der Blöcke zusammengemischt hat. Dann anhand des Inhalts zuordnen (wer ist Robert = TRIZ-Experte/Gastgeber, wer der Interessent) und zusammengefallene Redebeiträge trennen. Bei Audio-Input aus Schritt 0 ist das kein Sonderfall, sondern immer fällig: die `SPEAKER_0x`-Labels in Namen übersetzen und die kurzen Einwürfe gegenlesen, weil die Diarisierung genau dort danebenliegt.
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
