---
name: vorgespraech
description: Verarbeitet Transkripte von Vorgesprächen (Discovery-/Verkaufsgespräche) zu Schulungen, Workshops, Projekten oder Mitgliedschaften mit Interessenten. Triggert auf "/vorgespraech", "Vorgespräch verarbeiten", "Discovery Call", "Erstgespräch", "Akquisegespräch", oder wenn Robert ein Transkript eines Vor-/Verkaufsgesprächs mit einem Interessenten einfügt. NICHT für bestehende TRIZ-Mastery-Hub-Mitglieder verwenden — dafür ist der Skill member-interview zuständig.
version: 1.0.0
---

# Vorgespräch Processor

Dieser Skill verarbeitet das Transkript eines Vorgesprächs (Discovery- oder Verkaufsgespräch) mit einem Interessenten zu Schulungen, Workshops, Projekten oder einer Mitgliedschaft. Transkripte können deutsch oder englisch sein. Führe die drei Schritte ohne Rückfrage nacheinander aus.

**Abgrenzung:** Dieser Skill ist für Interessenten VOR einem Abschluss/Beitritt. Bestehende TRIZ-Mastery-Hub-Mitglieder werden mit dem Skill `member-interview` verarbeitet (der hat einen zusätzlichen Circle-Schritt). Wichtige Unterschiede zu member-interview: (1) kein Circle-Schritt, (2) eigene Frontmatter-Felder, (3) eigener Ablageort.

## Input

Zwei Formen:

**Dateipfad** — Robert gibt einen Pfad zum Transkript an:
```
/vorgespraech /path/to/transcript.txt
```

**Inline** — Robert fügt das Transkript direkt in den Chat ein (mit oder ohne `/vorgespraech`). Wenn im Text oder Kontext kein Datum steht, heutiges Datum für den Dateinamen verwenden.

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

Nach den drei Schritten kurz in einem Absatz berichten:
- Gespeicherte Dateien (mit Dateinamen)
- Angelegte Obsidian-Notiz (Dateiname)
- Aufgetretene Besonderheiten (z. B. Sprecher-Rekonstruktion nötig, Teile entfernt)
