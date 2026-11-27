---
language_code: "nl"
display_name: "Dutch"
cefr_levels: ["A2", "B1", "B2", "C1"]
error_taxonomy:
  - de_het
  - v2_word_order
  - separable_verb
  - er_construction
  - preposition
  - lexical_choice
  - register
---

# Dutch Language Pack (`nl`)

This pack is the first concrete Language Pack for Lingua Ludi.
Meta-guidance is written in British English; all learner-facing examples are Dutch.
No campaign setting or lore appears anywhere in this pack.

## CEFR Profiles

### A2

- `max_sentence_words`: 10.
- `register_guidance`: everyday `je/jij` only; avoid polite `u` except in fixed greetings.
- `vocabulary_ceiling_notes`: high-frequency everyday vocabulary (home, work, travel, food); no idioms.
- `grammar_focus`: present tense, separable verbs in main clauses, `de/het` articles, basic questions with inversion.

### B1

- `max_sentence_words`: 15.
- `register_guidance`: everyday `je/jij`; introduce polite `u` in formal service contexts.
- `vocabulary_ceiling_notes`: everyday and work-related topics; some transparent compounds.
- `grammar_focus`: past tenses (`imperfectum`, `perfectum`), subordinate clauses with `om ... te`, `er` + preposition, modal verbs.

### B2

- `max_sentence_words`: 22.
- `register_guidance`: `je/jij` and `u` both acceptable; workplace pragmatics (requests, disagreement softeners).
- `vocabulary_ceiling_notes`: abstract and professional topics; less transparent compounds.
- `grammar_focus`: full subordinate-clause word order (verb-final), passive voice, conditional constructions, `zullen` vs `zouden`.

### C1

- `max_sentence_words`: 30.
- `register_guidance`: flexible register, including formal written Dutch and nuanced politeness.
- `vocabulary_ceiling_notes`: near-native range, including idiomatic expressions.
- `grammar_focus`: complex subordination, participle clauses, stylistic fronting, nuance in modality and aspect.

## Error Taxonomy Notes

`lingua-coach` tags each learner error with exactly one tag from this taxonomy.
Each entry gives diagnostic criteria followed by a short neutral example (incorrect → correct).

- `de_het`: wrong grammatical gender article.
  Criteria: learner uses `de` where `het` is required or vice versa for a noun.
  Example: `de huis` → `het huis`.
- `v2_word_order`: verb not in second position in a main clause.
  Criteria: finite verb appears third or later in a declarative main clause.
  Example: `Ik ga morgen naar huis` is correct; `Ik morgen ga naar huis` is the error pattern.
- `separable_verb`: separable prefix not sent to the end.
  Criteria: in main clauses the prefix stays attached (`ik belop`), in subordinate clauses the whole verb must be final.
  Example: `Ik bel hem morgen op` (correct); `Ik belop hem morgen` in a main clause is the error pattern.
- `er_construction`: missing or misplaced `er`.
  Criteria: `er` + preposition required for pronominal adverbs (`erop`, `ermee`) is absent or placed wrongly.
  Example: `Ik reken mee` → `Ik reken ermee`.
- `preposition`: wrong preposition with a verb, noun, or adjective.
  Criteria: fixed preposition mismatch (`denken aan`, `wachten op`, `bang voor`).
  Example: `Ik denk over jou` → `Ik denk aan jou`.
- `lexical_choice`: word exists but is the wrong lexical item for the meaning.
  Criteria: false friend or near-synonym misselection, not a grammar error.
  Example: `actueel` (current) used for `actually` (`eigenlijk`).
- `register`: level or politeness mismatch with the situation.
  Criteria: overly formal or overly casual form for the context (e.g. `u` among peers, `jij` in a formal complaint).
  Example: addressing a manager as `jij` in a written formal letter.

## Wingman Scaffolding Cues

Neutral reusable Dutch cues the Wingman may offer, with zero setting proper nouns.

- Clarification: `Kun je dat herhalen?`, `Bedoel je ...?`, `Wat bedoel je daarmee?`
- Hesitation fillers: `Eh...`, `Nou, even kijken...`, `Dus...`
- Polite repair: `Sorry, ik begreep je niet helemaal.`, `Kun je dat anders zeggen?`
- Asking how to say something: `Hoe zeg je ... in het Nederlands?`, `Wat is het woord voor ...?`
