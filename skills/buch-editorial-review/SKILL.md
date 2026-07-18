---
name: buch-editorial-review
description: Erstellt eine Editorial Checklist für ein Buchmanuskript vor einer weltweiten KDP-Veröffentlichung — ein Risiko-Review auf politische/ethnische/religiöse Trigger, "How-to-für-Schaden"-Lesarten und Bildrechte, mit fertigem Copy-paste-Ersatztext für den Autor. Triggert auf "/buch-editorial-review", "Editorial Checklist", "Checkliste für das Buch", "KDP Review", "Manuskript prüfen", "Buch veröffentlichungsreif machen", oder wenn Robert ein Buchmanuskript (.docx) einfügt und nach einer Risiko-/Lektoratsprüfung fragt. NICHT für normales Sprachlektorat, Layout oder Korrektorat verwenden.
version: 1.0.0
---

# Editorial Checklist für Buchmanuskripte (KDP weltweit)

Erstellt eine Checkliste, die ein Buchmanuskript für eine weltweite KDP-Veröffentlichung absichert. Das ist **kein Sprachlektorat**. Es ist ein Risiko-Review mit drei Zielen:

1. Politische, ethnische und religiöse Trigger entfernen, die internationale Leser vom Inhalt ablenken.
2. Passagen entschärfen, die sich als Anleitung zu Täuschung, Regelumgehung oder illegalem Handeln lesen lassen.
3. Bildrechte so dokumentieren, dass sie einer Prüfung standhalten.

Adressat ist **der Autor**, nicht der Verlag. Der Ton entscheidet über die Annahmequote: Der Autor behält seine Stimme, die Muss-Punkte sind trotzdem klar als solche markiert.

## Vorbedingungen klären

Bevor Du loslegst, brauchst Du:

- **Pfad zum Manuskript.**
- **Zielplattform** (Default: KDP weltweit). Andere Plattform → Strenge anpassen.
- **Existiert eine Photo-Sources-Tabelle** (separate xlsx) oder stehen die Bildnachweise im Manuskript?

**Zuerst im Buchordner nachsehen, ob schon eine Checkliste existiert.** Wenn ja: lies sie, und schlage einen Merge vor, statt eine zweite danebenzulegen. Mehrere ältere Listen sind oft komplementär — die eine findet, was die andere übersieht.

## Schritt 1 — Manuskript einlesen

`.docx` per Python auspacken (schneller und vollständiger als Konvertierungstools):

```python
import zipfile, re, html
z = zipfile.ZipFile(pfad)
x = z.read('word/document.xml').decode('utf8')
x = re.sub(r'</w:p>', '\n', x)
x = re.sub(r'</w:tc>', ' | ', x)
x = re.sub(r'<[^>]+>', '', x)
text = html.unescape(x)
```

Erst das Inhaltsverzeichnis und die Kapitelstruktur erfassen, dann die Bildnachweis-Liste am Ende.

## Schritt 2 — Risiko-Scan

Keyword-Scan über den Fließtext (ohne TOC und Bildnachweise), dann **jeden Treffer im Kontext lesen**. Der Scan findet Kandidaten; die Entscheidung fällt beim Lesen.

Kategorien, die erfahrungsgemäß tragen:

- **Zeitgenössische Politik** — lebende Politiker, laufende Konflikte, aktuelle Parteipolitik. Das Schärfste im Buch sitzt fast immer hier.
- **Historisch aufgeladene Figuren** — Hitler, Stalin & Co. Einzeln meist vertretbar; **auf Häufung prüfen**. Einmal ist Beleg, dreimal ist Masche.
- **Religion und Ethnie** — besonders, wenn eine Mehrheit als Täter und eine Minderheit als Opfer benannt wird.
- **Nationale Stereotype** — "backward country", "die Amerikaner sind…", "aus unerfindlichen Gründen wählte Land X…".
- **Militär und Waffen** — auf *Häufung* und *Verherrlichung* prüfen, nicht auf bloßes Vorkommen. Ein Museumsfoto eines Panzers ist kein Problem.
- **Opferzahlen im lockeren Ton** — Sarkasmus neben einer Totenzahl ist in jedem Markt ein Fehler.
- **How-to-Charakter** — operative Details, wie jemand eine Kontrolle umgangen hat.
- **Lebende Privatpersonen namentlich beschuldigt** — anderes Risiko als bei Personen der Zeitgeschichte.
- **Klima, Impfen, Gender** — Nebensätze mit Privatmeinung zu Streitthemen. Kosten Reviews, bringen nichts.
- **Stilwörter** — "fanatic", "bureaucrat", "idiotic", "cover your ass".

