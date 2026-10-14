# Session handoff: Week 6 RAG homework

Last updated: 2026-10-14. Purpose: start a new chat session with full context, and use as the
factual source for the weekly reports of Anja Hubac and Aleksa Cevizovic.

> For a new session: read this file first, then `evidence.md` and `docs/homework-checklist.md`.
> The weekly report must not invent work, reflections, or AI use; ask the learner for anything
> not recorded here.

## 1. Project at a glance

- Course: SITA AI Bootcamp 2026, Week 6 (RAG). Pair work: Anja Hubac and Aleksa Cevizovic.
- Assignment: `Week 6 RAG Homework Starter.pdf` (on the Desktop); same text as `README.md`
  "Homework tasks" plus `docs/homework-checklist.md`.
- Stack: Python 3.9.6, standard library only; `pytest==8.3.3` for tests. No LLM or API calls:
  retrieval is lexical and answers quote the retrieved chunks.
- Pipeline: `src/corpus.py` (documents) -> `src/chunking.py` (chunks) -> `src/retrieval.py`
  (scoring, top-k) -> `src/rag.py` (answer with citations or refusal) -> `src/cli.py` (command).

### How to run

```bash
cd ~/Desktop/rag-week-6-homework-starter
source .venv-local/bin/activate
./scripts/run_tests.sh                     # 57 passed
python -m src.cli --question "What is retrieval augmented generation?"
python -m src.cli --question "What does metadata help with?" --stem --top-k 3
python -m src.cli --question "How do chunk size and overlap improve retrieval quality?" --strategy sentence --chunk-size 70 --top-k 3
```

The original `.venv` is broken (points to a missing Python 3.12); use `.venv-local`.

## 2. Current status

- All 5 outcomes documented in `evidence.md`, plus section 6 (extension). Every CLI output in
  it was re-run against the current code and matches exactly.
- Tests: 57 passed (29 after Codex, 28 added by Claude Code).
- Checklist: 22 of 23 items ticked. Unticked: "designated demo lead is recorded". Both members
  are absent this week, so `evidence.md` says there is no demo lead.
- Pair accountability in `evidence.md`: names, agreed equal split of responsibility, demo-lead
  note, AI-use statement, and "Both members reviewed and verified the work" (Aleksa confirmed).
- Not a Git project in practice: an empty `.git` was created at 14:17 with no commits or remote.

## 3. Timeline of work

| When (2026-10-14) | Who / tool | What |
|---|---|---|
| Before (files dated 2026-10-07) | Course | Original starter files received |
| 10:01-10:33 | Anja + OpenAI Codex | Fixed retrieval, refusal, citations, CLI; added tests; wrote `evidence.md` |
| Afternoon | Anja + Claude Code | Explanation of RAG, environment setup, Codex audit, sentence chunking + stemming extension, docs, checklist |

## 4. What OpenAI Codex did (session 10:01-10:33)

Source: Codex session log in `~/.codex/sessions/2026/10/14/`.

Problems it found in the starter:
- It answered "Who won the world cup in 1958?" because common words ("the", "in") matched.
- Scoring counted repeated words, so repetition inflated relevance.
- Citations listed every retrieved chunk, even irrelevant ones; there was no References section.
- The CLI printed only the answer, not the retrieved chunks or scores needed for evidence.

Changes:
- `src/retrieval.py`: stop-word list, `content_tokens()`, Dice similarity
  `2 * shared / (query_terms + chunk_terms)`, `min_score` validation, zero scores dropped,
  ties keep corpus order.
- `src/rag.py`: refusal if any query content term is missing from the retrieved context
  (catches "Who invented RAG?"); citations only for chunks that add uncovered query terms;
  References section; retrieved chunks returned for display.
- `src/chunking.py`: defaults moved to constants, changed from 140/30 to 220/40 characters.
- `src/cli.py`: prints chunking settings, top-k, each retrieved chunk with score and text.
- Tests: 9 -> 29 (new `tests/test_cli.py`; more in chunking, retrieval, rag tests).
- `README.md`: explanation of scoring, refusal, limitations, venv workaround.
- `evidence.md`: created with Outcomes 1-5 from real runs. Key Outcome 1 finding: at
  chunk_size=70, overlap=0 splits "overlap" into "over"/"lap" and the system refuses;
  overlap=20 keeps the word whole and it answers (score 0.40 -> 0.80).

