# Fixed Income Book — Agent Instructions (Evidence-First)

This repo is a fixed income / rates / credit textbook written as markdown chapters under `chapters/`.

Don't let perfect be the enemy of the good. Move efficiently through this work while still not compromising on quality of the content generated.

There are multiple agents working on this project. 
Multiple agents may work **in parallel** on different chapters/appendices.
- Only edit the chapter/appendix you are assigned (plus explicitly-required shared artifacts).
- Do not revert/“clean up” other agents’ changes (even if you disagree).
- Do not run `git restore`, `git checkout -- <file>`, `git reset`, or other rollback commands.
- Do not commit.
- `refactor_plan/` is deliberately gitignored (but still used); do not change `.gitignore`.
- Do not change the ToC structure in `CONTENTS.md` unless explicitly instructed.

## Mission

- Produce **publication-ready** chapters with **grounded correctness**.
- Prefer grounding **non-trivial claims** (definitions, conventions, algorithm steps, “desk reality” practices) in evidence from `books_rag`. When needed, supplement with authoritative online primary sources (ISDA/ARRC/CCP rulebooks, central banks, regulators).
- Use your own knowledge to improve **clarity and intuition**, but avoid asserting time-varying market practice without an authoritative source. If something remains unclear, mark it inline as `NOT SURE: ...` with the precise ambiguity.

## Quick Chapter Workflow (Default)

When you are assigned a chapter/appendix, follow `refactor_plan/overall_plan.md` end-to-end. Typical loop:

1. Skim `CONTENTS.md` notes + target chapter to find gaps.
2. Run initial lint (`python refactor_plan/chapter_lint.py --paths <TARGET_FILE>`) and record the output.
3. Create the per-target plan file from `refactor_plan/chapter_plan_template.md` (scope boundaries + conventions + evidence plan + cross-links).
4. Evidence-first discovery: 5–10 targeted `rg` queries in the book corpus + 2–3 `books_rag.answer()` variants + 1 `search_structured()` + `fetch_chunk()` for any definition/formula/convention you will rely on.
5. Edit for depth + correctness: learning objectives, intro cross-links, quote → cashflows → PV → risk, ≥1 worked example, ≥1 Pitfall + ≥1 Desk Reality, end-matter headings/order per `refactor_plan/end_matter_templates.md`.
6. Run `books_rag.verify_draft()` on **claim packets** (≤30 sentences) and rewrite anchors or mark `NOT SURE: ...`.
7. Re-run lint; fix remaining issues; do LaTeX hygiene.
8. Append-only shared logs as required (e.g., `refactor_plan/book_usage_log.md`, `refactor_plan/concept_coverage_matrix.md`).

## Depth & Intuition (Keep the Chapter “Teachable”)

Evidence-first should not translate into “delete anything you can’t cite.” The goal is **grounded correctness + human intuition**.

Use an “evidence sandwich” pattern for important ideas:
1. **Anchor (evidence-backed):** 1–3 sentences stating the definition/formula/convention precisely.
2. **Expand (your explanation):** unpack the terms, units, and what is held fixed. Add a mental model, diagram-in-words, or replication story.
3. **Check (reader validation):** include at least one of: unit check, sign check, limiting case, or a small toy numeric example (clearly labeled hypothetical).

When removing/shortening unsupported material, replace it with one of:
- A **timeless mechanism-level** explanation (what must be true, what can vary, and why).
- A **worked toy example** that builds intuition without claiming time-varying “typical” levels.
- A precise `NOT SURE: ...` placeholder stating the missing input/source needed.

`books_rag.verify_draft()` can be strict about paraphrases. To avoid losing depth:
- Run it on **claim packets** (definitions/conventions/time-varying factual claims), not on pure intuition text.
- Keep the **anchor** sentences closest to the source excerpt; keep the **expand/check** text as reasoning and sanity checks.
See also: `refactor_plan/depth_intuition_playbook.md`.