**Wichtig: Jedes Item gegen das Manuskript verifizieren, auch Items aus einer bestehenden Checkliste.** Frühere Listen enthalten erfahrungsgemäß Punkte, die auf nichts im Text zeigen. Ein Item, das der Autor nicht wiederfindet, kostet Dich die Glaubwürdigkeit für die ganze Liste.

## Schritt 3 — Items schreiben

Immer dasselbe Raster:

- **Problem.** Was ist riskant — und *warum*, konkret auf Markt und Leser bezogen. Nicht "ist sensibel", sondern was passiert.
- **Required edits.** Was muss weg, was bleibt.
- **Proposed new title.** Nur wenn der Titel selbst das Problem ist.
- **Proposed replacement text (copy-paste).** Fertiger Text. Kein "der Autor könnte hier umformulieren".
- **Figure.** Bildanweisung, falls betroffen.

Regeln, die sich bewährt haben:

- **Minimaleingriff, wo er reicht.** Manchmal ist es ein Satz plus ein Disclaimer, nicht die Neufassung des Abschnitts.
- **Umdeutung ins Defensive statt Löschung.** Der Trick, der ein Kapitel rettet: dieselbe Mechanik als Fehlermodus und Prävention formulieren, nicht als Taktik. "Disinformation" → "Layered communication and verification". "Operation Masquerade" → "Habituation in supervision". Der Inhalt bleibt, die Betriebsanleitung verschwindet.
- **Anti-Pattern-Label.** Wo der Autor an einem unethischen Kniff hängt: ausdrücklich als Fehlermodus des Prozesses kennzeichnen, plus Warnhinweis. Nicht streichen.
- **Option A/B bei Ermessensfragen** — mit klarer Empfehlung.
- **Anti-Stacking-Regel.** Nicht zwei Militär-/Politikbeispiele im selben Abschnitt. Wenn eines bleibt, wird das andere neutral.
- **Nicht überkorrigieren.** Empfehlungen, die dem Buch die Belege nehmen, ohne Risiko zu senken, gehören begründet abgelehnt — auch wenn sie aus einer früheren Liste stammen. Das gehört sichtbar in die Checkliste ("Reviewed — no action needed"), damit der Autor weiß, dass geprüft wurde.
- **Balance-Check.** Wenn das Buch überwiegend Negativbeispiele einer Nation bringt: als Beobachtung ansprechen, ohne Streichforderung. Der Fix ist Ergänzen, nicht Löschen.

## Schritt 4 — Front Matter

Immer eine Seite **"Scope, Ethics, and Compliance"** vorschlagen, direkt vor Kapitel 1 oder nach "From the Author". Sie rahmt das Buch als Denk-/Analysewerkzeug, sagt ausdrücklich, dass historische Beispiele der Prävention und Mustererkennung dienen, und benennt die Verantwortung des Lesers für Recht, Sicherheit und Compliance. Sie hilft auch der automatisierten KDP-Prüfung.

Textbaustein: siehe `references/frontmatter.md`.

## Schritt 5 — Bildrechte

- **Proof-Ordner** "Permissions proof": pro externem Bild ein Screenshot/PDF der Quellseite **mit den Lizenzbedingungen zum Downloadzeitpunkt**. Dateiname nach Figure-Nummer (`Fig084_source_license.png`). Eine Lizenzangabe in der Nachweisliste ist eine Behauptung; der Screenshot ist der Beleg.
- **Lizenzangaben gegen die Quelle prüfen.** Der häufigste echte Fund: eine CC-Lizenz, die auf einer kommerziellen Nachrichtenseite behauptet wird. Nachrichtenseiten vergeben keine CC-Lizenzen auf ihre Fotos. Immer auf Nachweise ohne Quell-URL prüfen.
- **CC BY-SA zählen.** ShareAlike heißt: Bearbeitungen müssen unter derselben Lizenz erscheinen. Im Buch ist oft unklar, was als Bearbeitung gilt (Zuschnitt, Farbkorrektur, Beschriftung, Einbettung ins Layout). Ab ca. 30 % ShareAlike-Anteil ein eigenes Item. Unverändert genutzt = nur Attribution, keine ShareAlike-Pflicht — das ist die entscheidende Unterscheidung. Nachweise, die selbst eine Änderung deklarieren ("a portion of the picture is used"), gezielt suchen.
- **Erkennbare Personen.** Pro Bild: Model Release für weltweit kommerzielle Nutzung, oder ersetzen. Bei Minderjährigen keine Ausnahme. Historische Aufnahmen und Museumsexponate sind in der Regel unproblematisch.
- **Druckfertige Attributionszeile** für jedes CC-BY-/CC-BY-SA-Bild liefern (Autor, Titel, Quelle, Lizenz, Link).
- **Final Acceptance vor Upload:** jedes Bild (1) in der Nachweisliste, (2) mit Proof-Datei, (3) mit korrekter gedruckter Attribution. Bilder mit erkennbaren Privatpersonen ersetzt, falls kein Release.

