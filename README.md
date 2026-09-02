# Woordje — Niederländisch B1/B2

Ein Einstufungstest, der die Häufigkeitsgrenze deines Wortschatzes sucht, plus
Wiederholung nach FSRS für alles, was darüber liegt. Läuft offline als PWA auf dem iPhone.

## Auf das iPhone bringen

1. Alle Dateien in ein GitHub-Repo legen, in den Repo-Einstellungen unter *Pages*
   den Branch als Quelle wählen. (Cloudflare Pages oder Vercel gehen genauso —
   Hauptsache HTTPS, ohne das läuft kein Service Worker.)
2. Die Seite in **Safari** öffnen. Chrome auf iOS kann nicht auf den Homescreen installieren.
3. Teilen-Symbol → *Zum Home-Bildschirm*.

Danach startet die App im Vollbild, ohne Safari-Leiste, und funktioniert offline.
Der Lernstand liegt in `localStorage` und überlebt normales Schliessen — aber nicht
das Löschen der Website-Daten. Deshalb: unter *Daten* ab und zu exportieren.

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

Was du nicht kennst und wofür es noch keine Karte gibt, landet auf einer Wunschliste.
Unter *Daten* lässt sie sich exportieren — das ist die Vorlage für die nächste
Wortcharge.

## Kartentypen

- **Erkennen** — niederländisches Wort, Bedeutung aufdecken, selbst bewerten.
  Für neue und junge Karten.
- **Lückentext** — Satz mit Lücke, vier Wörter derselben Wortart zur Auswahl.
  Kommt dazu, sobald die Stabilität einer Karte etwa eine Woche übersteigt.
  Erst wiedererkennen, dann im Kontext produzieren.

Bewertet wird mit den vier FSRS-Stufen; unter jeder steht, wann das Wort dann
wiederkommt.

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
(`woordje-v1` → `woordje-v2`), sonst serviert der Cache weiter die alte Fassung.

## Was hier nicht drin ist

- Keine Aussprache, kein Audio.
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