## Evidence-First Rules (books_rag MCP)

Do not draft/replace **substantive** chapter content until you have collected evidence for the key claims you plan to make (definitions, pricing equations, conventions, risk measures). Use these tools:

- `answer(query, alpha=0.3, rerank=true, limit=8)` → evidence pack (chunks + neighbor expansion)
- `search_structured(query, ..., limit=20–50)` → targeted hunting (filters, path scoping)
- `fetch_chunk(uuid)` → pull full context for quoting/formulas
- `verify_draft(text, support_threshold=3.0, max_sentences=30)` → per-sentence support checking
- `resolve_doc(path, ...)` → optional scoping for internal docs

### Local Search Is Allowed (and Encouraged When Retrieval Misses)

Evidence-first does **not** mean “books_rag-only”.

When `books_rag` retrieval clusters or misses a key definition/convention:
- Use local text search (`rg`, `grep`, filename search, etc.) against the **book corpus** and existing chapters to find the right terms/headings:
  - Book corpus root: `books/`.
  - Chapters live under `chapters/`.
  - If `rg` seems to skip the corpus because it is ignored, use `rg -uuu 'pattern' books/`.
- Then use what you found (exact wording, headings, file paths) to re-query `books_rag.search_structured()` (often with `source_path_like`) and/or `books_rag.fetch_chunk()` for the specific excerpt you rely on.

Local search is a **discovery** tool; publication output still follows house style:
- No inline citations in prose; sources go only in `## References`.
- If evidence remains missing/conflicting after local search, mark `NOT SURE: ...` with the precise ambiguity.

### WebSearch or your own knowledge
When both books_rag and local search fail to provide sufficient evidence, use WebSearch or your own knowledge to find the missing information. Do not speculate when you do this. Use your knowledge when you have a high confidence that the information is correct.

## Speed & Scope Control (Keep It High-Quality, Faster)

This project has a lot of chapters; perfectionism is the enemy of throughput.
- Target pace: **~45 minutes per chapter** (unless the user explicitly asks for a deep dive).
- Timebox evidence collection and drafting. Prefer a “good, supported, consistent” chapter over exhaustive coverage.
- Only rewrite what you need to fix correctness/consistency/structure; avoid stylistic rewrites of already-correct paragraphs.
- If you cannot quickly find strong evidence for a convention-sensitive claim, write the best supported version you can and add a precise `NOT SURE: ...` marker rather than spending hours searching.
- Run `books_rag.verify_draft()` on **edited sections only**, iterate 1–2 times, then move on.

Suggested 45-minute split (guideline):
- 5 min: read `CONTENTS.md` notes + skim target chapter for gaps
- 10–15 min: evidence collection (local `rg` + 2–3 `answer()` + 1–2 `search_structured()`)
- 20 min: edits (structure + one worked example + pitfalls/desk reality + end-matter)
- 5–10 min: QA (`verify_draft` on edited parts + lint) + append-only shared logs

## Repo Layout

- `CONTENTS.md`: table of contents + chapter-by-chapter improvement notes (use as the improvement plan source).
- `chapters/`: one chapter per file (e.g., `chapters/chapter_18_ois_discounting_curve.md`).
- `books/`: local reference corpus (may be gitignored); do **not** edit or commit book contents.


### Books RAG Improvement Mode (Do *Not* Change Retrieval)

Sometimes retrieval clusters or misses key conventions. Do not try to “fix” `books_rag` or its index as part of chapter work.
Instead:
1. Vary query phrasing (synonyms, instrument name, convention name, “definition”, “pricing equation”, “day count”, etc.).
2. Force source diversity using `search_structured()` with `source_path_like` filters.
3. Use local discovery (`rg`) against `books/` (or `/home/usman/work/books/books` if this repo has no local corpus) to find candidate headings, then pull those via `books_rag`.
4. If evidence is still weak/conflicting: decide a convention explicitly for the chapter (and align with `refactor_plan/notation_registry.md`), or mark `NOT SURE` with the missing input.

