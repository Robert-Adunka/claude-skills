---
name: special-event-nachbearbeitung
description: Führt Robert Schritt für Schritt durch die Nachbearbeitung eines Special Events auf dem TRIZ Mastery Hub, von der Aufnahme bis zu den Punkten für Presenter und Teilnehmer. Sagt bei jedem Schritt, was dran ist und wer ihn macht, erledigt die Schritte selbst, die per Terminal oder Circle MCP gehen (Komprimieren, Transkript laden und korrigieren, Teaser, Tags), und hakt den Stand als ToDo-Block in Roberts Daily Note in Obsidian ab. Triggert auf "/special-event-nachbearbeitung", "Event nachbearbeiten", "Nachbearbeitung Special Event", "Special Event Nachbereitung", "weiter mit der Nachbearbeitung", oder wenn Robert nach einem Live-Vortrag fragt, was jetzt zu tun ist. NICHT für die Vorbereitung eines Events (dafür event-grafiken-canva) und nicht für TRIZtalks-Kapitel.
version: 1.0.0
---

# Special Event Nachbearbeitung

Du gehst mit Robert die Nachbearbeitung eines Special Events gemeinsam durch. Bei jedem Schritt sagst du klar, **was jetzt dran ist** und **wer ihn macht**: Robert (Handarbeit in FCP oder im Circle-Backend) oder du (Terminal, Circle MCP, Skills). Du gehst erst weiter, wenn der Schritt erledigt ist.

Die Checkliste stammt aus Roberts Obsidian-Notiz `TRIZ-Mastery-Hub/TRIZ Mastery Hub.md` (Abschnitt "Special Event Nachbearbeitung") im Vault `Arbeit-ObsidianVault`. Ändert Robert dort die Liste, gilt seine Notiz.

## Feste Daten

- Event-Ordner: `/Users/robert/Documents/TCG-Ordner/Kurse-<Jahr>/_Circle/<YYMMDD>_Circle_<Presenter>_<Thema>/` (Schreibweise schwankt, z.B. `261005_CircleOlga-BioTRIZ`. Nach Datum suchen.)
- Vault: `/Users/robert/Library/Mobile Documents/iCloud~md~obsidian/Documents/Arbeit-ObsidianVault/`
- Circle-Space für Ankündigungen: Live Sessions, ID 1800372 (event)
- Circle-Space für Aufzeichnungen: Video Library, ID 1889732 (früher "Special Events Recordings")
- Tag "Presentator Award": ID 297090
- Tag "Attended event": ID 262172
- Bulk Action "Reward Presenter": vergibt 50 Punkte, schickt eine Dank-DM mit Link zur Video Library und entfernt den Tag 297090 wieder
- Bulk Action "Reward Event Attendence": vergibt 5 Punkte, schickt eine DM und entfernt den Tag 262172 wieder

Der Circle MCP kann Bulk Actions **nicht** auslösen. Die startet Robert immer selbst im Backend. Biete nicht an, sie über den Browser zu klicken.

## Start: Event finden und Stand laden

1. Event bestimmen: Hat Robert kein Event genannt, nimm das letzte vergangene Event im Space Live Sessions (`list_events`, nach Datum) und lass es dir bestätigen.
2. Event-Ordner suchen (Datum als Präfix).
3. Stand in den Daily Notes suchen (siehe unten). Gibt es noch keinen Block für dieses Event, leg ihn in der heutigen Daily Note an. Hak Schritte ab, die sich am Ordnerinhalt schon als erledigt zeigen (z.B. liegt `*-small.mp4` schon da).
4. Kurzer Überblick an Robert: Event, Presenter, Ordner, welche Schritte erledigt sind, welcher als nächstes dran ist.

## Stand in der Daily Note

Der Fortschritt steht als ToDo-Block in Roberts Daily Note, weil er ihn dort im Blick hat. Keine eigene Statusdatei anlegen.

- **Daily Notes finden:** Dateien `YYYY-MM-DD.md` im Vault, die aktuellen liegen im Vault-Root, ältere unter `Daily/`. Daily Notes legt ein Cron Job an. **Nie selbst eine Daily Note erstellen.** Fehlt die heutige, Robert Bescheid geben und warten.
- **Block anlegen:** direkt unter `# ToDos:` als ersten Eintrag einfügen. Die Unterpunkte wörtlich aus dem Abschnitt "Special Event Nachbearbeitung" der Notiz `TRIZ-Mastery-Hub/TRIZ Mastery Hub.md` übernehmen, inklusive der `[[...]]`-Links, eingerückt mit einem Tab. Format wie bei früheren Events (z.B. `Daily/2026-09-21.md`):

```
- [ ] Nachbearbeitung <Presenter>:
	- [ ] Download file
	- [ ] FCP-Edit: Bauchbinde anfang, Abspann, TRIZ Mastery Hub Logo
	- [ ] ...
```

