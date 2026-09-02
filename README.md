# Amai — Flämisch B1/B2

Ein Einstufungstest, der die Häufigkeitsgrenze deines Wortschatzes sucht, plus
Wiederholung nach FSRS für alles, was darüber liegt. Läuft offline als PWA auf dem iPhone.

## Auf das iPhone bringen

1. Alle Dateien in ein GitHub-Repo legen, in den Repo-Einstellungen unter *Pages*
   den Branch als Quelle wählen. (Cloudflare Pages oder Vercel gehen genauso —
   Hauptsache HTTPS, ohne das läuft kein Service Worker.)
2. Die Seite in **Safari** öffnen. Chrome auf iOS kann nicht auf den Homescreen installieren.
3. Teilen-Symbol → *Zum Home-Bildschirm*.

Schritt 3 ist nicht optional. Als Lesezeichen in Safari fällt die App unter die
Sieben-Tage-Regel der Intelligent Tracking Prevention und verliert nach einer Woche
ohne Besuch ihren gesamten Speicher. Web-Apps auf dem Home-Bildschirm sind davon
ausdrücklich ausgenommen und laufen in einem eigenen, von Safari getrennten
Datenbereich.

## Wie sicher ist der Lernstand

Alles liegt in `localStorage` auf dem Telefon. Nichts wird hochgeladen — auch wer die
Seite öffnet, sieht nur eine leere App.

Unter *Daten* legt **In Dateien speichern** die Sicherung über das Teilen-Menü ab.
Von dort in iCloud Drive gelegt, liegt sie ausserhalb des Telefons. Automatisch geht
das nicht: eine Web-App darf nicht von sich aus in iCloud schreiben.

Geschützt gegen: Safaris Sieben-Tage-Aufräumung, normales Schliessen, Neustart.
Beim Start fordert die App zusätzlich dauerhaften Speicher an
(`navigator.storage.persist`), was auf unterstützenden Systemen vor der automatischen
Räumung schützt. Der aktuelle Status steht unter *Daten*.

Nicht geschützt gegen: das Löschen des Icons vom Home-Bildschirm, einen vollen
Telefonspeicher (WebKit räumt dann pro Origin und immer vollständig) und den Wechsel
auf ein neues Gerät. Deshalb: unter *Daten* exportieren. Nach 30 Tagen ohne Sicherung
erinnert die Startseite daran.

Der Speicherschlüssel heisst `amai.v1` und sollte so bleiben, auch wenn die App
umbenannt wird — daran hängt der gesamte Lernstand.

## Dateien

| Datei | Zweck |
|---|---|
| `index.html` | Die ganze App: Oberfläche, Einstufung, Lernlogik |
| `words.js` | Die gelernten Wörter: deutsche Bedeutung, Wortart, Beispielsatz |
| `testwords.js` | 12 000 Prüfwörter für die Einstufung, nur Wort und Rang |
| `fsrs.js` | `ts-fsrs` 5.4.2, für den Browser gebündelt (unverändert) |
| `sw.js` | Service Worker, hält die App offline verfügbar |
| `manifest.webmanifest`, `icon-*.png` | Damit iOS es als App behandelt |

## Wie die Einstufung funktioniert

Zwei getrennte Listen. **Geprüft** wird gegen `testwords.js`: die 12 000 häufigsten
Wörter, die in Flandern tatsächlich bekannt sind. Diese Wörter brauchen keine
Übersetzung, sie werden nur abgefragt — deshalb kann die Liste gross sein.
**Gelernt** wird nur aus `words.js`, wo jedes Wort eine geschriebene Bedeutung und
einen Beispielsatz hat.

Der Test läuft als Treppe: bei Rang 1500 losgehen, bei „kenne ich“ zu selteneren
Wörtern springen, bei „nein“ zurück, und die Schrittweite bei jedem
Richtungswechsel halbieren.

Zwei Dinge fangen Selbstüberschätzung ab:

- **Gegenprobe.** Auf „ja“ folgt oft eine Multiple-Choice-Frage nach der Bedeutung.
  Daneben liegen heisst nicht gekannt.
- **Erfundene Wörter.** Etwa jedes siebte Item ist ein Kunstwort wie *verkwalmen* —
  lautlich niederländisch, aber nicht existent. Wer da „ja“ sagt, bekommt ab dann
  jede Behauptung überprüft, und am Ende einen Hinweis.

Die Grenze wird am Schluss nicht aus dem höchsten Treffer abgeleitet, sondern als
der Schnitt, der den wenigsten Antworten widerspricht. Ein einzelner Ausrutscher
verschiebt das Ergebnis dadurch kaum. Bleiben die Antworten widersprüchlich,
verlängert sich der Test von 18 auf bis zu 35 Items.

Getestet gegen simulierte Lernerinnen: ohne Antwortrauschen trifft der Schätzer den
Zielrang auf wenige Ränge genau (799, 1997, 4996, 8996 für die Ziele 800, 2000, 5000
und 9000). Bei 15 % zufällig falschen Antworten liegt der Median bei 799, 1996, 4821
und 9000. Der Test braucht dafür 19 bis 30 Fragen.

## Die fünf Fächer

Auf der Startseite steht der Fortschritt durch die Liste, gegliedert in fünf Fächer:
**Neu**, **Noch 3**, **Noch 2**, **Noch 1**, **Gekonnt**.

Das ist eine Anzeige, kein eigener Mechanismus. Darunter läuft weiter FSRS; das
Fach ergibt sich aus der Stabilität eines Wortes — also daraus, wie lange es hält,
bevor die Erinnerung unter die Zielgenauigkeit fällt:

| Fach | Stabilität |
|---|---|
| Noch 3 | unter 2 Tagen |
| Noch 2 | 2 bis 10 Tage |
| Noch 1 | 10 bis 30 Tage |
| Gekonnt | ab 30 Tagen, oder beim Erstkontakt aussortiert |

Leitner-Fächer mit festen Abständen wären die klassische Umsetzung, aber sie
ignorieren, wie schwer ein einzelnes Wort für dich ist. FSRS schätzt das pro Wort
aus den Antworten. Die Fächer sind das Bild davon.

## Aussortieren

**Beim allerersten Kontakt richtig getippt** heisst: du kennst das Wort. Es wandert
sofort nach *Gekonnt* und kommt nie wieder. Dafür ist die App gebaut — nicht Wörter
lernen, sondern die Lücken finden.

**Ein Wort, das einmal danebenging**, muss mehrere Abfragen mit wachsendem Abstand
überstehen, bevor es aussortiert wird. Ein einziger Treffer kurz nach dem Fehler
sagt fast nichts darüber aus, ob das Wort in einem Monat noch da ist.

## Tempo und Pausen

Es gibt kein Tagesziel. Neue Wörter werden **pro Runde** zugeteilt, nicht pro
Kalendertag — eine Woche Pause kostet dich also nichts.

Eine Runde besteht aus höchstens 70 % Wiederholungen, am längsten überfällige
zuerst, der Rest sind neue Wörter. Der Anteil neuer Wörter sinkt, wenn die
Trefferquote der letzten 40 Abfragen fällt.

**Ein Rückstand wird nie angezeigt und nie eingefordert.** Wer nach einem Monat
zurückkommt, bekommt dieselbe Runde wie immer; der Stau läuft über die folgenden
Runden von selbst ab. Zehn Runden simuliert, ab Grenze 500:

| Trefferquote | bearbeitete Wörter je Runde | nach 10 Runden |
|---|---|---|
| 90 % | 20 → 6 | 116 Wörter, bis Rang 2502 |
| 75 % | 20 → 6 | 88 Wörter, bis Rang 2018 |
| 50 % | 20 → 6 | 74 Wörter, bis Rang 1802 |

Die Abstände selbst rechnet FSRS weiter in echten Tagen, nicht in Nutzungstagen.
Vergessen richtet sich nach verstrichener Zeit, nicht danach, wie oft die App
offen war. Was sich an deinem Tempo orientiert, ist die **Menge**, nicht der
**Zeitpunkt**.

## Wörter ergänzen

Im Pool auf *Wort hinzufügen* — das landet ganz vorn in der Warteschlange und
überlebt Updates der App, weil es im gespeicherten Stand liegt.

Für grössere Mengen `words.js` direkt bearbeiten. Ein Eintrag:

```js
{"nl":"aarzelen","art":null,"de":"zögern","pos":"ww",
 "cloze":"Ik {aarzelde} even voordat ik antwoordde.","rank":8206}
```

`art` ist `"de"`, `"het"` oder `null`. `pos` ist `ww` (Verb), `zn` (Substantiv),
`bn` (Adjektiv) oder `bw` (Adverb). Die geschweiften Klammern markieren, was im
Lückentext verschwindet — bei trennbaren Verben beide Teile, etwa
`Het {viel} me {op} dat ...`. `rank` steuert nur, wo das Wort in der Einstufung
auftaucht; eine grobe Schätzung genügt.

Nach jeder Änderung an den Dateien die Versionsnummer in `sw.js` erhöhen
(`amai-v1` → `amai-v2`), sonst serviert der Cache weiter die alte Fassung.

## Was hier nicht drin ist

- Keine Aussprache, kein Audio.
- Wo zwei niederländische Wörter dieselbe deutsche Bedeutung tragen, zählt jedes
  von beiden als richtig; die App nennt danach das gesuchte Wort. Bei rund 3 % der
  Einträge kommt das noch vor — der Rest ist von Hand auseinandergehalten
  („der Herr (Anrede)" gegen „der Herr (Gebieter)").
- Keine Serien, keine Erinnerungen, keine Push-Nachrichten. Wenn du drei Tage nicht
  reinschaust, passiert nichts — FSRS holt das von selbst wieder auf.
- Kein Sync zwischen Geräten. Export und Import sind der Weg.

## Herkunft der Daten

Ränge und Bekanntheitswerte: Dutch Crowdsourcing Project (Brysbaert, Keuleers &
Mandera 2019, https://osf.io/5fk8d/). 54 000 Wörter, rund 300 000 Teilnehmende, mit
getrennten Werten für Belgien und die Niederlande. Der Rang ist die Position nach
SUBTLEX-NL-Häufigkeit unter den Wörtern mit `prevalence_BE >= 1.4` — also unter dem,
was Flamen wirklich kennen. Das Feld `gap` ist die Differenz Belgien minus
Niederlande: hohe Werte markieren flämisch-spezifische Wörter.

Eine Einschränkung dazu: Bekanntheit misst Wiedererkennen, nicht Sprachgebrauch. Die
Daten finden Wörter, die ausserhalb Flanderns *unbekannt* sind, nicht Wörter, die in
Flandern *bevorzugt* werden. Wer in Amsterdam `jurk` sagt, versteht `kleedje`
trotzdem.

Die deutschen Bedeutungen und Beispielsätze sind von Hand geschrieben, nicht aus einem
Wörterbuch übernommen. Bei Wörtern mit mehreren Bedeutungen steht die häufigste
(`voorkomen` → „verhindern“, nicht „vorkommen“). Wenn dir etwas schief vorkommt,
korrigiere es in `words.js`.

Scheduler: [ts-fsrs](https://github.com/open-spaced-repetition/ts-fsrs) 5.4.2, MIT.
