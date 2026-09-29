---
name: "event-grafiken-canva"
description: "Erstellt die drei Canva-Eventgrafiken (OpenGraph, PostThumb, EventCover) für einen Vortrag und lädt sie herunter; optional legt es dazu ein Special Event im Circle-Space Live Sessions an und liefert das LinkedIn-Material (16:9-Bild, Event-Texte, Post, Contentmaschine-Eintrag)."
---

# Event-Grafiken in Canva erstellen (optional mit Circle-Event)

Für jeden Event entstehen drei Canva-Designs im festen Layout: links Foto (Kopf) des Vortragenden, rechts blaue Fläche (#5b91c3) mit weißem, abgerundetem Kasten (80 % Deckkraft), darin fett „Vorname Nachname:“ und darunter der Vortragstitel in normaler Schrift. Am Ende liegen IMMER alle drei PNGs in exakter Größe im Download-Ordner des Nutzers.

Optional (nur wenn der Nutzer es wünscht): Anlegen des passenden Special Events in Circle (Teil B).

Immer wenn ein Circle-Event angelegt wird (Teil B), gehört Teil C dazu: LinkedIn-Material zum Kopieren (16:9-Bild, Event-Texte, kurzer Post) und nach dem Posten der Eintrag in der Contentmaschine.

## Benötigte Eingaben

1. Name des Vortragenden (mit Titel wie „Dr.“, falls vorhanden)
2. Vortragstitel (fehlt er, aus der Beschreibung 2 Vorschläge ableiten und wählen lassen)
3. Kurztitel für den Dateinamen (CamelCase, ohne Leerzeichen, z. B. „SolutionVerification“). Fehlt er, einen kurzen Vorschlag machen.
4. Foto: als Anhang im Chat ODER ein Bild, das schon im Canva-Konto liegt. Ist kein Foto angegeben, zuerst mit search-designs nach früheren Grafiken des Vortragenden suchen (Name in CamelCase bzw. Vorname) und die mediaId des Fotos aus read-design (open_transaction, danach cancel) wiederverwenden. Bekannt: Robert Adunka = MAGfe4KEO0s (quadratisch), Simon Litvin = MAGkK9LqxMY, Olga Bogatyreva = MAGuD96fNRs, Anatoly Agulyansky = MAHWAbIU8s8.
5. Nur für Teil B: Datum (muss ein Montag sein) und Beschreibungstext des Vortrags.

Fehlt etwas und lässt es sich nicht finden, einmal gebündelt nachfragen. Wenn nicht klar ist, ob ein Circle-Event gewünscht ist, in derselben Frage mitfragen.

# Teil A: Grafiken

## Vorlagen (Canva)

Vorlage ist das Set „SimonLitvin-SolutionVerification-…“:

| Format | Design-ID | Größe | Bild-Element | Bildrahmen | Textkasten |
|---|---|---|---|---|---|
| Event-OpenGraph | DAHVRme2JGA | 1200×630 | PB8NrRkjSv5BVJLx-LBCythNrvshN2nTM | 594.5×684.6 | PB8NrRkjSv5BVJLx-LBNDMxD84JL1VDk5 |
| PostThumb_600x300 | DAHVRsYT1kw | 600×300 | PBKJ4DfQHlBWlSPW-LBsZKjQBRg0XfPhS | 270×300 | PBKJ4DfQHlBWlSPW-LBXH6LjylKlZ60mq |
| EventCover_1024x366 | DAHVRkd0bCM | 1024×366 | PBHRRHFLMhnSNP3J-LBHkRSP7DzYL3Vd3 | 369.8×425.9 | PBHRRHFLMhnSNP3J-LBdk8SDgQm56d7yL |

Bei copy-design blieben die locator_ids bisher gleich; trotzdem kurz per read-design der Kopie prüfen, nie selbst zusammenbauen. Findet man die Vorlagen nicht mehr per ID, mit search-designs nach „SimonLitvin-SolutionVerification“ suchen.

## Dateinamen

Muster (Name in CamelCase ohne Leerzeichen und ohne „Dr.“, Umlaute ausschreiben: ä→ae usw.):

- `<NameCamelCase>_<Kurztitel>-Event-OpenGraph`
- `<NameCamelCase>_<Kurztitel>-PostThumb_600x300`
- `<NameCamelCase>_<Kurztitel>-EventCover_1024x366`

PNGs heißen genauso, mit Endung `.png`. Beispiel: `OlgaBogatyreva_BioTRIZ-EventCover_1024x366.png`

## Ablauf Grafiken

1. **Foto nach Canva bringen** (falls nicht schon vorhanden). curl zu www.canva.com ist in Cloud und device_bash blockiert (403), daher über Claude in Chrome:
   1. `create-upload-url` aufrufen (Einmal-URL).
   2. Tab auf https://www.canva.com/ öffnen.
   3. Per `javascript_tool` ein `<input type=file aria-label="claude upload input">` einfügen, dessen change-Handler `fetch(uploadUrl, {method:'POST', headers:{'Content-Type':'application/octet-stream'}, body: inp.files[0], credentials:'omit'})` ausführt und Status + Antworttext in `window.__up` speichert.
   4. Input per `find` suchen, dann `file_upload` mit dem Pfad des Anhangs (uploads-Ordner).
   5. Nach ~3 s `window.__up` auslesen → `{"mediaId":"MA…"}`. Tab schließen.
   Nie über öffentliche Hoster hochladen; keine Hilfsdateien im Download-Ordner des Nutzers ablegen.
2. **Drei Kopien anlegen**: `copy-design` für jede der drei Vorlagen-IDs.
3. **Jede Kopie bearbeiten**: `read-design` mit `open_transaction: true`, dann EIN `edit-design`-Aufruf mit diesen Operationen in genau dieser Reihenfolge:
   1. `update_title` → neuer Dateiname.
   2. **Titel (wichtig, sonst wird er fett!)**: Canva übernimmt beim Ersetzen die Formatierung des Zeichens VOR dem Treffer (hier der fette Zeilenumbruch nach dem Namen). Deshalb NICHT den ganzen alten Titel ersetzen, sondern:
      - `find_and_replace_text`: find `EN TRIZ Solution Verification Process` → replace `§<neuer Titel>` (davor steht das normal formatierte „G“, also wird der neue Titel normal).
      - `find_and_replace_text`: find `G§` → replace `` (leer), löscht die Hilfszeichen.
   3. `find_and_replace_text`: `Simon Litvin` → neuer Name (bleibt fett, Doppelpunkt bleibt).
   4. `update_fill` im Bild-Element (asset_type image, alt_text = Name).
   5. `crop_media` so, dass das Bild den Rahmen vollständig füllt (cover): Bildhöhe = Rahmenhöhe, Bildbreite = Rahmenhöhe × Seitenverhältnis des Fotos, top = 0, left so, dass das Gesicht etwa mittig sitzt (zwischen −(Bildbreite − Rahmenbreite) und 0). Für ein quadratisches Foto mit mittigem Gesicht: OpenGraph left -45, 684.63×684.63; PostThumb left -15, 300×300; EventCover left -28, 425.87×425.87. Ist das Foto hochformatiger als der Rahmen, stattdessen Bildbreite = Rahmenbreite und vertikal leicht nach oben ausrichten.
4. **Prüfen**: Thumbnail und Dokument aus der Antwort ansehen: Name fett, Titel fontWeight "normal", Text komplett im weißen Kasten, Foto ohne Lücken/Verzerrung, Kopf gut sichtbar. Bei langen Namen oder Titeln die Schriftgrößen mit `format_text` verkleinern (Name immer etwas größer als Titel). Geht etwas schief: Transaktion mit `cancel` verwerfen, neu öffnen, neu machen.
5. **Speichern**: Der Auftrag des Nutzers gilt als Freigabe (neue Kopien, Vorlagen bleiben unberührt). Wenn alle drei Prüfungen bestanden sind, jeweils `edit-design` mit `finalize: "commit"`.
6. **PNG-Export (immer, mit festen Maßen)**: `get-export-formats` einmal aufrufen, dann `export-design` IMMER mit expliziter Größe (sonst liefert Canva teils doppelte Auflösung):
   - OpenGraph: `{type: "png", width: 1200, height: 630}`
   - PostThumb: `{type: "png", width: 600, height: 300}`
   - EventCover: `{type: "png", width: 1024, height: 366}`
   Direkt danach herunterladen, die Links laufen schnell ab.
7. **Download in den Download-Ordner (immer)**:
   - curl auf export-download.canva.com ist in Cloud und device_bash blockiert (403), daher über Claude in Chrome.
   - Zugriff auf ~/Downloads per `device_request_folder_access` sicherstellen.
   - Neuen Tab, zu einer der Export-URLs navigieren (same-origin), dann per `javascript_tool` für jede URL `fetch` → blob → `<a download="<Dateiname>.png">` klicken (800 ms Pause), Status und Größe zurückgeben, nur bei `r.ok` laden.
   - Existiert eine Datei schon, unter `<Name>_neu.png` laden und per device_bash mit `mv -f` ersetzen.
   - Mit device_bash in `$HOME/mnt/Downloads` per `file` prüfen: echte PNGs mit exakt 1200×630, 600×300 und 1024×366. Fehlerhafte neu exportieren und laden. Tab schließen.
8. **Abschluss Teil A**: Kurz melden, dass die drei PNGs im Download-Ordner liegen, und die Links (edit_url) zu den Canva-Designs ausgeben.

# Teil B (optional): Special Event in Circle anlegen

Nur ausführen, wenn der Nutzer es wünscht. Vom Nutzer kommen: Datum, Vortragender, Beschreibungstext (plus die Grafiken aus Teil A).

## Feste Vorgaben

- Space: „Live Sessions“, https://triz-mastery-hub.circle.so/c/live-sessions, space_id **1800372**
- Tags (topics): **Special Event = 513625** und **Open to All = 513626** (immer beide)
- Tag: immer **Montag**. Ist das Datum kein Montag, nachfragen.
- Zeit: **15:00–16:00 Uhr Europe/Berlin** (3:00 PM – 4:00 PM). In UTC umrechnen: Winterzeit (CET) → `14:00:00Z`, Sommerzeit (CEST, letzter Sonntag im März bis letzter Sonntag im Oktober) → `13:00:00Z`. duration_in_seconds 3600.
- location_type: **live_room**
- host: „Robert Adunka“
- Benachrichtigungen: send_email_confirmation, send_email_reminder, send_in_app_notification_confirmation, send_in_app_notification_reminder = true; rsvp_disabled false; hide_attendees false; hide_location_from_non_attendees false.
- Status: zunächst **draft**, veröffentlichen nur auf ausdrückliche Anweisung.
- Referenz-Events im Space (Stil): „Oleg Feygenson: AI Finds More Resources. TRIZ Finds the Right One.“, „Min-Gyu Lee: Introduction to CECA+ and FA+“.

## Inhalte erzeugen

- **Event-Name**: `<Name des Vortragenden>: <Vortragstitel>` (wie in den Referenz-Events).
- **Body (HTML)**: Circle akzeptiert HTML im Feld `body` und rendert es. Aufbau:
  - `<h2>` Vortragstitel
  - Beschreibung in `<p>`-Absätzen, inhaltlich unverändert (nur Ich-Form ggf. in 3. Person), wichtige Begriffe `<strong>`
  - Enthält die Beschreibung Aufzählungen: `<h3>` + `<ul><li>`
  - Falls eine Bio mitgeliefert wird: `<h3>About the speaker</h3>` mit Name `<strong>`
  - `&` als `&amp;` schreiben
- **Meta title**: max. ca. 60 Zeichen, prägnant, ggf. „| TRIZ Mastery Hub“
- **Meta description**: 140–160 Zeichen, nennt Vortragenden und Nutzen
- **OpenGraph title**: neugierig machend, max. ca. 70 Zeichen
- **OpenGraph description**: 1–2 Sätze, max. ca. 200 Zeichen, gern mit Datum
- Sprache der Texte = Sprache der Beschreibung (meist Englisch).

## Ablauf Teil B

1. Name, Termin (lokal + UTC), Body-Text, Meta- und OpenGraph-Texte dem Nutzer vorlegen und **Freigabe abwarten**. Änderungen einarbeiten.
2. **Bilder zu Circle hochladen** (S3 ist aus Cloud und device_bash blockiert, daher über Chrome):
   1. Die PNGs aus dem Download-Ordner per `device_stage_files` in die Cloud holen; Größe und Base64-MD5 berechnen (`openssl md5 -binary <datei> | base64`).
   2. Tab auf https://triz-mastery-hub.circle.so/c/live-sessions öffnen, per `javascript_tool` ein `<input type=file multiple aria-label="claude upload input">` einfügen, das `window.__files = Array.from(inp.files)` setzt; per `find` + `file_upload` die gestagten Dateien (EventCover + Event-OpenGraph) hineinladen.
   3. ERST DANN je Datei `create_direct_upload` (key "", filename, content_type image/png, byte_size, checksum) – die URLs sind nur 5 Minuten gültig.
   4. Sofort per `javascript_tool` je Datei `fetch(direct_upload.url, {method:'PUT', body: file, credentials:'omit', headers:{'Content-Type':'image/png','Content-MD5': md5, 'Content-Disposition': <aus der Antwort>}})` → Status 200 prüfen.
   5. Die `signed_id`s verwenden: EventCover → `cover_image`, Event-OpenGraph → `meta_tag_attributes.opengraph_image`.
3. **Event anlegen** mit `create_event` (space_id 1800372 oben UND im event-Objekt), status draft, event_type single, topics [513625, 513626], event_setting_attributes wie oben, meta_tag_attributes, cover_image.
4. **Prüfen**: Antwort kontrollieren (starts_at, location_type live_room, topics, cover_image_url, opengraph_image). Optional die Event-URL im Chrome-Tab ansehen (Formatierung, Cover). Tab schließen.
5. **Abschluss**: Link zum Event ausgeben, Termin in lokaler Zeit nennen, darauf hinweisen, dass es ein Entwurf ist.

# Teil C: LinkedIn-Event und Ankündigungspost (nach Teil B)

Robert legt das LinkedIn-Event und den Post **selbst von Hand** an. Nicht per Chrome ausfüllen (LinkedIn verschluckt beim Tippen Zeichen, und Robert macht es lieber selbst). Aufgabe ist nur, alles fertig zum Kopieren bereitzustellen. Erst starten, wenn das Circle-Event veröffentlicht ist, denn der Link im LinkedIn-Event zeigt darauf.

## 1. Titelbild 16:9

LinkedIn empfiehlt für Event-Titelbilder 16:9 (mind. 480 px breit). Das EventCover (1024×366) passt nicht, das OpenGraph-Bild wird zugeschnitten: links 80 px vom Foto abschneiden, blauer Kasten und weißer Textkasten bleiben vollständig.

```bash
cd ~/Downloads && cp <Name>_<Kurztitel>-Event-OpenGraph.png <Name>_<Kurztitel>-LinkedIn_16x9.png && sips -c 630 1120 --cropOffset 0 80 <Name>_<Kurztitel>-LinkedIn_16x9.png
```

Ergebnis 1120×630 per `file` prüfen und das Bild ansehen (Gesicht gut sichtbar, Kasten komplett). Ein Canva-Export in doppelter Größe als Basis wurde bisher abgelehnt, daher direkt das 1200×630-PNG nehmen.

## 2. Event-Beschreibung als .txt

Datei `~/Downloads/<Name>_<Kurztitel>-LinkedIn-Beschreibung.txt` schreiben und per SendUserFile schicken:
- Inhalt wie der Circle-Body, aber als reiner Text: nur ASCII, keine Formatierung, Aufzählungen mit "- " am Zeilenanfang, keine Überschriften.
- Letzter Absatz: "Live session of the TRIZ Mastery Hub, open to all. Register via the link to join."
- Max. 5.000 Zeichen.

Dazu im Chat die übrigen Formularwerte als Liste (keine Tabelle):
- Art: Online, Event-Format "Externer Link"
- Name des Events (max. 75 Zeichen) = Circle-Event-Name
- Zeitzone Berlin, Datum, 15:00 bis 16:00
- Externer Link = Circle-Event-URL
- Titelbild = die 16:9-Datei
- Referent:innen: nicht vorschlagen einzutragen, ohne zu fragen (LinkedIn schickt dann eine Einladung)

## 3. Kurzer LinkedIn-Post

Regeln aus dem Memory "LinkedIn Post Formatting Rules" gelten: Englisch, nur ASCII, keine Formatierung, keinerlei Gedankenstriche, **1 bis 3 Sätze**: Hook plus Termin plus Zugangsinfo ("Free for everyone with a free login on TRIZ Mastery Hub."). Keine Agenda, keine Aufzählung. Im Codeblock ausgeben, damit er sich kopieren lässt.

Beispiel (Simon Litvin, Startups):
```
90% of startups fail, most of them because they misread the market or cannot defend what makes them different. On October 26, Simon Litvin shows how GEN TRIZ tools helped build successful startups, with real case studies.
Free for everyone with a free login on TRIZ Mastery Hub.
```

## 4. Contentmaschine (nachdem Robert den Post-Link schickt)

Datei in `/Users/robert/Library/Mobile Documents/iCloud~md~obsidian/Documents/Arbeit-ObsidianVault/Contentmaschine/` anlegen, Name `YYMMDD <Name> <Kurztitel> Ankuendigung.md` (Datum der Veröffentlichung). Inhalt nur Frontmatter plus veröffentlichter Posttext:

```
---
Platform: LinkedIn
Veröffentlicht: YYYY-MM-DD
Link: <Post-Link ohne utm-Parameter>
---
<Posttext>
```

Den Index `__Contentmaschine.md` nicht anfassen. Hat Robert den Text vor dem Posten geändert, die veröffentlichte Fassung erfragen.

## Hinweise

- Nie die Canva-Vorlagen selbst verändern, immer auf Kopien arbeiten.
- Offene Transaktionen, die nicht gespeichert werden, mit `finalize: "cancel"` schließen.
- Kein neues Textfeld per add_text anlegen (falsche Schrift).
- Antworten an den Nutzer auf Deutsch.