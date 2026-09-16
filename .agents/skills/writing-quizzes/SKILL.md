---
name: writing-quizzes
description: >-
  Author, revise, and publish the Novedu chapter quizzes of this Stablecoins
  sample book. Use this skill whenever the user asks to create a quiz for a
  chapter, add/change/review quiz questions, adjust a grading rubric or
  evaluation prompt, write or run golden-answer evals, diagnose eval
  mismatches, or publish/update a quiz (validate, mint a code, link it from
  the chapter) — even if they don't say "quiz YAML", "eval YAML", or name the
  Novedu app. Also use it when reviewing existing *-quiz.yaml or
  *-quiz.eval.yaml files for question, rubric, or regression-test quality.
---

# Writing quizzes for the Stablecoins sample book

Each content chapter gets one LLM-graded quiz (`<chapter>-quiz.yaml`, sibling
of the `.qmd`) and one paired golden-answer regression file
(`<chapter>-quiz.eval.yaml`). Quizzes are Novedu activities: open-ended
questions only, in German, graded by a small LLM against a hidden rubric,
with an optional per-question discussion chat. Students use them as anonymous
self-checks. Eval files are teacher-only test data; never publish them as
activities or mint codes for them.

This is a **sample book**, so every quiz is self-contained: no fragment
libraries, no compound quizzes, no images, no research questions. The grading
context is repeated inline in every quiz (it is short).

Ground truth for the platform lives in the Novedu repo
(`~/github/chat-prototype`): authoring guide `activities/quizzes/README.md`,
eval guide `activities/evals/README.md`, and the CLI skill
`.agents/skills/novedu-tutor-cli/SKILL.md`. Don't re-derive platform rules —
the CLI validates with the app's exact pipeline.

Read `references/question-design.md` before writing or reviewing questions —
it distills the assessment literature and ends with the audit checklist.
Read `references/golden-answer-evals.md` whenever creating or changing a quiz,
touching an `evaluation` rubric, reviewing an eval file, or diagnosing grading.

## Chapter quiz skeleton

Copy this structure exactly:

```yaml
# yaml-language-server: $schema=https://raw.githubusercontent.com/Teaching-HTL-Leonding/novedu-chat-mvp/refs/heads/main/activities/quizzes/quiz-yaml.schema.json
# Quiz zum Kapitel „<Titel>“ (<folder>/<file>.qmd).

id: <chapter-slug>-quiz
name: "Quiz: <Kapiteltitel>"
title: "Teste dein Wissen: <Kapiteltitel>"
description: |
  <N> kurze Fragen zum Kapitel *<Titel>*. Antworte in eigenen Worten. Es gibt
  keine Antwortmöglichkeiten zum Anklicken, und das Quiz ist anonym: Es ist
  für dich da, damit du siehst, was du schon verstanden hast und was du noch
  einmal lesen solltest. Ein bis zwei Sätze reichen meistens.

llm:
  model: Qwen/Qwen3.8-27B-FP8
  provider: SCCH

# Shared context for BOTH the grader and the discussion chat. Identical in
# every quiz of this book (keep it so: edit all three when you change it).
instructions: |
  Die Schülerinnen und Schüler sind 14 bis 18 Jahre alt und besuchen den
  Wirtschaftskundeunterricht einer österreichischen Schule. Sie haben das
  Kapitel „<Titel>“ eines kurzen Buchs über Stablecoins gelesen. Sie kennen
  die drei Funktionen des Geldes, den Unterschied zwischen Bargeld und
  Bankguthaben, Inflation und Wechselkurse. Blockchain-Technik kennen sie
  nicht und müssen sie nicht erklären.

  Bewertung. Die Kriterien jeder Frage nennen die erwartete Antwort und die
  Aspekte, die die Antwort enthalten muss. Prüfe die inhaltliche Richtigkeit:
  - `correct`: Jeder erforderliche Aspekt ist vorhanden, und nichts in der
    Antwort ist falsch.
  - `partial`: Ein Teil der erforderlichen Aspekte ist vorhanden, oder die
    Antwort ist im Kern richtig, enthält aber einen falschen Punkt.
  - `incorrect`: Kein erforderlicher Aspekt ist vorhanden, oder die Kernaussage
    ist falsch.
  Wenn du zwischen zwei Urteilen schwankst, nimm das niedrigere.

  - Die Schülerinnen und Schüler antworten in eigenen Worten und oft
    umgangssprachlich. Alltagsformulierungen zählen, wenn die Idee stimmt;
    verlange keinen Fachbegriff, den die Kriterien nicht nennen.
  - Rechtschreibung und Grammatik zählen nie.
  - Eine kurze Antwort ist bei gleicher Richtigkeit genauso gut wie eine
    lange. Belohne nie die Länge.
  - Das Feedback ist auf Deutsch und in einfacher Sprache. Sprichst du die
    Person an, dann mit „du“, nie mit „Sie“; du musst sie aber nicht in jedem
    Feedback ansprechen. Wenn das Urteil nicht `correct` ist, nenne die richtige
    Antwort. Bei `partial` sage zuerst, was stimmt, und dann, was fehlt.
  - Erwähne nie Punkte, Noten oder die Bewertungskriterien, und schreibe das
    Urteil oder ein Etikett wie „Urteil:“ nie ins Feedback. Beginne direkt mit
    dem Inhalt.
  - Keine Anlageberatung: Empfiehl nie, Kryptowerte zu kaufen oder zu meiden.

discussion:
  instructions: |
    Du darfst die richtige Antwort nennen und erklären. Bleib bei dieser einen
    Frage und beim Inhalt des Kapitels „<Titel>“, halte jede Antwort kurz
    (höchstens fünf Sätze) und nimm ein konkretes Beispiel aus dem Alltag,
    wenn es hilft. Antworte auf Deutsch.

questions:
  - id: <kebab-case-stable-id> # unique, never renamed (stats key)
    title: "<Kurzes Etikett>"
    question: |
      <Student-visible Markdown, German. See references/question-design.md.>
    evaluation: |
      <Server-only rubric, German. Template below.>
```

