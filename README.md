# Week 6 RAG Homework Starter

This starter app helps students practice Week 6 concepts from the RAG lecture:

- corpus preparation
- chunking with overlap
- simple retrieval
- answer generation with citations
- refusal behavior when evidence is missing

## Learning goals

By the end of this homework, a student pair should be able to:

1. Explain how chunking strategy affects retrieval quality.
2. Implement a retrieval pipeline that returns top-k relevant chunks.
3. Return answers with explicit citations to source chunks.
4. Refuse unsupported questions instead of hallucinating.
5. Show evidence for one supported and one unsupported question.

## Project structure

```text
rag-week-6-homework-starter/
  src/
    corpus.py
    chunking.py
    retrieval.py
    rag.py
    cli.py
  tests/
    test_corpus.py
    test_chunking.py
    test_retrieval.py
    test_rag.py
    test_cli.py
  docs/
    homework-checklist.md
  scripts/
    run_tests.sh
  evidence.md
  requirements.txt
```

## Setup

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

## Run tests

```bash
./scripts/run_tests.sh
```

## Run demo CLI

```bash
python -m src.cli --question "What is retrieval augmented generation?"
python -m src.cli --question "Who won the world cup in 1958?"
python -m src.cli --corpus-profile training-rag --question "How do citations improve trust?"
```

The CLI prints the chunking configuration, ranked chunk IDs, scores, full retrieved
text, answer, references, and refusal status. Chunk sizes and overlap are measured
in **characters**, after whitespace normalization. Defaults in `src/chunking.py`
are `chunk_size=220, overlap=40`; these keep the short default documents intact.
Chunk IDs identify a document and its zero-based chunk index within a configuration.
Record the configuration when citing evidence, since changing it changes chunk contents.

Compare the same question while changing only overlap:

```bash
python -m src.cli --question "How do chunk size and overlap improve retrieval quality?" --chunk-size 70 --overlap 0 --top-k 3
python -m src.cli --question "How do chunk size and overlap improve retrieval quality?" --chunk-size 70 --overlap 20 --top-k 3
```

Retrieval removes common function words and scores distinct content words using
`2 * shared_terms / (query_terms + chunk_terms)` (Dice similarity). It returns up to
`k` positive-scoring chunks in descending order, preserving corpus order on ties.
The default score cutoff is `0.01`. Repeated words cannot inflate relevance.

Answers quote source text and cite only ranked chunks that add previously uncovered
query terms. A separate References section identifies each cited source. The system
refuses if any query content term is absent from the retrieved context, retaining
that context in the CLI for inspection. This catches both unrelated questions and
topic matches such as "Who invented retrieval augmented generation?".

This is a conservative lexical teaching baseline: exact word matching can refuse
supported paraphrases, and matching every term does not establish semantic support
for a question. It does not validate relationships, dates, or negation. Extractive
answers avoid generating additional facts; inspect the cited evidence when reviewing
grounding. Character windows can split words and sentences, as the overlap experiment
demonstrates. A larger corpus would need further tuning.

See `evidence.md` for captured commands, outputs, ranking calculations, and tests.

If a copied `.venv` points to a missing Python installation, create a fresh environment
in a different directory, then install the pinned requirements:

```bash
python3 -m venv .venv-local
source .venv-local/bin/activate
python -m pip install -r requirements.txt
./scripts/run_tests.sh
```

## Extension: sentence chunking and stemming

Two opt-in options improve on the baseline; defaults are unchanged.

- `--strategy sentence` groups whole sentences into chunks of at most `--chunk-size`
  characters. `--sentence-overlap N` repeats the last N sentences of the previous chunk
  (default 0). Sentences longer than the chunk size are split at word boundaries.
- `--stem` applies a light suffix stemmer (`helps`, `helping` -> `help`) to both query
  and chunk terms, for scoring, refusal checks, and citations alike.

```bash
python -m src.cli --question "How do chunk size and overlap improve retrieval quality?" --strategy sentence --chunk-size 70 --top-k 3
python -m src.cli --question "What does metadata help with?" --stem --top-k 3
```

Limitations: abbreviations such as "e.g." are treated as sentence ends; the stemmer does
not handle irregular forms (ran/run) and can over-stem; matching remains lexical, so
synonyms are not recognised.

## Corpus profiles

- `default`: compact baseline corpus for Week 6 fundamentals.
- `training-rag`: profile mirroring examples with focus on citations, references, and LLM-as-a-judge framing.

## Homework tasks for students

1. **Outcome 1 (chunking impact):** tune chunking parameters in `src/chunking.py` and compare retrieval quality for the same query.
2. **Outcome 2 (top-k retrieval):** verify `src/retrieval.py` returns correctly ordered top-k relevant chunks.
3. **Outcome 3 (citations):** ensure `src/rag.py` returns grounded answers with explicit citations to chunk IDs.
4. **Outcome 4 (refusal):** ensure unsupported questions trigger refusal instead of hallucinated answers.
5. **Outcome 5 (evidence):** document one supported and one unsupported query with full retrieval and output evidence.

## Recommended evidence template (`evidence.md`)

Use this exact structure for easy review:

1. **Outcome 1: Chunking impact**
  - Config A:
  - Config B:
  - Retrieval difference summary:
2. **Outcome 2: Top-k retrieval**
  - Query:
  - Top-k output:
  - Why ranking is correct:
3. **Outcome 3: Citation quality**
  - Supported query:
  - Answer with citations:
  - Retrieved chunk IDs:
4. **Outcome 4: Refusal behavior**
  - Unsupported query:
  - Retrieved context:
  - Refusal output:
5. **Outcome 5: Final evidence pair**
  - Supported case command + output
  - Unsupported case command + output

Use `docs/homework-checklist.md` as submission guidance.
