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

## Tagesrhythmus

Die App ist auf einmal am Tag ausgelegt. Fällig ist alles, was bis heute Abend
dran wäre, und jedes Wort kommt höchstens einmal pro Tag — innerhalb einer Runde
darf es sich wiederholen, danach ist es für heute durch. Deshalb steht kein
Countdown auf der Startseite: entweder ist etwas fällig, oder es heisst
*Für heute erledigt*.

Wer trotzdem weitermachen will, drückt *Trotzdem üben*. Das zieht Wiederholungen
vor, bringt aber keine zusätzlichen neuen Wörter — sonst wäre die Bremse unten
wirkungslos.

## Tempo

Zwei Bremsen sorgen dafür, dass nicht zu viel Schweres auf einmal kommt.

**Die Tagesmenge passt sich an.** Der Richtwert steht in den Einstellungen, aber die
App weicht davon ab: Liegt die Trefferquote der letzten 40 Abfragen unter 65 %,
kommen nur noch 30 % davon dazu; unter 80 % sind es 60 %; über 92 % gibt es 40 %
mehr.

**Neue Wörter kommen aus einem Fenster.** Es beginnt 500 Ränge über deiner Grenze
und wächst um 60 Ränge pro Karte, die es in den Langzeitrhythmus geschafft hat.
Ohne dieses Fenster stünde nach zwei Wochen Rang 3000 an, egal wie es läuft.

Zehn simulierte Tage ab Grenze 500, je nach Trefferquote:

| Trefferquote | neue Wörter pro Tag | nach 10 Tagen |
|---|---|---|
| 90 % | 8 → 11 | 86 Wörter, bis Rang 1958 |
| 75 % | schwankt 5–8 | 62 Wörter, bis Rang 1677 |
| 50 % | fällt auf 2 | 32 Wörter, bis Rang 1501 |

Was du nicht kennst und wofür es noch keine Karte gibt, landet auf einer Wunschliste.
Unter *Daten* lässt sie sich exportieren — das ist die Vorlage für die nächste
Wortcharge.

## Wie eine Karte abläuft

**Jedes Wort beginnt als Abfrage — auch beim allerersten Mal.** Die deutsche
Bedeutung steht oben, darunter der Beispielsatz mit Lücke, darunter ein
Eingabefeld. Nichts wird vorher vorgezeigt.

Das ist Absicht: ein Rateversuch vor der Auflösung verankert ein Wort besser als
blosses Anschauen, auch wenn der Versuch danebengeht. Bei einem neuen Wort tippst
du also einmal ins Leere, drückst *Weiss ich nicht* und siehst dann die Lösung —
genau das ist der Lerneffekt. Neue Wörter tragen die Markierung *neu*.

Geurteilt wird in vier Stufen:

| Eingabe | Urteil |
|---|---|
| genau richtig | **Richtig** |
| ein Zeichen daneben | **Fast — achte auf die Schreibweise** |
| richtig, falscher Artikel | **Richtig, aber es heisst het/de** |
| ein anderes Wort aus der Liste | **Das ist „…“** mit dessen Bedeutung |

Gross- und Kleinschreibung, Satzzeichen und überzählige Leerzeichen werden
ignoriert. Den Artikel darfst du mitschreiben, musst du aber nicht.

*Weiss ich nicht* zeigt die Lösung sofort. Das Wort bleibt im Pool und kommt in
derselben Runde noch einmal.

Du bewertest nichts selbst. Die Stufe für FSRS ergibt sich aus der Antwort:

| | |
|---|---|
| falsch oder *Weiss ich nicht* | Nochmal |
| Schreibfehler oder falscher Artikel | Schwer |
| richtig, aber zäh | Schwer |
| richtig | Gut |
| richtig und flott | Leicht |

„Flott" richtet sich nach der Wortlänge: rund vier Sekunden plus 0,4 pro Zeichen
zum Lesen, Überlegen und Tippen. Über dem Zweieinhalbfachen davon zählt es als
mühsam. Auf dem Ergebnisbildschirm steht, wann das Wort dadurch wiederkommt.

Die Tastatur reicht durch die ganze Runde: Enter prüft, Enter geht weiter.

Karten, die auf „in wenigen Minuten“ gesetzt werden, hängt die Runde hinten wieder
an — höchstens zweimal pro Wort, damit eine Runde nicht endlos wird.

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
- Keine Alternativantworten: Wenn zwei niederländische Wörter dieselbe deutsche
  Bedeutung haben, zählt nur das hinterlegte. Der Beispielsatz mit Lücke soll das
  eingrenzen, trifft aber nicht immer.
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