Defaults deliberately NOT set (do not add them): `anonymous` (defaults true —
self-check), `shuffle` (defaults true), `question_count` (chapter quizzes ask
everything).

## Question rules

- **One concept per question, one or at most two sub-questions.** "Was
  passiert, und warum?" is the allowed maximum. Three asks in one prompt is a
  hard rule violation — split or cut.
- Questions and rubrics in **German**. Answers are graded on content, not
  language.
- Write student-facing text per the `student-technical-writing` skill and
  `AGENTS.md`: warm, plain, short sentences, du-Form, no dashes.
- Cover the chapter's actual emphases, not trivia; prefer explanation,
  application to a case, and "Stimmt das?" stems over definition recall.
- Numbers in questions only as Größenordnungen („rund 300 Milliarden“).
- `id`s are permanent — they key the teacher statistics. Choose meaningful
  kebab-case; never rename after publishing.
- Four to five questions per chapter.

## Rubric (`evaluation`) template

Grading runs on a small model (`Qwen/Qwen3.8-27B-FP8` on SCCH, free). The
generic verdict rules live in the quiz's `instructions:`, so a rubric states
only what the grader must verify:

```
Erwartet: <die erwartete Antwort, EINMAL, 1 bis 3 Zeilen>.
Erforderlich: <die Aspekte, die genannt sein müssen, nummeriert bei mehreren>.
<optional> Teilweise: <eine konkrete halb richtige Antwort, die es zu nennen lohnt>.
<optional> Falsch: <höchstens eine typische Fehlvorstellung>.
```

Start WITHOUT paraphrase lists or tone rules. Then run the golden-answer eval:
where the small grader gets a case wrong, add the one line that fixes it and
nothing more. Stay under about 8 lines unless evals forced more. The rubric is
server-only and may state answers freely.

## Golden-answer evals

Treat the paired `<chapter>-quiz.eval.yaml` as part of the quiz. Per question
one clearly `correct` answer, one `partial` that mirrors the rubric's boundary,
one confidently `incorrect` answer (a typical misconception from the
tutor's knowledge base where one fits). Write answers the way 15-year-olds type in
German: short, everyday words, lowercase starts, harmless typos. Keep a
one-line comment above a non-obvious case. All cases are synthetic.

Run the eval after writing or changing a quiz. Grading runs on the quiz's own
Qwen model; the audit of the feedback text uses the frontier judge:

```bash
cd ~/github/chat-prototype
npm run cli --silent -- validate /abs/path/<chapter>-quiz.yaml --kind quiz
npm run cli --silent -- validate /abs/path/<chapter>-quiz.eval.yaml --kind eval
npm run cli --silent -- eval /abs/path/<chapter>-quiz.eval.yaml \
  --judge-llm-provider "Azure Foundry" --judge-llm-model gpt-5.6-terra \
  --judge-llm-reasoning high --report /abs/path/eval-results/<name>.md
```

A failing case is the trigger, and the only trigger, for adding context to
that question's rubric. Reports are committed under `eval-results/`.

## Publish workflow

Quizzes are served straight from the book's public GitHub repo
(`general-stuff/novedu-stablecoins-sample`); the Novedu server re-reads the raw
URL on every load, so publishing an edit = `git push`.

1. Author/edit the quiz YAML and its sibling eval.
2. Validate both (commands above).
3. Run the eval and repair genuine mismatches.
4. Commit and push (quiz edits go live immediately for existing codes).
5. First publish only: add ONE entry to `stablecoins-activities.yaml` under
   `activities.quizzes`, keyed by chapter slug, then run
   `npm run cli --silent -- codes sync /abs/path/stablecoins-activities.yaml`.
   That mints the new code and rewrites `stablecoins-activities.lock.yaml`;
   commit registry AND lock file.
6. First publish only: add `## Teste dein Wissen {#sec-<slug>-quiz}` at the
   end of the chapter `.qmd`: one sentence naming the question count, then
   `{{< quiz <chapter-slug> title="<Kapiteltitel>" >}}` — the registry KEY,
   never a code. An unknown key fails the render.

**Never `codes create` a book quiz and never paste a code into a chapter.**