### References (required)

No inline citations in chapter prose. Put sources **only** in the chapter’s `## References` section near the end.

Aim for **~5–8 references per chapter** (readable; no UUIDs in the prose).

Rules:
- Cite the **exact excerpt** that states the definition/formula/assumption (not a nearby discussion).
- If sources disagree, present the discrepancy and ask which convention to adopt.
- Never claim “current market practice today” without a dated authoritative source in the corpus.
- Avoid long verbatim quotes from textbooks; paraphrase in your own words. If you include a direct quote, keep it **under ~25 words**.

### Writing Voice (Readable, Not Citation-y)

Write in a neutral textbook voice. Use sources as *support*, not as narration.
- Avoid prose like “Tuckman says…” / “Hull states…”. Instead: state the fact cleanly, then include the supporting sources in `## References`.
- Only name authors in the body when you are explicitly comparing conflicting conventions or quoting a short excerpt.

## Parallel-Work Hygiene (Shared Files)

Some `refactor_plan/` files are shared across agents. To reduce merge conflicts:
- Treat `refactor_plan/book_usage_log.md` as **append-only** (add new rows at the end; never edit prior entries).
- Treat `refactor_plan/concept_coverage_matrix.md` as **append-only** (add new rows/entries at the end; never edit prior entries).
- If you hit a conflict, prefer keeping **both** agents’ appended entries rather than “choosing one”.

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
2. If still unclear, **search locally** in the book corpus with `rg`/`grep` (and skim the relevant file sections) to find the exact wording/headings to re-query in `books_rag`.
3. If still unclear, **search online** for an authoritative primary source (e.g., ISDA/ARRC/CCP rulebooks, central bank publications, regulator docs, etc). Prefer dated sources.
4. If it’s still unclear or sources conflict, use your knowledge to fill in the gaps but do not speculate. If you are not sure about something, mark it as `NOT SURE: ...` with the precise ambiguity.

## Quality Bars

- **Clarity**: define notation before use; unit checks and sign conventions; state assumptions.
- **Practicality**: include “desk reality” failure modes (but only when evidence exists; otherwise request inputs).
- **Consistency**: reuse the same definitions across chapters (DV01 sign, clean/dirty, day counts, etc.). When introducing DV01/PV01/CS01, always state bump object, bump size, units, and sign.

## GitHub Math Rendering Guardrails (Mandatory)

- Use inline math as `$...$`; use display math as `$$ ... $$` with blank lines around blocks.
- Avoid raw `<` and `>` in math; use `\lt`, `\gt`, `\le`, `\ge`.
- Do not use `\operatorname{...}`; use `\mathrm{...}` or plain symbols.
- Avoid risky macro+subscript forms like `\text{X}_{i}`; prefer `X_i` or plain identifiers.
- Avoid escaped dollar `\$` inside math expressions; move currency outside math or write `USD`.
- Avoid math inside markdown emphasis like `*...$...$...*`.
- Avoid fragile inline math in headings; prefer plain-text heading labels.
- Prefer stable symbols in prose where possible (use backticks for identifiers when math is not necessary).
- Be careful with math inside blockquotes, lists, and tables; if renderer is fragile, rewrite to inline/plain text.
- After every file edit, run:
  - `python3 refactor_plan/github_math_escape_validate.py --paths <file> --max-errors 300`
  - `.venv-github-render/bin/python refactor_plan/mathjax_render_check.py --paths <file> --max-errors 300`
- After push, run:
  - `python3 refactor_plan/github_api_render_validate.py --paths <file> --ref main --max-errors 300`
- A file is only done when `github_api_render_validate.py` returns `OK`.

## Skills (session-level instructions)

The following “skills” may be available in the Codex environment. Use them when the user request matches the description.
