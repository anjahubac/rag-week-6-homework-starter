# Week 6 RAG homework evidence

Recorded on 2026-10-14. All CLI output below was captured from actual runs. Use Python 3.9+ and install requirements.txt in an active virtual environment for tests.

Pair accountability:

- Members: Anja Hubac, Aleksa Cevizovic.
- Contributions (agreed equal split of responsibility):
  - Anja Hubac: Outcome 1 (chunking comparison, Configs A-C), Outcome 2 (top-k retrieval, Dice scoring, and ranking calculation), the sentence chunking extension, and local environment setup and test runs.
  - Aleksa Cevizovic: Outcome 3 (citations and References section), Outcome 4 (refusal coverage check), Outcome 5 (final evidence pair), and the stemming extension.
- Demo lead: none this week; both members are absent from the session.
- AI use: the implementation changes, tests, and evidence were drafted with AI coding assistants (OpenAI Codex, then Claude Code). Every CLI output in this file was captured from actual runs and re-checked against the current code, and the full test suite was run locally. Both members reviewed and verified the work.

1. Outcome 1: Chunking impact

    - Config A: default corpus; chunk_size=70 characters, overlap=0, top_k=3.

    ```bash
    python3 -m src.cli --question 'How do chunk size and overlap improve retrieval quality?' --chunk-size 70 --overlap 0 --top-k 3
    ```

    ```text
    Question:
    How do chunk size and overlap improve retrieval quality?

    Corpus profile: default
    Chunking: chunk_size=70, overlap=0 (characters)
    Top-k: 3; corpus chunks: 9

    Retrieved context (descending score):
    1. doc-2-chunk-1 | score=0.400000 | Chunking and Metadata
    lap improve retrieval quality. Metadata helps filtering and source tra
    2. doc-2-chunk-0 | score=0.250000 | Chunking and Metadata
    Chunking splits documents into smaller parts. Good chunk size and over
    3. doc-1-chunk-0 | score=0.181818 | RAG Fundamentals
    Retrieval augmented generation combines retrieval with text generation

    Answer:
    I do not have enough grounded evidence in the provided corpus to answer that question.

    Refused: True
    Citations: None
    ```

    - Config B: same corpus, question, chunk_size=70, and top_k=3; overlap=20 characters.

    ```bash
    python3 -m src.cli --question 'How do chunk size and overlap improve retrieval quality?' --chunk-size 70 --overlap 20 --top-k 3
    ```

    ```text
    Question:
    How do chunk size and overlap improve retrieval quality?

    Corpus profile: default
    Chunking: chunk_size=70, overlap=20 (characters)
    Top-k: 3; corpus chunks: 9

    Retrieved context (descending score):
    1. doc-2-chunk-1 | score=0.800000 | Chunking and Metadata
    chunk size and overlap improve retrieval quality. Metadata helps filt
    2. doc-2-chunk-0 | score=0.250000 | Chunking and Metadata
    Chunking splits documents into smaller parts. Good chunk size and over
    3. doc-1-chunk-0 | score=0.181818 | RAG Fundamentals
    Retrieval augmented generation combines retrieval with text generation

    Answer:
    Grounded response based on retrieved context:
    [doc-2-chunk-1] chunk size and overlap improve retrieval quality. Metadata helps filt

    References:
    [doc-2-chunk-1] Chunking and Metadata (doc-2)

    Refused: False
    Citations: doc-2-chunk-1
    ```

    - Config C (extension): same corpus, question, chunk_size=70, and top_k=3; sentence strategy with sentence_overlap=0.

    ```bash
    python3 -m src.cli --question 'How do chunk size and overlap improve retrieval quality?' --strategy sentence --chunk-size 70 --top-k 3
    ```

    ```text
    Question:
    How do chunk size and overlap improve retrieval quality?

    Corpus profile: default
    Chunking: strategy=sentence, chunk_size=70 (characters), sentence_overlap=0 (sentences)
    Top-k: 3; corpus chunks: 10

    Retrieved context (descending score):
    1. doc-2-chunk-1 | score=0.923077 | Chunking and Metadata
    Good chunk size and overlap improve retrieval quality.
    2. doc-1-chunk-0 | score=0.181818 | RAG Fundamentals
    Retrieval augmented generation combines retrieval with text

    Answer:
    Grounded response based on retrieved context:
    [doc-2-chunk-1] Good chunk size and overlap improve retrieval quality.

    References:
    [doc-2-chunk-1] Chunking and Metadata (doc-2)

    Refused: False
    Citations: doc-2-chunk-1
    ```

    - Retrieval difference summary: both configurations create 9 chunks. Without overlap, the word overlap becomes over in doc-2-chunk-0 and lap in doc-2-chunk-1. The top score is 0.400000 and the content-term coverage check refuses because overlap is missing. With 20 characters of overlap, doc-2-chunk-1 contains all six query content terms (chunk, size, overlap, improve, retrieval, quality), scores 0.800000, and supplies a cited answer. Only overlap changed, so this experiment isolates boundary preservation. Overlap repeats text and can increase chunk count on longer documents; it does not guarantee complete sentences. The final defaults in src/chunking.py are 220/40 to retain complete statements in this tiny corpus. That configuration creates 3 chunks, one per document; overlap is unused when a document fits in one chunk. These results do not establish optimal parameters for larger corpora.

    Config C keeps chunk_size=70 but cuts only at sentence ends. The best chunk is exactly the sentence "Good chunk size and overlap improve retrieval quality.", which contains all six query content terms and few others, so the score rises to 0.923077 (vs 0.800000 in B) and the cited text starts and ends cleanly. Config B's best chunk began mid-sentence ("chunk size and overlap...") and ended mid-word ("filt"). Sentence chunking creates 10 chunks: three corpus sentences are longer than 70 characters, so they are split at word boundaries (for example doc-1-chunk-0 ends at "with text" and doc-1-chunk-1 is "generation."). No word is cut, but a long sentence can still span two chunks. Only two chunks score above zero, so top_k=3 returns two.

