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
| `words.js` | 256 Wörter mit deutscher Bedeutung, Wortart und Beispielsatz |
| `fsrs.js` | `ts-fsrs` 5.4.2, für den Browser gebündelt (unverändert) |
| `sw.js` | Service Worker, hält die App offline verfügbar |
| `manifest.webmanifest`, `icon-*.png` | Damit iOS es als App behandelt |

## Wie die Einstufung funktioniert

Die Wörter sind nach Häufigkeitsrang sortiert (aus dem OpenSubtitles-Korpus,
`hermitdave/FrequencyWords`, Liste `nl_50k`). Der Test läuft als Treppe: bei etwa
Rang 2500 losgehen, bei „kenne ich“ zu selteneren Wörtern springen, bei „nein“
zurück, und die Schrittweite bei jedem Richtungswechsel halbieren.

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
Zielrang exakt, bei 15 % zufällig falschen Antworten liegt der Median bei Rang 1500,
4000 und 9000 weiterhin auf dem Ziel. Oberhalb von Rang 20000 wird es unzuverlässig —
dort enthält die Liste schlicht zu wenige Wörter.

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

Häufigkeitsränge: OpenSubtitles-2018-Wortformen aus `hermitdave/FrequencyWords`
(CC-BY-SA 4.0). Es sind Wortform-Ränge, keine Lemma-Ränge — die Infinitivform eines
Verbs steht deshalb tiefer in der Liste, als das Lemma insgesamt vorkommt. Für die
grobe Sortierung nach Schwierigkeit reicht das.

Die deutschen Bedeutungen und Beispielsätze sind von Hand geschrieben, nicht aus einem
Wörterbuch übernommen. Bei Wörtern mit mehreren Bedeutungen steht die häufigste
(`voorkomen` → „verhindern“, nicht „vorkommen“). Wenn dir etwas schief vorkommt,
korrigiere es in `words.js`.

Scheduler: [ts-fsrs](https://github.com/open-spaced-repetition/ts-fsrs) 5.4.2, MIT.
