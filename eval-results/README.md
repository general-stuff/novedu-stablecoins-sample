# Eval-Läufe

Alle Läufe: Bewertung und Tutor-Antworten auf `Qwen/Qwen3.8-27B-FP8` (SCCH, das
Modell der Aktivitäten), Prüfung von Feedback und Tutor-Verhalten durch
`gpt-5.6-terra` (Azure Foundry, reasoning high) als Richter. Befehl siehe
README im Wurzelverzeichnis. Ein Lauf kostet rund 150.000 Eingabe- und
30.000 Ausgabe-Tokens.

| Lauf | Quiz-Fälle | Urteile korrekt | False-correct | Tutor-Gespräche | vom Richter markiert |
| --- | ---: | ---: | ---: | ---: | ---: |
| 1 (16.9.2026) | 45 | 45 | 0 | 6 | 11 (4 Tutor, 7 Quiz-Feedback) |
| 2 (16.9.2026) | 45 | 45 | 0 | 6 | 4 (0 Tutor, 4 Quiz-Feedback) |
| 3 (16.9.2026, nur Quiz 1) | 16 | 16 | 0 | 0 | 0 |

Was zwischen Lauf 1 und 2 geändert wurde (nur Formulierungen in den Prompts,
keine Golden Answers):

* **Tutor:** Bei einer Fehlvorstellung stellte der Tutor nur eine Rückfrage,
  statt sie richtigzustellen. Die Regel verlangt jetzt zuerst ein bis zwei
  Sätze Richtigstellung, dann die Rückfrage.
* **Tutor:** Die Regel „jede Zahl als ungefähr, Stand 2026“ ließ der Richter
  auch auf „1 Coin = 1 Dollar“, Jahreszahlen und Beispielbeträge anwenden. Sie
  gilt jetzt ausdrücklich nur für Marktzahlen und Statistiken.
* **Quizzes:** „spricht die Person mit du an“ wurde als Pflicht gelesen, in
  jedem Feedback ein „du“ zu verwenden. Jetzt heißt es: wenn angesprochen wird,
  dann mit „du“.

Die vier Markierungen in Lauf 2 betreffen Feedback-Texte, in denen das kleine
Modell eine Erklärung ungenau formuliert hat (zum Beispiel „wird weniger
gehandelt“ statt „wird unter 1 Dollar gehandelt“). Die Urteile waren richtig.
Solche Ausreißer wechseln von Lauf zu Lauf; sie sind ein Argument dafür, ein
stärkeres Modell zu wählen, wenn das Feedback selbst im Vordergrund steht.

Lauf 3 prüft nur das Quiz zu „Was ist ein Stablecoin?“, weil nur dort etwas
geändert wurde: Die Frage „Wie der Wert bei 1 Dollar bleibt“ wurde durch die
Bildfrage „Zwei Herausgeber im Vergleich“ ersetzt (neue ID
`zwei-herausgeber-im-bild`, vier Golden Answers statt drei). Die Rubrik
beginnt mit einer Zeile, die das Bild beschreibt, weil der Grader das Bild
nicht sieht. Alle 16 Fälle wurden richtig beurteilt, kein Feedback wurde
markiert. Die anderen drei Eval-Dateien sind unverändert und durch Lauf 2
abgedeckt.