2. Outcome 2: Top-k retrieval

    - Query: retrieval evidence. Default corpus, chunk_size=220, overlap=40.
    - Top-k output: actual commands and full output for both k=2 and k=3.

    ```bash
    python3 -m src.cli --question 'retrieval evidence' --chunk-size 220 --overlap 40 --top-k 2
    ```

    ```text
    Question:
    retrieval evidence

    Corpus profile: default
    Chunking: chunk_size=220, overlap=40 (characters)
    Top-k: 2; corpus chunks: 3

    Retrieved context (descending score):
    1. doc-1-chunk-0 | score=0.250000 | RAG Fundamentals
    Retrieval augmented generation combines retrieval with text generation. The model should answer using retrieved evidence rather than unsupported assumptions.
    2. doc-3-chunk-0 | score=0.125000 | Refusal Behavior
    If retrieved context does not support a claim, the system should refuse and ask for better evidence. This reduces hallucinations and increases trust.

    Answer:
    Grounded response based on retrieved context:
    [doc-1-chunk-0] Retrieval augmented generation combines retrieval with text generation. The model should answer using retrieved evidence rather than unsupported assumptions.

    References:
    [doc-1-chunk-0] RAG Fundamentals (doc-1)

    Refused: False
    Citations: doc-1-chunk-0
    ```

    ```bash
    python3 -m src.cli --question 'retrieval evidence' --chunk-size 220 --overlap 40 --top-k 3
    ```

    ```text
    Question:
    retrieval evidence

    Corpus profile: default
    Chunking: chunk_size=220, overlap=40 (characters)
    Top-k: 3; corpus chunks: 3

    Retrieved context (descending score):
    1. doc-1-chunk-0 | score=0.250000 | RAG Fundamentals
    Retrieval augmented generation combines retrieval with text generation. The model should answer using retrieved evidence rather than unsupported assumptions.
    2. doc-3-chunk-0 | score=0.125000 | Refusal Behavior
    If retrieved context does not support a claim, the system should refuse and ask for better evidence. This reduces hallucinations and increases trust.
    3. doc-2-chunk-0 | score=0.100000 | Chunking and Metadata
    Chunking splits documents into smaller parts. Good chunk size and overlap improve retrieval quality. Metadata helps filtering and source traceability.

    Answer:
    Grounded response based on retrieved context:
    [doc-1-chunk-0] Retrieval augmented generation combines retrieval with text generation. The model should answer using retrieved evidence rather than unsupported assumptions.

    References:
    [doc-1-chunk-0] RAG Fundamentals (doc-1)

    Refused: False
    Citations: doc-1-chunk-0
    ```

    - Why ranking is correct: lowercase alphanumeric tokens are deduplicated and common function words removed. The score is 2 times the number of shared content terms divided by the sum of query and chunk content-term counts (Dice similarity). The query has 2 terms.

    | Chunk ID | Shared terms | Chunk terms | Calculation | Score |
    | --- | --- | --- | --- | --- |
    | doc-1-chunk-0 | 2 | 14 | 4 / (2 + 14) | 0.250000 |
    | doc-3-chunk-0 | 1 | 14 | 2 / (2 + 14) | 0.125000 |
    | doc-2-chunk-0 | 1 | 18 | 2 / (2 + 18) | 0.100000 |

    These scores give doc-1, doc-3, doc-2 in descending order; k=2 is exactly the first two entries of k=3. Chunks below the default 0.01 cutoff or with zero score are excluded, so fewer than k results are possible. Ties preserve input corpus order. Tests also verify exact scores on an independently specified fixture, repeat-word invariance, filtering, and invalid parameters.