- **Fortsetzen:** Den Block in der neuesten Daily Note suchen, die `Nachbearbeitung <Presenter>` enthält, und dort weiter abhaken.
- **Abhaken:** Nach jedem erledigten Schritt `- [ ]` auf `- [x]` setzen. Ein Ergebnis höchstens als kurzer Zusatz hinten an die Zeile (z.B. Post-URL, "23 getaggt"), sonst nichts in die Daily schreiben. Sind alle Unterpunkte erledigt, auch die Elternzeile abhaken.
- Übersprungene Schritte: `- [-]` mit kurzem Grund.

Die Schrittnummern unten entsprechen der Reihenfolge in der Obsidian-Liste.

## Die Schritte

### 1. Download file (Robert)
Robert lädt die Aufnahme aus dem Circle-Room in den Event-Ordner. Der Dateiname sieht aus wie `<event-slug>-triz-mastery-hub-<nummer>.mp4`. Dazu gleich die Teilnehmerliste aus dem Room mitnehmen (`room_..._participants_list.csv`), die wird in Schritt 15 gebraucht.

### 2. FCP-Edit (Robert)
Bauchbinde am Anfang, Abspann, TRIZ-Mastery-Hub-Logo. Export in den Event-Ordner als `<PresenterOhneLeerzeichen>_<ThemaCamelCase>.mp4`, z.B. `SimonLitvin_SolutionVerification.mp4`. Schlag Robert den Namen vor. Kurzbefehle stehen in der Vault-Notiz `Programme/Final Cut Pro.md` (Basic Lower Third: ctrl+Umschalt+T, Logo: Cmd+I, Transform im Inspector).

### 3. Video komprimieren (du)
```
ffmpeg -i <Name>.mp4 -c:v libx264 -crf 23 -preset slow -c:a aac -b:a 128k -movflags +faststart <Name>-small.mp4
```
Ausgabe immer `<Name>-small.mp4` im selben Ordner. Im Hintergrund laufen lassen, das dauert. Danach Größe vorher und nachher melden. Die Originaldatei nicht löschen.

### 4. Post in Video Library anlegen (du legst den Entwurf an, Robert prüft)
Hol die Ankündigung aus dem Event in Live Sessions (`list_events`/`get_event`). Baue daraus einen Entwurf im Stil der bisherigen Posts:

- Titel: `<Presenter>: <Vortragstitel>`
- Body: Beschreibungstext aus der Ankündigung, danach Überschrift H3 `Presentation slides:`
- Status: **draft**
- Immer explizit mitgeben: `hide_meta_info: true` (Robert soll nicht als Autor angezeigt werden), `is_comments_enabled: true` und `is_liking_enabled: true` (Members sollen Notizen und Reaktionen hinterlassen können). Ohne diese Angaben stellt Circle Kommentare und Likes aus und zeigt die Metainfo.

Vorher Titel und Text zeigen und das OK abwarten. Erst dann per `create_post` (space_id 1889732, `tiptap_body`) anlegen. Post-URL in die Statusdatei schreiben.

### 5. bis 7. Cover, Video, Folien hochladen (Robert)
Im Circle-Editor:
- Cover: `*PostThumb_600x300.png` bzw. `*Post_600x300.png` aus dem Event-Ordner als Card-Thumbnail (meist schon von der Event-Vorbereitung vorhanden)
- Video: `<Name>-small.mp4` ganz oben im Post
- Folien: PDF unter "Presentation slides:". Liegt nur eine `.pptx` vor, biete an, sie per `soffice --headless --convert-to pdf` umzuwandeln.

Danach veröffentlicht Robert den Post. Warte, bis Circle das Video transkodiert hat, das dauert einige Minuten.

### 8. Transkript herunterladen (du)
Über `get_post` den Recording-Post laden, im `inline_attachments`-Eintrag des Videos steht `webvtt_file_url`. Die Datei per `curl -sL` als `captions.vtt` in den Event-Ordner laden. Fehlt die URL, ist die Transkription noch nicht fertig: Robert bitten, kurz zu warten. Ersatzweise lädt Robert sie im Editor herunter ("Make downloadable" eingeschaltet).

### 9. Transkript korrigieren (du, Robert gibt frei)
Grundlage ist der Prompt in der Vault-Notiz `Prompts/Transkript korrigieren, z.B. trees -> TRIZ.md`, Abschnitt **"Version Universell"**. Lies ihn bei jedem Lauf frisch und befolge ihn. Folien-PDF des Vortrags als Referenz nutzen.

1. Liefere "Intended changes" (gruppiert) und "Ambiguous / not changed". Bei langen Transkripten die Fundstellen per `grep -n` zählen statt alles in den Chat zu kippen.
2. Warte auf Roberts Freigabe.
3. Wende nur die freigegebenen Änderungen an und schreibe `captions_corrected.vtt` in den Event-Ordner.
4. Prüfe: Datei beginnt mit `WEBVTT`, Zahl der Cues und alle Zeitstempel-Zeilen identisch mit `captions.vtt` (per `diff` auf die `-->`-Zeilen).