## Schritt 5b — Zählen, nicht schätzen

Lizenzverteilung, Figure-Anzahl und Lücken per Skript ermitteln, nicht per Gefühl. Zahlen im Dokument ("50 von 133 Figures tragen ShareAlike") sind das, was den Autor überzeugt und was die Arbeit planbar macht.

## Schritt 6 — Dokument bauen

Word-Datei über den `docx`-Skill (docx-js). Aufbau:

1. Titel, Manuskriptname, Opening Note ("proposed safe baseline — keep your voice").
2. "How to read": Prioritätenlegende (Critical / High / Medium / Low / Optional).
3. Part A: Front Matter.
4. Part B: Hotspot-Rewrites, nach Priorität sortiert.
5. Part C: Bilder, Rechte, Permissions.
6. Summary-Tabelle: #, Priority, Action, Section, Effort — mit Gesamtaufwand.

Die Aufwandsschätzung pro Item ist kein Beiwerk. Sie ist der Grund, warum der Autor anfängt.

**Fallstricke beim Bauen:**

- Apostrophe in JS-Strings (`KDP's`, `don't`) sprengen `'…'`-Strings. Typografische Apostrophe (`’`) verwenden — im Word-Dokument ohnehin schöner.
- `docx` ist ggf. nicht global installiert: im Arbeitsverzeichnis `npm install docx`.
- **Summary-Tabelle mit `PageBreak` auf eine eigene Seite setzen.** Sonst bricht sie mitten in einer Zeile um; LibreOffice wiederholt die Kopfzeile dann unzuverlässig, und es bleibt ein verwaistes Textfragment mit leeren Zellen zurück. Auf einer Seite ist die Übersicht ohnehin brauchbarer — man kann sie als ein Blatt ausdrucken. Zusätzlich `cantSplit: true` auf die Datenzeilen.

**Verifizieren — nicht optional:**

```bash
SOF=/Applications/LibreOffice.app/Contents/MacOS/soffice
PROFILE="-env:UserInstallation=file:///tmp/lo-headless"   # eigenes Profil, siehe unten
"$SOF" $PROFILE --headless --convert-to pdf out.docx
pdftoppm -jpeg -r 80 out.pdf page
# dann die page-*.jpg wirklich ansehen
```

**`-env:UserInstallation` ist Pflicht, nicht Kür.** Ohne das benutzt der Headless-Lauf Roberts persönliches LibreOffice-Profil und hinterlässt dort eine Lock-Datei (`~/Library/Application Support/LibreOffice/4/.lock`). Beim nächsten normalen Start von LibreOffice kommt dann „Either another instance of LibreOffice is accessing your personal settings or your personal settings are locked" — verwirrend und unnötig. Mit eigenem Profil bleibt sein Profil unberührt.

Falls doch mal ein verwaister Lock übrig ist: Zeitstempel in der Datei prüfen (`cat .lock`) und mit `lsof` sehen, ob jemand sie offen hält. Hält niemand sie offen, ist „Continue" im Dialog gefahrlos — die Warnung zielt auf zwei gleichzeitige Instanzen, die es dann nicht gibt.

Der erste `soffice`-Aufruf mit einem frischen Profil initialisiert nur dieses und liefert kein PDF — einfach nochmal aufrufen. Zusätzlich den Text aus der fertigen .docx extrahieren und auf Struktur, Mojibake und `undefined` prüfen. Beides tun: die Textextraktion findet Encoding-Fehler, das Rendern findet Layout-Fehler. Keines findet beides.

Dateiname: `JJMMTT_EditorialChecklist_<Buchtitel>_v<N>.docx`, abgelegt im Buchordner. PDF danebenlegen — der Autor liest eher das PDF.

## Referenz

Das Muster stammt aus `260203_EditorialChecklistMicroStandards.docx` (Buch "Microstandards of Thinking", Lev Pevzner, 2026) — die Checkliste, die beim Autor funktioniert hat und deren Änderungen vollständig in die Druckfassung eingegangen sind. Bei Unsicherheit über Ton oder Schnitttiefe dort nachsehen:
`/Users/robert/Documents/TCG-Ordner/_Adunka/Buchprojekte/2026_LevPevzner_Microstandards/EditorialChecklist/`

Ein ausgearbeitetes Beispiel mit Merge zweier Vorgängerlisten:
`/Users/robert/Documents/TCG-Ordner/_Adunka/Buchprojekte/2026_LevPevzner_AnatomyOfMistakes/260717_EditorialChecklist_AnatomyOfMistakes_v3.docx`
