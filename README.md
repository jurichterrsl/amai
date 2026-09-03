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
| `fsrs.js` | `ts-fsrs` 5.4.2, für den Browser gebündelt (unverändert) |
| `sw.js` | Service Worker, hält die App offline verfügbar |
| `manifest.webmanifest`, `icon-*.png` | Damit iOS es als App behandelt |

## Wo es anfängt

Bei Rang 1. Es gibt keinen Einstufungstest — die Abfrage selbst ist der Test.

Ein Wort, das du beim ersten Kontakt richtig tippst, wandert sofort nach *Gekonnt*
und kommt nie wieder. Die ersten paar hundert Wörter sind damit in ein paar Runden
abgehakt, und im Unterschied zu einem Ja/Nein-Test prüfen sie dabei auch den
Artikel: gut jedes vierte niederländische Substantiv ist ein *het*-Wort, und ein
angekreuztes „kenne ich" hätte das nie aufgedeckt.

## Zwei Modi

**Abfrage** — der Normalbetrieb. Eine gemischte Runde aus fälligen Wiederholungen
und neuen Wörtern, alles getippt. Das ist der Modus, der den Lernstand vorantreibt.

**Einprägen** — für Wörter, die hängenbleiben. Er nimmt ausschliesslich Wörter aus
dem Fach *Noch 3*, also die, die dir zuletzt nicht sassen, und geht mit ihnen
dreimal durch, jeweils vollständig, bevor der nächste Durchgang beginnt:

1. **Karte** — deutsche Seite, Tippen dreht auf die niederländische. Auf der
   Rückseite steht beides untereinander, damit sich die Formen vergleichen lassen.
   Nur ansehen.
2. **Wählen** — die deutsche Bedeutung und vier niederländische Wörter zur Auswahl,
   drei davon aus derselben Wortart mit ähnlicher Häufigkeit.
3. **Tippen** — wie in der Abfrage, mit Artikel.

**Nur der dritte Durchgang zählt für FSRS.** Karte und Auswahl sind Vorbereitung:
Wiedererkennen ist deutlich leichter als Produzieren, und wenn das mitgerechnet
würde, hielte der Scheduler dich für sicherer, als du bist.

**Welche Wörter genommen werden.** Aus *Noch 3* die mit der niedrigsten Stabilität
zuerst — das sind die, die dem Vergessen am nächsten sind. Was heute schon geübt
wurde, bleibt aussen vor.

Der Tagesfilter ist nicht nur gegen Wiederholung. FSRS wertet eine Abfrage am
selben Tag fast gar nicht: Stabilität 0,3 wird zu 0,34, das Wort bliebe für immer
in *Noch 3*. Einen Tag später bringt dieselbe Antwort 0,3 auf 1,95 und mit *Leicht*
auf 3,39 — und damit wandert es nach *Noch 2*. Der Abschlussbildschirm sagt, wie
viele aufgestiegen sind.

Die Modi stehen auf der Startseite untereinander, gleich eingefärbt, jeder mit
einer Zeile, die sagt, was er tut. *Einprägen* erscheint nur, wenn für heute noch
etwas in *Noch 3* liegt, und nennt die Anzahl.

Ein weiterer Modus ist ein weiterer Eintrag in der Liste `MODES` in `index.html`;
das Markup passt sich an.

## Wie eine Karte abläuft

**Jedes Wort beginnt als Abfrage — auch beim allerersten Mal.** Die deutsche
Bedeutung steht oben, darunter ein Eingabefeld. Nichts wird vorher vorgezeigt.
Ein Rateversuch vor der Auflösung verankert ein Wort besser als blosses Anschauen,
auch wenn der Versuch danebengeht.

Wo immer das niederländische Wort zu sehen ist — auf der Auflösung wie auf der
Kartenrückseite — steht die deutsche Bedeutung darunter, damit sich beide
gegenüberstellen lassen.

Wo ein Beispielsatz hinterlegt ist, steht er mit Lücke dazwischen. Die 256 ältesten
Einträge haben einen, die übrigen nicht.

Geurteilt wird in sechs Stufen:

| Eingabe | Urteil |
|---|---|
| genau richtig | **Richtig.** |
| ein Zeichen daneben | **Fast — achte auf die Schreibweise.** |
| Artikel fehlt bei einem Substantiv | **Fast — mit Artikel: de/het …** |
| richtig, falscher Artikel | **Richtig, aber es heisst het/de …** |
| anderes Wort mit derselben Bedeutung | **Auch richtig. Gesucht war …** |
| ein anderes Wort aus der Liste | **Das ist „…"** mit dessen Bedeutung |
| sonst | **Leider nicht.** |

Gross- und Kleinschreibung, Satzzeichen und überzählige Leerzeichen werden
ignoriert. **Bei Substantiven gehört der Artikel dazu** — „de kans", nicht „kans".
Das Geschlecht lässt sich im Niederländischen nicht ableiten, also muss es
mitgelernt werden.

*Weiss ich nicht* zeigt die Lösung sofort.

**In derselben Runde wiederholt wird nur, was du gar nicht wusstest** — eine falsche
Antwort oder *Weiss ich nicht*, und höchstens einmal. Ein Tippfehler oder ein
falscher Artikel zählt als gewusst und kommt erst zum nächsten Termin wieder. Eine
Runde von sieben Karten wird damit auch im schlechtesten Fall nicht länger als
vierzehn Abfragen.