### 10. Korrigiertes Transkript hochladen (Robert)
Robert ersetzt im Circle-Editor das Transkript des Videos durch `captions_corrected.vtt`.

"Make downloadable" schaltet Robert nur für den Transkript-Download kurz ein und danach bewusst wieder aus, damit die Aufzeichnung nur im Hub zu sehen ist und Members an den Hub gebunden bleiben. Ein ausgeschalteter Download ist also gewollt, nicht als Problem melden.

### 11. Teaser erstellen (du)
Starte den Skill `youtube-teaser` mit `captions_corrected.vtt`, Ankündigungstext, Post-URL und Aufnahmedatum. Alles liegt schon vor, also keine Rückfragen nach diesen Daten. Der Skill wartet nach der Vorauswahl auf Roberts Wahl der Teaserstelle. Fallback, falls der Skill nicht verfügbar ist: Prompt aus der Vault-Notiz `Prompts/Teaser für Video erstellen.md`.

Den finalen Teaser **im Chat** als einen Codeblock zum Kopieren ausgeben, so wie `youtube-teaser` ihn liefert, damit Robert ihn direkt weiterverwenden kann. Zusätzlich als `teaser.md` in den Event-Ordner speichern.

### 12. Trello-Karte für Martin Helm (du schreibst, Robert legt an)
Die Karte bekommt **den kompletten Inhalt von `teaser.md`**, unverändert, nicht gekürzt und nicht umsortiert. Gib ihn noch einmal als einen Codeblock zum Kopieren aus, ohne Tabellen:
- Erste Zeile (Kartentitel): `Teaser: <Presenter>: <Titel>`
- Danach der vollständige Inhalt von `teaser.md`, inklusive YouTube-Titel, Beschreibung und Tags
- Am Ende: Dateiname der Schnittfassung und Pfad zum Event-Ordner

Keine eigene Auswahl oder Neuformulierung für die Karte bauen. Robert kopiert den Block 1:1 nach Trello.

### 13. Banner im Feed (Robert)
Hol per `list_events` das nächste Special Event in Live Sessions und nenne (auch eins am selben Tag, das nach dem nachbearbeiteten Event beginnt; nach Startzeit sortieren, nicht nur nach Datum) Titel, Datum und die passende `*EventCover_1024x366.png` bzw. `*Event-OpenGraph.png` aus dessen Event-Ordner. Das Banner selbst tauscht Robert im Backend.

Dazu immer den Bannertext als Codeblock zum Kopieren liefern, auf Englisch, Uhrzeit in deutscher Zeit (Europe/Berlin) mit "CET" dahinter, damit die Teilnehmer die Zeitzone kennen, genau in diesem Stil:
```
Special Event on October 5th at 3:00 PM CET with Carmen Lehner & Jens Träger: Think. Solve. Create.
```
Also `Special Event on <Monat> <Tag mit st/nd/rd/th> at <h:mm AM/PM> CET with <Presenter>: <Kurztitel>`. Kurztitel ist der Teil des Event-Titels vor einem Gedankenstrich oder Untertitel. Immer "CET" schreiben, auch in der Sommerzeit.

### 14. Punkte für Presenter (du taggst, Robert löst aus)
Presenter per `search_community_member` finden, Treffer bestätigen lassen, dann Tag "Presentator Award" (297090) setzen. Robert startet danach die Bulk Action "Reward Presenter", die den Tag selbst wieder entfernt. Nichts nachputzen. Die DM verweist auf die Aufzeichnung in der Video Library, deshalb diesen Schritt erst machen, wenn der Recording-Post veröffentlicht ist (Schritte 4 bis 7).

### 15. Punkte für Teilnehmer (du taggst, Robert löst aus)
Aus der Teilnehmer-CSV (`room_*_participants_list.csv`, Spalte `ID` ist die Community-Member-ID) allen den Tag "Attended event" (262172) geben, **außer Robert Adunka und dem Presenter**. Danach nur melden, wie viele getaggt wurden und wer bewusst ausgelassen wurde. Robert startet die Bulk Action "Reward Event Attendence", die den Tag selbst wieder entfernt. Nichts nachputzen.

## Abschluss

Sind alle Punkte abgehakt, kurze Zusammenfassung im Chat: Post-URL, Pfad zu `teaser.md`, Anzahl getaggter Teilnehmer.

## Regeln

- Schritt für Schritt. Nie mehrere Robert-Schritte auf einmal abfragen, außer sie gehören zusammen (5 bis 7).
- Alles, was in Circle öffentlich sichtbar wird oder Mitglieder betrifft (Post anlegen, Tags setzen), vorher zeigen und bestätigen lassen.
- Robert kann Schritte überspringen oder umstellen.
- Texte zum Kopieren in Codeblöcken, ohne Markdown-Tabellen.
- Alles, was im Hub oder auf YouTube erscheint, auf Englisch. Mit Robert auf Deutsch.
