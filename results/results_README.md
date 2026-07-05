# Results

This folder contains the evaluation outputs for every experiment in the study. All configurations were evaluated on the **same fixed 500-instance test split** of the expert-labelled PubMedQA subset (PQA-L), using identical corpus chunks and the same generator, so that differences between systems are attributable to the single factor that was varied.

## Files in this folder

| File | Experiment | Description |
|---|---|---|
| `eval_norag.json` | No-RAG baseline | The language model answers each question with no retrieved evidence. |
| `eval_results.json` | BioBERT-RAG | RAG using the BioBERT embedder (biomedical vocabulary, no similarity objective). |
| `eval_minilm.json` | MiniLM-RAG | RAG using all-MiniLM-L6-v2 (general vocabulary, similarity objective). |
| `eval_spubmed.json` | S-PubMedBert-RAG | RAG using S-PubMedBert-MS-MARCO (biomedical vocabulary and similarity objective). |
| `comparison_summary.json` | All systems | Combined metrics for every configuration, produced by the comparison cell. |

## Record format

Each `eval_*.json` file is keyed by the question's PubMed identifier (PMID). Each record contains:

| Field | Meaning |
|---|---|
| `pred` | the model's predicted decision (`yes` / `no` / `maybe`) |
| `gold` | the expert ground-truth decision |
| `retrieval_hit` | whether the correct source abstract was among the retrieved passages (`true`/`false`; not applicable to No-RAG) |
| `faithful` | the faithfulness verdict for the answer (`SUPPORTED` / `NOT_SUPPORTED`; `N/A` for No-RAG) |

`comparison_summary.json` stores the aggregate metrics (accuracy, macro-F1, per-class F1, retrieval hit rate, faithfulness rate) for each system.

---

## Experiments carried out

Four configurations were evaluated. They form a controlled comparison across two independent factors: whether retrieval is used at all, and — among the retrieval systems — whether the embedding model has biomedical domain adaptation, a sentence-similarity training objective, or both.

| Configuration | Retrieval | Embedder property |
|---|---|---|
| No-RAG | None | Language model answers unaided |
| BioBERT-RAG | Dense | Biomedical vocabulary; no similarity objective |
| MiniLM-RAG | Dense | General vocabulary; similarity objective |
| S-PubMedBert-RAG | Dense | Biomedical vocabulary and similarity objective |

## Overall results

Evaluated on the PQA-L test set (n = 500):

| Configuration | Accuracy | Macro-F1 | Hit rate | Faithfulness | Hallucination |
|---|---|---|---|---|---|
| No-RAG | 0.322 | 0.293 | — | — | — |
| BioBERT-RAG | 0.532 | 0.472 | 0.880 | 0.750 | 0.250 |
| MiniLM-RAG | 0.640 | 0.543 | 0.996 | 0.910 | 0.090 |
| **S-PubMedBert-RAG** | **0.652** | **0.557** | 0.994 | 0.906 | 0.094 |

## Per-class F1

| Configuration | yes | no | maybe |
|---|---|---|---|
| No-RAG | 0.465 | 0.234 | 0.179 |
| BioBERT-RAG | 0.660 | 0.507 | 0.250 |
| MiniLM-RAG | 0.762 | 0.644 | 0.222 |
| S-PubMedBert-RAG | 0.767 | 0.636 | 0.267 |

---

## What the experiments show

**Retrieval substantially improves clinical question answering.** Every retrieval-augmented configuration outperforms the no-retrieval baseline by 21–33 accuracy points, confirming that grounding the model in retrieved biomedical evidence improves factual accuracy.

**The embedder's training objective matters more than its domain vocabulary.** BioBERT, despite its biomedical adaptation, is the weakest retriever (hit rate 0.880) and weakest overall system, because it was not trained to produce similarity-oriented embeddings. The similarity-trained embedders retrieve the correct evidence for over 99% of questions.

**A domain-adapted sentence-transformer is best.** S-PubMedBert, which combines biomedical adaptation with a similarity objective, achieves the highest accuracy and macro-F1. The margin over MiniLM is modest; the decisive gap is between both similarity-trained models and BioBERT.

**Reliability tracks retrieval quality.** Hallucination is highest for BioBERT-RAG (0.250) and much lower for the similarity-trained systems (~0.09), because better retrieval supplies the model with the correct evidence to ground its answer.

**Generation is the remaining bottleneck.** Even the best system retrieves the correct evidence for 99.4% of questions but answers only 65.2% correctly, indicating that once evidence is available, the reasoning stage — not retrieval — is the main source of remaining error.

**The `maybe` class is the hardest for every system.** Its per-class F1 stays low (0.18–0.27) across all configurations, reflecting that `maybe` is the smallest and most ambiguous class in the dataset rather than a weakness of any single system.

---

## Notes on reproducibility

- A fixed random seed produces the same 500-instance test split on every run.
- Only the embedding model changes between the retrieval configurations; the corpus, pipeline, and generator are identical.
- The faithfulness and hallucination figures are produced by an automated language-model judge that has not been validated against human annotation, and should be read as a relative, automated indicator of grounding rather than an absolute measure.
- Because the generator runs under 4-bit quantisation and library versions evolve, exact metric values may vary slightly when the notebook is re-run.
