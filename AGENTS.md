# Stablecoins: ein Novedu-Beispielbuch

This repository holds a small **German** book on stablecoins for
Wirtschaftskunde in the Sekundarstufe II. It exists to show how Novedu
(https://novedu.at) can support a school subject that is not programming:
one AI tutor for the whole topic, one AI-graded quiz per chapter, and one
writing task with an AI coach. Chapters are Quarto Markdown (`*.qmd`) in the
numbered folders, and `quarto render` turns them into an HTML book and one PDF
handout (same setup as the Creative Coding books in `~/github/ddp-second`).

It is a **sample, not real teaching material**. Everything stays as plain as
it can be: no fragment libraries, no compound quizzes, no images, no tools, no
research boxes. Every Novedu YAML file is self-contained so a teacher can read
one file and understand the whole activity.

The facts, numbers, and dates in the chapters were researched in September
2026 and cross-checked against each other; the chapters and the tutor's
knowledge base (`0010-einstieg/stablecoin-tutor.yaml`, which also lists the
typical misconceptions) are the source of truth for every activity. When a
quiz rubric, an eval case, or the coach needs a fact, take it from the
chapter it belongs to, and never invent a number.

## Audience

Schülerinnen und Schüler der Sekundarstufe II, 14 bis 18 Jahre, in
Wirtschaftskunde, Geografie und Wirtschaftskunde (GWK) oder BWL an
berufsbildenden Schulen in Österreich.

What they bring: the three functions of money,
the difference between cash from the central bank and Giralgeld from a bank,
inflation, exchange rates, and a rough idea that Bitcoin is a digital
currency without a central bank whose price swings. They do not know
blockchain technology, and the book does not teach it beyond "Technik, die
Überweisungen ohne Bank ermöglicht".

## Language and tone

* **German throughout**: chapters, quiz questions, tutor and coach prompts,
  eval cases. YAML field names and activity ids stay English.
* Students are addressed with **du**. The tone is sachlich and warm: neither
  Krypto-Werbung noch Panikmache, and never an Anlageempfehlung. Nothing in
  the book or its activities encourages buying crypto assets.
* Österreichisches Standarddeutsch: „Jänner“, „Schülerinnen und Schüler“ or
  a neutral plural, Euro amounts as „100.000 Euro“.
* Numbers are Größenordnungen: „rund 300 Milliarden Dollar (Stand 2026)“,
  never a precise figure without „rund“ and a date.
* Quotes: German „…“ quotation marks are fine in prose. No em-dashes or
  en-dashes anywhere; use a comma, a colon, or a new sentence. No
  ellipsis „…“ in prose.

## Writing student-facing prose

Every chapter, every quiz question, and every tutor or coach prompt follows
the `student-technical-writing` skill. Load it before drafting or reviewing.
Two of its rules are overridden here: the text is German, not American
English, and the grep audit for curly quotes ignores German „…“ marks (run
`grep -nP '[\x{2014}\x{2013}]'` for dashes; that one must come back empty).
Everything else applies: the chant, the keynote voice, plain words, one idea
per sentence, sentence-case headings, no glossary, no history of the material.

## Chapter shape

* Front matter is a single `title:`. Headings are `## Überschrift {#sec-<slug>}`
  with unique anchors.
* A chapter runs 900 to 1400 words, opens with a concrete situation or
  question, and explains a term where it first matters. One or two Markdown
  tables per chapter are fine (comparison of Geldformen, Bauarten, Pro/Kontra).
* Each content chapter carries, right after its intro paragraph, one or two
  sentences that say what to ask the tutor about in this chapter, followed by
  the box `{{< tutor stablecoin-tutor >}}`.
* Each content chapter ends with `## Teste dein Wissen {#sec-<slug>-quiz}`: one
  sentence naming the question count, then
  `{{< quiz <registry-key> title="<Kapiteltitel>" >}}`. The writing task is
  linked with `{{< writing kommentar title="..." >}}`.
* Callouts: `::: {.callout-note}` for „Stimmt das?“ boxes that correct a
  misconception, `::: {.callout-tip}` for a Merksatz. Use them sparingly.
* Activities are linked only through the shortcodes (`{{< tutor >}}`,
  `{{< quiz >}}`, `{{< writing >}}`), by registry key, never by pasted URL.

## Novedu activities

* Files sit next to the chapter: `<chapter>-quiz.yaml`,
  `<chapter>-quiz.eval.yaml`, `0030-kommentar-writing.yaml`; the tutor and its
  eval live in `0010-einstieg/`.
* Every activity runs on `Qwen/Qwen3.8-27B-FP8`, provider `SCCH`, no
  `reasoning` level.
* Quiz and eval rules: `writing-quizzes` skill. Rubrics are inline
  (`Erwartet / Erforderlich / Teilweise / Falsch`), grading context is inline
  in `instructions:`, there are no fragments.
* The registry `stablecoins-activities.yaml` maps keys to files. After adding
  an activity, push, then run `codes sync` (below) and commit the regenerated
  `stablecoins-activities.lock.yaml`. Never `codes create` a book activity and
  never paste a code into a chapter.

## The Novedu CLI

Run it from the Novedu repo with absolute paths:

```bash
cd ~/github/chat-prototype
npm run cli --silent -- validate /abs/path/file.yaml --kind quiz|tutor|writing|eval
npm run cli --silent -- prompts  /abs/path/file.yaml --kind tutor|writing|quiz
npm run cli --silent -- codes sync /abs/path/stablecoins-activities.yaml
npm run cli --silent -- eval "/abs/path/**/*.eval.yaml" \
  --judge-llm-provider "Azure Foundry" --judge-llm-model gpt-5.6-terra \
  --judge-llm-reasoning high --report /abs/path/eval-results/<name>.md
```

`validate` and `prompts` are offline and free. `codes sync` and `eval` need a
signed-in teacher (`whoami`). Evals grade on the activity's own Qwen model
and audit the feedback with the frontier judge; reports go to
`eval-results/` and are committed.