## 5. What Claude Code did (this session)

1. Explained RAG and the project file by file (corpus, chunking, retrieval, scoring, refusal).
2. Set up `.venv-local`, installed requirements, ran tests (29 passed).
3. Reconstructed exactly what Codex changed, from the Codex session log (no Git history).
4. Evaluated the chunking strategy with an experiment: fixed character windows cut words and
   sentences; one-sentence chunks scored highest (0.92) and gave clean citations. Also found
   that "What does metadata help with?" is refused under every chunking setup because
   "help" does not match "helps", which is a matching problem, not a chunking one.
5. Planned and implemented the extension (checklist H), both opt-in so defaults are unchanged:
   - Sentence chunking (`src/chunking.py`): `split_sentences`, `split_text_by_sentence`,
     `split_long_sentence`; `build_chunks(strategy="char"|"sentence", sentence_overlap=N)`.
     Overlap is in whole sentences. A first version cut an over-long sentence by characters
     and produced a chunk containing only "."; a failing test caught it, and long sentences
     are now split at word boundaries.
   - Stemming (`src/retrieval.py`): `stem()` with light suffix rules (-ies, -s, -ing, -ed,
     final -e); `use_stemming` flag through `content_tokens`, `lexical_score`,
     `retrieve_top_k`, and `build_grounded_answer`, so scoring, refusal, and citations match
     words the same way.
   - CLI flags: `--strategy {char,sentence}`, `--sentence-overlap N`, `--stem`.
   - 28 new tests (57 total).
6. Checked course documents for rules: keep the exact Outcome 1-5 structure, at least 3
   extension tests, refresh the test snapshot, Python 3.9, no new dependencies.
   `weekly-assignment-5.md` is a different (W05) project and does not apply.
7. Updated `evidence.md`: Config C in Outcome 1, section 6 Extension with real runs, test
   snapshot 57, pair accountability, AI-use statement.
8. Updated `README.md`: fixed project structure, added Extension section.
9. Ticked 22 of 23 items in `docs/homework-checklist.md` after verifying each one.

Results of the extension:

| Case | Before | After |
|---|---|---|
| Chunk-size question, chunk_size=70 | 0.80, cited chunk starts mid-sentence | 0.92 with `--strategy sentence`, clean sentence cited |
| "What does metadata help with?" | refused ("help" vs "helps") | answered with `--stem`, cites doc-2-chunk-0 |
| "Who invented RAG?" | refused | still refused with `--stem` |

Known limitations (documented): abbreviations such as "e.g." count as sentence ends; the
stemmer misses irregular forms (ran/run) and can over-stem; matching is still lexical, so
synonyms and paraphrases are not recognised.

## 6. Pair split (as recorded in evidence.md)

Agreed equal split of responsibility (who owns and can explain each area, not who typed it):

- Anja Hubac: Outcome 1 (chunking, Configs A-C), Outcome 2 (top-k, Dice scoring, ranking
  table), sentence chunking extension, environment setup and test runs.
- Aleksa Cevizovic: Outcome 3 (citations, References), Outcome 4 (refusal check), Outcome 5
  (final evidence pair), stemming extension.

Facts recorded so far: Anja ran the Codex and Claude Code sessions on her machine. Aleksa
confirmed reviewing and verifying the work. Nothing else about individual activity is recorded.

## 7. For the weekly reports

Facts available from this file: task, what was built, verification (57 tests, outputs
re-checked), limitations, AI use (Codex, then Claude Code), the agreed split, demo-lead status.

Each member still needs to supply in their own words:
- what they personally did, reviewed, or learned this week;
- a difficulty or limitation they met and what they learned from it;
- how they used and checked the AI output;
- whether they need help, and one next step.

The `student-weekly-report` skill (Serbian AI Bootcamp weekly report) can draft each report
from this file plus the member's own notes.

## 8. Open items

- Demo lead: none designated (both absent). Name one only if the course requires it.
- Optional: commit the work to Git (the `.git` folder is empty).
