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

Transkription läuft lokal mit Whisper, es wird nichts hochgeladen. Vorher prüfen, ob das CLI da ist (`which whisper`); fehlt es, Robert darauf hinweisen statt einen Ersatzweg zu improvisieren.

```bash
whisper "<Audiodatei>" --model medium --language de --output_format txt --output_dir "/Users/robert/Documents/TCG-Ordner/_Adunka/Vorgespraeche/"
```

- **Modell:** `medium` ist der Standard — guter Kompromiss aus Qualität und Laufzeit. Nur auf `large-v3` wechseln, wenn Robert es verlangt oder das Ergebnis erkennbar unbrauchbar ist.
- **Sprache:** `--language de` bei deutschen Gesprächen, `--language en` bei englischen. Wenn unklar, den Parameter weglassen und Whisper erkennen lassen.
- **Laufzeit:** auf CPU grob die Länge der Aufnahme bis zum Doppelten davon. Immer im Hintergrund starten und nicht blockierend warten. Vorab mit `ffprobe` die Dauer ermitteln und Robert eine Zeitschätzung nennen.
- **Ergebnis:** Whisper legt `<Basisname>.txt` im Zielordner ab. Diese Datei ist ab hier das **Roh-Transkript** und geht so in Schritt 1 und 2. In Schritt 2 wird sie auf das Namensschema `..._raw.txt` gebracht; die von Whisper erzeugte Datei danach entfernen, damit im Ordner keine Dublette liegt.
- **Datum:** Audiodateinamen enthalten meist kein Datum. Dann das Änderungsdatum der Audiodatei als Gesprächsdatum verwenden, sonst heute.

Whisper-Transkripte haben typischerweise keine Sprecherkennung — der Text läuft als ein Block durch. Die Zuordnung der Redebeiträge zu Robert und zum Interessenten passiert dann in Schritt 1 über die Sprecher-Rekonstruktion aus dem Kontext.

## Schritt 1 — Transkript bereinigen

Lies das Roh-Transkript. Wende **nur die minimal nötigen Korrekturen** für Speech-to-Text-Fehler an. Nicht paraphrasieren, keine Grammatik glätten, keine Füllwörter entfernen, Stil nicht ändern.

**TRIZ-Begriffe** (nur wenn der Kontext eindeutig die Methodik meint):
- "Tris", "trees", "Chris", "Pris", "twist" → TRIZ
- "Arize", "arise", "a rise", "a riz" → ARIZ

**Personennamen** (phonetische Varianten korrigieren):
- Varianten → Genrikh Altshuller, Boris Zlotin, Alla Zusman, Valeri Souchkov, Sergei Ikovenko, Zion Bar-El, Nikolai Khomenko, Denis Cavallucci, Oleg Feygenson, Darrell Mann, Karen Gadd
- Firmen-/Methodennamen: "Trismastery Hub"-Varianten → TRIZ Mastery Hub; "Ideasio/Ideation Workbench" → Ideation Workbench; "Kepner Trigo" etc. → Kepner-Tregoe

**Standardfall ist die leichte Korrektur oben.** Nur wenn das Transkript es klar erfordert, zusätzlich:
- **Sprecher-Rekonstruktion aus dem Kontext**, falls die Transkriptionssoftware die Sprecher vertauscht oder innerhalb der Blöcke zusammengemischt hat. Dann anhand des Inhalts zuordnen (wer ist Robert = TRIZ-Experte/Gastgeber, wer der Interessent) und zusammengefallene Redebeiträge trennen.
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