3. Outcome 3: Citation quality

    - Supported query: What is retrieval augmented generation?
    - Answer with citations:

    ```text
    Grounded response based on retrieved context:
    [doc-1-chunk-0] Retrieval augmented generation combines retrieval with text generation. The model should answer using retrieved evidence rather than unsupported assumptions.

    References:
    [doc-1-chunk-0] RAG Fundamentals (doc-1)
    ```

    - Retrieved chunk IDs: doc-1-chunk-0, doc-2-chunk-0.
    The answer cites doc-1-chunk-0, which contains the definition and covers all three query content terms. doc-2-chunk-0 matches only retrieval and adds no uncovered query terms, so it is shown in retrieval evidence but excluded from the answer and references. The answer quotes source text exactly; its reference lists the chunk ID, title, and document ID.

4. Outcome 4: Refusal behavior

    - Unsupported query: Who invented retrieval augmented generation?
    - Retrieved context:

    ```text
    doc-1-chunk-0 | score=0.333333
    Retrieval augmented generation combines retrieval with text generation. The model should answer using retrieved evidence rather than unsupported assumptions.
    doc-2-chunk-0 | score=0.090909
    Chunking splits documents into smaller parts. Good chunk size and overlap improve retrieval quality. Metadata helps filtering and source traceability.
    ```

    - Refusal output:

    ```text
    I do not have enough grounded evidence in the provided corpus to answer that question.
    Refused: True
    Citations: None
    ```

    The retrieved chunks describe RAG and chunking, but do not identify an inventor. The content term invented is absent; topical overlap alone therefore cannot pass the coverage check. The system keeps retrieval evidence visible and makes no inventor claim. The unrelated World Cup question is also tested and returns empty retrieval plus the same refusal. This is a conservative lexical guard, not semantic entailment: supported paraphrases can be refused, and shared words do not validate relationships, dates, or negation.

5. Outcome 5: Final evidence pair

    - Supported case command + output

    ```bash
    python3 -m src.cli --question 'What is retrieval augmented generation?' --chunk-size 220 --overlap 40 --top-k 2
    ```

    ```text
    Question:
    What is retrieval augmented generation?

    Corpus profile: default
    Chunking: chunk_size=220, overlap=40 (characters)
    Top-k: 2; corpus chunks: 3

    Retrieved context (descending score):
    1. doc-1-chunk-0 | score=0.352941 | RAG Fundamentals
    Retrieval augmented generation combines retrieval with text generation. The model should answer using retrieved evidence rather than unsupported assumptions.
    2. doc-2-chunk-0 | score=0.095238 | Chunking and Metadata
    Chunking splits documents into smaller parts. Good chunk size and overlap improve retrieval quality. Metadata helps filtering and source traceability.

    Answer:
    Grounded response based on retrieved context:
    [doc-1-chunk-0] Retrieval augmented generation combines retrieval with text generation. The model should answer using retrieved evidence rather than unsupported assumptions.

    References:
    [doc-1-chunk-0] RAG Fundamentals (doc-1)

    Refused: False
    Citations: doc-1-chunk-0
    ```

    - Unsupported case command + output

    ```bash
    python3 -m src.cli --question 'Who invented retrieval augmented generation?' --chunk-size 220 --overlap 40 --top-k 2
    ```

    ```text
    Question:
    Who invented retrieval augmented generation?

    Corpus profile: default
    Chunking: chunk_size=220, overlap=40 (characters)
    Top-k: 2; corpus chunks: 3

    Retrieved context (descending score):
    1. doc-1-chunk-0 | score=0.333333 | RAG Fundamentals
    Retrieval augmented generation combines retrieval with text generation. The model should answer using retrieved evidence rather than unsupported assumptions.
    2. doc-2-chunk-0 | score=0.090909 | Chunking and Metadata
    Chunking splits documents into smaller parts. Good chunk size and overlap improve retrieval quality. Metadata helps filtering and source traceability.

    Answer:
    I do not have enough grounded evidence in the provided corpus to answer that question.

    Refused: True
    Citations: None
    ```

    - Test output snapshot: the provided test script was run with pytest==8.3.3 in a fresh Python 3.9 environment (.venv-local) because the copied .venv points to a missing Python 3.12 executable. After activating your own working environment, run:

    ```bash
    ./scripts/run_tests.sh
    ```

    ```text
    .........................................................                [100%]
    57 passed in 0.13s
    ```