Du bewertest nichts selbst. Die Stufe für FSRS ergibt sich aus der Antwort:

| | |
|---|---|
| falsch oder *Weiss ich nicht* | Nochmal |
| Schreibfehler oder falscher Artikel | Schwer |
| richtig, aber zäh | Schwer |
| Wort mit gleicher Bedeutung | Gut |
| richtig | Gut |
| richtig und flott | Leicht |

„Flott" richtet sich nach der Wortlänge: rund vier Sekunden plus 0,4 pro Zeichen
zum Lesen, Überlegen und Tippen.

**Bedienung.** *Prüfen* und *Weiter* liegen an derselben Stelle, damit der Daumen
nicht wandern muss. Enter prüft, danach genügt ein Tipp auf *Weiter*. Das
Eingabefeld bekommt den Fokus **synchron in derselben Berührung**, die die Karte
umblättert — iOS öffnet die Tastatur nur dann von selbst. Ein `setTimeout`
dazwischen würde die Kette unterbrechen, und du müsstest jedes Mal erst ins Feld
tippen.

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

## Pool

Der Pool ist die Übersicht über die fünf Fächer. Oben eine Reihe von Filtern mit
den Zahlen, darunter ein Suchfeld, darunter die Wörter des gewählten Fachs mit
Bedeutung und nächstem Termin.

- **✕** legt ein Wort dauerhaft als gekonnt ab.
- **↺** holt ein aussortiertes Wort zurück in den Pool — falls du es beim ersten
  Kontakt geraten hast.
- *Neu* steht in der Reihenfolge der Warteschlange, die Lernfächer nach nächstem
  Termin, *Gekonnt* nach Häufigkeit.

Pro Fach werden 150 Wörter gezeigt; bei mehr hilft die Suche, die über
Niederländisch und Deutsch geht. Das gewählte Fach bleibt erhalten, auch nach
*Wort hinzufügen*.

## Aussortieren

**Beim allerersten Kontakt richtig getippt** heisst: du kennst das Wort. Es wandert
sofort nach *Gekonnt*, es wird gar keine Karte angelegt, und der Bildschirm sagt
*Gekonnt — kommt nicht mehr*, statt einen Termin zu nennen. Dafür ist die App
gebaut — nicht Wörter lernen, sondern die Lücken finden.

**Ein Wort, das einmal danebenging**, muss mehrere Abfragen mit wachsendem Abstand
überstehen, bevor es aussortiert wird. Ein einziger Treffer kurz nach dem Fehler
sagt fast nichts darüber aus, ob das Wort in einem Monat noch da ist.

## Tempo und Pausen

Es gibt kein Tagesziel. Neue Wörter werden **pro Runde** zugeteilt, nicht pro
Kalendertag — eine Woche Pause kostet dich also nichts.

Eine Runde umfasst standardmässig **sieben Karten** (unter *Daten* änderbar) und
besteht aus höchstens 70 % Wiederholungen, am längsten überfällige zuerst, der Rest
sind neue Wörter.

**Eine Runde hat immer die eingestellte Länge**, solange es überhaupt so viel zu
tun gibt. Reichen die neuen Wörter nicht, füllen Wiederholungen auf, und umgekehrt.

Es gibt keine Bremse nach Trefferquote mehr. Sie mass etwas anderes, als sie zu
messen vorgab: die Wiederholungsschlange besteht per Bauart aus genau den Wörtern,
die gerade nicht sitzen, also sinkt die Quote zwangsläufig, je länger man übt — und
die Runde wurde kürzer, obwohl nichts falsch lief.

Gegen zu viel Schweres auf einmal wirken zwei andere Dinge, die das ohnehin besser
tun: das **Schwierigkeitsfenster**, das die Ränge begrenzt, und die **Verdrängung** —
sammeln sich Fehler an, füllen Wiederholungen die Runde und drücken die neuen
Wörter von selbst auf zwei pro Runde herunter.

**Ein Rückstand wird nie angezeigt und nie eingefordert.** Wer nach einem Monat
zurückkommt, bekommt dieselbe Runde wie immer; der Stau läuft über die folgenden
Runden von selbst ab.

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

## Aktualisieren

Nach jeder Änderung an den Dateien **zwei Stellen** hochzählen, sonst serviert der
Cache weiter die alte Fassung:

- `CACHE` in `sw.js` (`amai-v12` → `amai-v13`)
- `VERSION` in `index.html` (`v12` → `v13`)

Die App sucht beim Start selbst nach einer neuen Fassung und lädt einmal neu,
sobald der neue Service Worker übernimmt. Unter *Daten* stehen die laufende
Version und ein Knopf *Nach Aktualisierung suchen*.

Wenn das iPhone trotzdem an der alten Fassung hängt, in dieser Reihenfolge:

1. App über den App-Umschalter ganz schliessen und zweimal neu öffnen — beim ersten
   Start wird der neue Service Worker geholt, beim zweiten ist er aktiv.
2. Unter *Daten* auf *Nach Aktualisierung suchen* tippen.
3. Die Seite in Safari mit angehängtem `?v=2` öffnen. Diese Adresse steht nicht im
   Cache, wird also frisch geladen.
4. Erst als letztes Mittel: Icon vom Home-Bildschirm löschen und neu hinzufügen.
   **Das löscht den Lernstand** — vorher unter *Daten* sichern.

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
