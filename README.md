# Stablecoins: ein Novedu-Beispielbuch

Ein kurzes deutsches Buch über Stablecoins für den Wirtschaftskundeunterricht in
der Sekundarstufe II, gebaut als **Beispiel dafür, wie [Novedu](https://novedu.at)
ein Schulfach außerhalb der Informatik unterstützen kann**: ein KI-Tutor für das
ganze Thema, ein KI-bewertetes Quiz pro Kapitel und eine Schreibaufgabe mit
KI-Coach. Es ist kein fertiges Unterrichtsmaterial.

Ein `quarto render` erzeugt aus denselben Quellen zwei Ausgaben:

* ein HTML-Buch (Kapitelnavigation, vor/zurück) in `_output/`
* ein gemeinsames PDF, `_output/book.pdf`, für den Druck

Jeder Push auf `main` veröffentlicht beides über GitHub Pages: das Buch unter
<https://general-stuff.github.io/novedu-stablecoins-sample/> und das PDF unter
<https://general-stuff.github.io/novedu-stablecoins-sample/book.pdf>.

Die technische Struktur (Quarto-Buch, Shortcodes, Aktivitäts-Registry, Skills,
CI) ist von den Creative-Coding-Büchern übernommen. Bewusst weggelassen, weil es
ein Beispiel ist: Fragment-Bibliotheken, Sammel-Quizzes, Tools im Tutor. Jede
Aktivität ist eine einzelne, für sich lesbare YAML-Datei. Eine einzige Frage im
ersten Quiz zeigt ein Bild (eine SVG-Grafik neben der Quiz-Datei), damit das
Beispiel auch diese Möglichkeit vorführt.

## Aufbau des Repositories

| Pfad | Was es ist |
| --- | --- |
| `_quarto.yml` | Buchdefinition: Kapitelreihenfolge, Ausgabeformate, Novedu-Basis-URL |
| `index.qmd` | Das Vorwort |
| `AGENTS.md` | Der Schreibvertrag für alle, die ein Kapitel oder eine Aktivität bearbeiten (Mensch oder Agent): Zielgruppe, Sprache, Kapitelform, Aktivitätsregeln, CLI-Befehle. `CLAUDE.md` ist ein Symlink darauf |
| `0010-einstieg/` | Das einleitende Kapitel, das den Tutor vorstellt, dazu `stablecoin-tutor.yaml` (die Tutor-Aktivität) und `stablecoin-tutor.eval.yaml` (Verhaltens-Eval, nur für Lehrkräfte) |
| `0020-stablecoins/` | Die drei Inhaltskapitel. Neben jedem `.qmd` liegt sein `<kapitel>-quiz.yaml` (das Quiz mit offenen Fragen und Bewertungsrubriken) und `<kapitel>-quiz.eval.yaml` (Golden-Answer-Regressionsfälle). `0030-kommentar-writing.yaml` ist die Schreibaufgabe mit Coach |
| `stablecoins-activities.yaml` | Die Novedu-Aktivitäts-Registry: jede Aktivität unter einem stabilen Schlüssel. Handgeschrieben |
| `stablecoins-activities.lock.yaml` | Generierte Zuordnung Schlüssel → Aktivitätscode. Wird von `codes sync` neu geschrieben; nicht von Hand ändern |
| `_extensions/` | Quarto-Shortcodes `quiz`, `tutor` und `writing`: sie machen aus einem Registry-Schlüssel eine Box mit Link (im PDF zusätzlich QR-Code und Adresse) |
| `eval-results/` | Die Markdown-Berichte der Eval-Läufe (Bewertung durch Qwen, Prüfung durch gpt-5.6 als Richter) |
| `.agents/skills/` | Skills für KI-Agenten: `student-technical-writing`, `writing-quizzes`, `writing-for-agents`. `.claude` ist ein Symlink darauf |
| `.github/workflows/` | CI: rendert das Buch, lädt PDF und Website als Artefakte hoch und veröffentlicht auf GitHub Pages |
| `emoji-pdf.lua`, `pdf-compact.tex`, `styles.css` | Emoji-Ersatz im PDF, kompaktes Drucklayout, kleine HTML-Anpassungen |
| `_output/`, `.quarto/` | Build-Ausgabe, nicht versioniert |

## Buch bauen

```bash
quarto render          # beide Formate nach _output/
quarto preview         # HTML mit Live-Reload beim Schreiben
```

Du brauchst Quarto (die CI pinnt die Version), eine LaTeX-Distribution (TinyTeX
reicht) für das PDF und `rsvg-convert`.

## Wie die Novedu-Aktivitäten ins Buch kommen

Ein Kapitel nennt eine Aktivität nur über ihren Schlüssel in der Registry:

```markdown
{{< tutor stablecoin-tutor >}}
{{< quiz was-ist-ein-stablecoin title="Was ist ein Stablecoin?" >}}
{{< writing kommentar title="Dein Kommentar für die Schülerzeitung" >}}
```

Der Shortcode schlägt den Schlüssel in `stablecoins-activities.lock.yaml` nach
und baut daraus den Link `https://novedu.at/<code>`. Ein unbekannter Schlüssel
bricht den Render ab, ein toter Link kann so nicht entstehen.

Novedu liest die YAML-Dateien bei jedem Aufruf direkt von der Raw-URL dieses
Repositories. Eine Änderung an einem Quiz oder am Tutor veröffentlichen heißt
darum: `git push`. Nur neue Aktivitäten brauchen einen neuen Code:

```bash
cd ~/github/chat-prototype          # das Novedu-Repository mit der CLI
npm run cli --silent -- codes sync /pfad/zu/stablecoins-activities.yaml
```

Danach Registry und Lock-Datei gemeinsam committen.

## Aktivitäten prüfen und testen

```bash
cd ~/github/chat-prototype
# Offline, kostenlos, keine Anmeldung nötig:
npm run cli --silent -- validate /pfad/zu/0010-einstieg/stablecoin-tutor.yaml --kind tutor
npm run cli --silent -- prompts  /pfad/zu/0010-einstieg/stablecoin-tutor.yaml --kind tutor

# Evals: bewertet auf dem Modell der Aktivität (Qwen 3.8 bei SCCH), das
# Feedback und das Tutor-Verhalten prüft ein stärkeres Modell als Richter.
npm run cli --silent -- eval "/pfad/zu/**/*.eval.yaml" \
  --judge-llm-provider "Azure Foundry" --judge-llm-model gpt-5.6-terra \
  --judge-llm-reasoning high --report /pfad/zu/eval-results/<name>.md
```

Die Eval-Dateien sind Testdaten für Lehrkräfte: Sie bekommen nie einen Code,
und Schülerinnen und Schüler sehen sie nie. Die Berichte liegen in
`eval-results/`.