6. Extension (checklist H): sentence chunking and stemming

    Both features are opt-in (--strategy sentence, --stem). Default behaviour is unchanged: every command in Outcomes 1-5 above was re-run after the extension and produced identical output. The extension adds 28 tests (29 before, 57 after).

    - Sentence chunking (src/chunking.py): split_text_by_sentence groups whole sentences up to chunk_size characters. Overlap is measured in whole sentences (--sentence-overlap) and is skipped when the carried sentence would leave no room for the next one. A sentence longer than chunk_size is split at word boundaries; only a single word longer than chunk_size is cut by characters. Evidence: Config C in Outcome 1.
    - Stemming (src/retrieval.py): stem() applies light suffix rules (-ies to -y, -s, -ing, -ed, final -e), so helps/help, chunks/chunking/chunk, and retrieve/retrieved match. Stemming happens inside content_tokens, so retrieval scoring, the refusal coverage check, and citation selection all compare terms the same way.

    Without stemming, the question uses "help" but the corpus says "helps", so the coverage check refuses a supported question:

    ```bash
    python3 -m src.cli --question 'What does metadata help with?' --top-k 3
    ```

    ```text
    Question:
    What does metadata help with?

    Corpus profile: default
    Chunking: chunk_size=220, overlap=40 (characters)
    Top-k: 3; corpus chunks: 3

    Retrieved context (descending score):
    1. doc-2-chunk-0 | score=0.100000 | Chunking and Metadata
    Chunking splits documents into smaller parts. Good chunk size and overlap improve retrieval quality. Metadata helps filtering and source traceability.

    Answer:
    I do not have enough grounded evidence in the provided corpus to answer that question.

    Refused: True
    Citations: None
    ```

    With stemming, help and helps share the stem help, the score rises from 0.100000 to 0.210526, and the answer is cited:

    ```bash
    python3 -m src.cli --question 'What does metadata help with?' --stem --top-k 3
    ```

    ```text
    Question:
    What does metadata help with?

    Corpus profile: default
    Chunking: chunk_size=220, overlap=40 (characters)
    Matching: stemming=on
    Top-k: 3; corpus chunks: 3

    Retrieved context (descending score):
    1. doc-2-chunk-0 | score=0.210526 | Chunking and Metadata
    Chunking splits documents into smaller parts. Good chunk size and overlap improve retrieval quality. Metadata helps filtering and source traceability.

    Answer:
    Grounded response based on retrieved context:
    [doc-2-chunk-0] Chunking splits documents into smaller parts. Good chunk size and overlap improve retrieval quality. Metadata helps filtering and source traceability.

    References:
    [doc-2-chunk-0] Chunking and Metadata (doc-2)

    Refused: False
    Citations: doc-2-chunk-0
    ```

    Stemming does not weaken refusal. invented stems to invent, which still appears nowhere in the corpus:

    ```bash
    python3 -m src.cli --question 'Who invented retrieval augmented generation?' --stem --top-k 2
    ```

    ```text
    Question:
    Who invented retrieval augmented generation?

    Corpus profile: default
    Chunking: chunk_size=220, overlap=40 (characters)
    Matching: stemming=on
    Top-k: 2; corpus chunks: 3

    Retrieved context (descending score):
    1. doc-1-chunk-0 | score=0.333333 | RAG Fundamentals
    Retrieval augmented generation combines retrieval with text generation. The model should answer using retrieved evidence rather than unsupported assumptions.
    2. doc-2-chunk-0 | score=0.095238 | Chunking and Metadata
    Chunking splits documents into smaller parts. Good chunk size and overlap improve retrieval quality. Metadata helps filtering and source traceability.

    Answer:
    I do not have enough grounded evidence in the provided corpus to answer that question.

    Refused: True
    Citations: None
    ```

    - Limitations: sentence detection treats abbreviations such as "e.g." as sentence ends (the corpus has none). The stemmer is rule-based: irregular forms (ran/run) do not match and some words are over-stemmed (retrieve becomes retriev). Matching is still lexical after stemming, so synonyms and paraphrases are not recognised.
