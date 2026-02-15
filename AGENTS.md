# Fixed Income Book — Agent Instructions (Evidence-First)

This repo is a fixed income / rates / credit textbook written as markdown chapters under `chapters/`.

There are multiple agents working on this project. 
Do not modify chapters which you are not working on.
Do not run git restore. 
Don't commit into git.
refactor_plan/ directory is deliberately under git ignore. Don't change .gitignore.

## Mission

- Produce **publication-ready** chapters with **grounded correctness**.
- Prefer grounding **non-trivial claims** (definitions, conventions, algorithm steps, “desk reality” practices) in evidence from `books_rag`. When needed, supplement with authoritative online primary sources (ISDA/ARRC/CCP rulebooks, central banks, regulators).
- Use your own knowledge to improve **clarity and intuition**, but avoid asserting time-varying market practice without a dated authoritative source. If something remains unclear, mark it inline as `NOT SURE: ...` with the precise ambiguity.

## Repo Layout

- `CONTENTS.md`: table of contents + chapter-by-chapter improvement notes (use as the improvement plan source).
- `chapters/`: one chapter per file (e.g., `chapters/chapter_18_ois_discounting_curve.md`).
- `CLAUDE.md`: legacy authoring guidance; treat `AGENTS.md` as the authoritative workflow when they differ.
- `books/`: local reference corpus (may be gitignored); do **not** edit or commit book contents.

## Iterative Workflow (work in small batches)

Work in batches of **1–3 chapters** at a time so the human can review quickly.

For each chapter:
1. **Read** the full chapter + its section in `CONTENTS.md` (avoid duplicating adjacent chapters; add cross-references instead).
2. **Collect evidence first** via `books_rag.answer()`; then refine with `books_rag.search_structured()`; expand context with `books_rag.fetch_chunk()`.
3. **Draft only what you can support** (evidence-first). Keep citations light (see “Citations” below).
4. **Run `books_rag.verify_draft()`** on the modified section(s) and rewrite until substantive sentences are supported.
5. **Replace placeholders** (e.g., “dont know”) with supported content; if still unclear, leave a precise `NOT SURE: ...` marker.
6. Keep chapter structure consistent: intro → numbered sections → summary → key concepts table → (notation if needed) → flashcards → mini problem set → references.

## Evidence-First Rules (books_rag MCP)

Use these MCP tools (do not write prose before Step A is done):

- `answer(query, alpha=0.3, rerank=true, limit=8)` → evidence pack (chunks + neighbor expansion)
- `search_structured(query, ..., limit=20–50)` → targeted hunting (filters, path scoping)
- `fetch_chunk(uuid)` → pull full context for quoting/formulas
- `verify_draft(text, support_threshold=3.0, max_sentences=30)` → per-sentence support checking
- `resolve_doc(path, ...)` → optional scoping for internal docs

### Citations (required)

Keep citations **minimal** for readability (no UUIDs in the prose). Aim for **~3–6 references per chapter**, added in a short `## References` section near the end. Use one of:

- Parenthetical citation: `(Author, Book Title, “Section/Heading”)`
- Footnote citation: `[^short_ref]` with a footnote defining the full reference

Rules:
- Cite the **exact excerpt** that states the definition/formula/assumption (not a nearby discussion).
- If sources disagree, present the discrepancy and ask which convention to adopt.
- Never claim “current market practice today” without a dated authoritative source in the corpus.
- Avoid long verbatim quotes from textbooks; paraphrase in your own words. If you include a direct quote, keep it **under ~25 words**.

## Beginner-First Clarity (do not over-summarize)

- Prefer **rewriting for clarity** over deleting content. Keep enough detail for a newcomer to build intuition.
- Add **worked examples**, diagrams-in-words, and short “why this matters” notes where helpful.
- When removing content for correctness, replace it with either:
  - a corrected, sourced explanation, or
  - a clearly labeled placeholder: `NOT SURE: ...` (see below).

## Desk-Focused Orientation (middle office → trading desk)

This book is written for readers who may already understand basic finance but want to operate fluently in a **front-office / trading desk** environment.

- Keep (and add) **desk-reality explanations**: how a quote becomes cash, how P&L gets attributed (carry vs roll vs MTM), and how conventions create breaks.
- Prefer **practical framing**: “what traders mean when they say…”, “what product control checks…”, “what risk systems often assume…”.
- Avoid deleting desk callouts unless they are incorrect; instead **fix** them, add caveats, or mark `NOT SURE` where required.

## Continuous Improvement Pass

When working a batch of chapters:
- Do a full read-through and **fix** any errors in math, units, sign conventions, and timelines.
- Improve unclear explanations (especially where a middle-office reader might misunderstand “what the desk means”).
- Tighten examples and sanity checks; if you change a worked example, update the corresponding problem/solution if present.
- Remove legacy sections like `## Source Map` and any `Last Updated` lines; use `## References` instead.

## Uncertainty & External Lookup Policy

If something is unclear or you can’t retrieve strong evidence from `books_rag`:
1. **Try again** with better `books_rag` queries (narrow terms, alternative phrasing, larger limits).
2. If still unclear, **search online** for an authoritative primary source (e.g., ISDA/ARRC/CCP rulebooks, central bank publications, regulator docs, etc). Prefer dated sources.
3. If it’s still unclear or sources conflict, keep the topic but mark it inline as:
   - `NOT SURE: <precise question / ambiguity + what input is needed>`
   and (optionally) add a short “Inputs needed” bullet near the end of the chapter.

## Quality Bars

- **Clarity**: define notation before use; unit checks and sign conventions; state assumptions.
- **Practicality**: include “desk reality” failure modes (but only when evidence exists; otherwise request inputs).
- **Consistency**: reuse the same definitions across chapters (DV01 sign, clean/dirty, day counts, etc.).

## Skills (session-level instructions)

The following “skills” may be available in the Codex environment. Use them when the user request matches the description.

