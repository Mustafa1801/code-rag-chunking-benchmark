# Code-RAG Chunking Benchmark


> **Decision:** Use this benchmark when comparing complete code-retrieval pipelines on one pinned corpus and question set; skip it if you need a universal chunk-size winner without running retrieval experiments.
>
> **For:** code-search engineers, RAG evaluators, and repository intelligence teams  ·  **Not for:** teams looking for fabricated benchmark results or a model-only comparison

A chunker cannot be judged in isolation from retrieval. This benchmark compares three complete code-retrieval pipelines on the same pinned corpus and question set, then reports retrieval quality, latency, and index cost separately.

It is a **benchmark design and fillable question set**, not a claim that one strategy has already won.

## What is actually being compared

| Pipeline | Chunk/index unit | Candidate generation | Optional relationship signal | Hypothesis to test—not a fact |
|---|---|---|---|---|
| P1 file/window | Fixed token windows within files, with path/language metadata | Embedding vector search (for example Qdrant) | none | Broad natural-language questions may retrieve enough local context, but boundaries may split symbols |
| P2 symbol-aware hybrid | tree-sitter function/class/module chunks; bounded parent context | Lexical/trigram + embedding fusion | symbol name/path boosts | Symbol boundaries may improve definition questions while hurting cross-symbol tasks |
| P3 graph-enriched | P2 chunks | Hybrid retrieval | SCIP definition/reference neighbors appended under a fixed budget | Cross-file questions may improve at extra index/latency cost |

Zoekt, Qdrant, tree-sitter, and SCIP are **components**, not alternatives in one popularity table. OpenGrok/livegrep are useful lexical baselines; LlamaIndex can orchestrate experiments; Continue is an integrated implementation to study. None of their project pages establishes the hypotheses above.

## Freeze a corpus

Choose one repository snapshot that you are allowed to process and redistribute results about. Record:

- immutable commit and license;
- languages, files, non-generated lines, and excluded vendored/generated paths;
- parser versions/grammar commits;
- embedding model and version, tokenizer, chunk budget, overlap, and metadata;
- retrieval implementation commits and hardware;
- cold/warm-cache policy.

Do not tune chunking on the final question set. Split questions into development and held-out evaluation groups.

## Question taxonomy and ground truth

[`assets/question-set.example.jsonl`](assets/question-set.example.jsonl) defines the schema. Include at least these classes:

1. exact symbol definition;
2. caller/callee or reference tracing;
3. configuration-to-runtime flow;
4. error origin and handling path;
5. cross-file behavior/invariant;
6. architecture/concept question whose answer spans symbols;
7. negative question where the repository does not contain the claimed behavior.

Each question names one or more relevant spans (`path`, start/end lines, rationale). Two reviewers should resolve disagreement before freezing held-out labels. If relevance is incomplete, label the metric as pooled/partial judgment rather than ground truth.

## Metrics

For every pipeline and question class report:

- **Recall@k:** whether any/all judged relevant spans appear in top *k*; define which variant you use.
- **MRR:** reciprocal rank of the first relevant result.
- **nDCG@k** only when graded relevance is available.
- **context precision / duplicate rate:** useful tokens versus repeated/irrelevant tokens under the same context budget.
- **p50/p95 query latency:** cold and warm reported separately.
- **index wall time, peak memory, and index bytes.**
- **failure bucket:** split-boundary, naming mismatch, cross-file miss, generated-code noise, stale index, or no-answer false positive.

Do not combine quality and cost into one score unless weights were fixed before the run. Report per-taxonomy results; an average can hide a pipeline that fails every cross-file question.

## Reproducible run

1. Build all pipelines from the same clean corpus snapshot.
2. Enforce the same final context-token budget and `k` reporting points.
3. Run the development set; tune only declared parameters.
4. Freeze configuration digests.
5. Run held-out questions in randomized order, with cold/warm protocols declared.
6. Store ranked chunk IDs, scores, timings, and index stats—not only final metrics.
7. Inspect the top failure bucket and publish representative redacted examples.
8. Repeat after changing one variable; do not change chunker, embedding model, and corpus simultaneously.

## Component evidence map

- [tree-sitter](https://github.com/tree-sitter/tree-sitter) supports concrete syntax trees/incremental parsing used to construct symbol chunks.
- [SCIP](https://github.com/sourcegraph/scip) defines a code-intelligence index for symbols/occurrences used by P3.
- [Zoekt](https://github.com/sourcegraph/zoekt), [OpenGrok](https://github.com/oracle/opengrok), and [livegrep](https://github.com/livegrep/livegrep) provide different lexical/search baselines.
- [Qdrant](https://github.com/qdrant/qdrant) provides vector/payload retrieval infrastructure; it does not choose chunk quality.
- [LlamaIndex](https://github.com/run-llama/llama_index) supplies ingestion/retrieval/evaluation components that can orchestrate a run.
- [Continue](https://github.com/continuedev/continue) is an integrated coding-assistant stack whose indexing/context design can be studied, but product behavior bundles multiple decisions.

These links support component roles. Pipeline composition, question taxonomy, fairness controls, and metrics are this benchmark’s authored protocol and remain to be validated on the chosen corpus.

## Interpreting a result

A winner is local to the frozen corpus, queries, labels, embedding model, and context budget. Prefer the simplest pipeline within the uncertainty band of the best result. If P3 adds cost without a held-out cross-file gain, do not keep it because “graphs sound smarter.” If judgments are sparse, say evidence is insufficient.

## Maintenance

A new benchmark version is required when corpus commit, parser/grammar, embedding model, retrieval implementation, question labels, or metric code changes. Preserve old configurations and raw rankings. Never replace `NOT_RUN` with zero.

Sources: [`sources/sources.json`](sources/sources.json). Question artifact: [`assets/question-set.example.jsonl`](assets/question-set.example.jsonl).

## Adjacent guides

- [llm-test-determinism-guide](https://github.com/Simmondsun/llm-test-determinism-guide) — Use it when retrieval is inside an application whose regression tests must separate exact and sampled checks.
- [ai-research-reproducibility](https://github.com/Lindenberg668/ai-research-reproducibility) — Use it when benchmark claims need an immutable research packet and variance record.
- [agent-observability-field-guide](https://github.com/duthdoll45/agent-observability-field-guide) — Use it when retrieval, model calls, and downstream tool decisions need trace-level diagnosis.

## License

Original benchmark protocol and example question schema are available under CC BY 4.0. Corpus and linked projects retain their own licenses.

If this saved you a decision, star the repo — it is how the next reader finds it.
