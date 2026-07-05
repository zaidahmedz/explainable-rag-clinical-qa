# Explainable Retrieval-Augmented Generation for Clinical Question Answering

An explainable RAG framework that answers biomedical questions by retrieving evidence from PubMedQA, generating grounded answers with a quantised large language model, and exposing the evidence behind every answer through an explainability layer.

This repository contains the implementation notebook, evaluation outputs, and supporting material for the MSc thesis *Explainable Retrieval-Augmented Generation for Clinical Question Answering*.

---

## Overview

Standalone large language models can answer clinical questions fluently but without evidence or transparency, and they sometimes hallucinate. This project grounds answer generation in retrieved biomedical evidence and makes each answer verifiable. It also runs a controlled comparison of embedding models to determine what makes a good biomedical retriever.

The pipeline is: build a corpus from PubMedQA → chunk and embed it → index it in FAISS → retrieve the top-k evidence for a question → generate a grounded yes/no/maybe answer with an instruction-tuned LLM → expose the evidence, attribution, confidence, and a faithfulness check.

## Key findings

Evaluated on the expert-labelled PubMedQA test split (500 questions):

| System | Accuracy | Macro-F1 | Hit rate | Hallucination |
|---|---|---|---|---|
| No-RAG (unaided) | 0.322 | 0.293 | — | — |
| BioBERT-RAG | 0.532 | 0.472 | 0.880 | 0.250 |
| MiniLM-RAG | 0.640 | 0.543 | 0.996 | 0.090 |
| **S-PubMedBert-RAG** | **0.652** | **0.557** | 0.994 | 0.094 |

- **Retrieval helps substantially** — every retrieval-augmented system beats the no-retrieval baseline by 21–33 accuracy points.
- **The embedder's training objective matters more than its domain vocabulary** — a similarity-trained model retrieves far better than a biomedical model without a similarity objective.
- **A domain-adapted sentence-transformer is best** — combining biomedical vocabulary with a similarity objective gives the strongest overall result.
- **Generation, not retrieval, is the remaining bottleneck** — the best system retrieves the correct evidence 99.4% of the time but answers 65.2% correctly.

*Note: these figures are from the reference run in this repository. Re-running may vary slightly with environment and model versions.*

## Explainability layer

Every answer is accompanied by:

- **Retrieval transparency** — the retrieved passages with ranks and similarity scores
- **Evidence highlighting** — the sentences that most support the answer
- **Source attribution** — PubMed IDs and chunk IDs for traceability
- **Confidence estimation** — retrieval confidence derived from similarity scores
- **Faithfulness assessment** — an automated check of whether the answer is supported by the evidence (used to measure hallucination)

---

## Repository structure

```
.
├── README.md
├── requirements.txt
├── .gitignore
│
├── notebook/
│   └── Clinical_RAG_Colab_T4.ipynb      # main implementation notebook (Colab, T4 GPU)
│
├── data/
│   └── README.md                         # how to obtain PubMedQA (not redistributed here)
│
├── results/
│   ├── comparison_summary.json           # combined metrics for all systems
│   ├── eval_results.json                 # BioBERT-RAG (proposed) predictions
│   ├── eval_norag.json                   # no-retrieval baseline
│   ├── eval_minilm.json                  # MiniLM-RAG baseline
│   └── eval_spubmed.json                 # S-PubMedBert-RAG baseline
│
├── figures/
│   ├── accuracy_comparison.png           # accuracy / macro-F1 across systems
│   ├── hit_rate.png                      # retrieval hit rate
│   ├── per_class_f1.png                  # per-class F1 (yes/no/maybe)
│   └── architecture.png                  # pipeline diagram
│


Notes on what to commit and what to leave out:

- **Do commit** the notebook, the JSON result files, the figures, and the documentation. These are small and are the reproducible record of the work.
- **Do not commit** the raw PubMedQA data (see licensing below), the FAISS index, or the embedding arrays (`chunk_embeddings.npy`, `*_embeddings.npy`, `faiss.index`). These are large, regenerable artefacts and are excluded via `.gitignore`.

---

## Getting started

### Requirements

- A GPU environment. The notebook is written for **Google Colab with a free T4 GPU**; it also runs on any comparable CUDA GPU.
- Python 3.10+. Key libraries are listed in `requirements.txt` (Transformers, Sentence-Transformers, FAISS, bitsandbytes, NumPy).

### Data

The retrieval corpus and evaluation labels come from **PubMedQA** (Jin et al., 2019). The dataset is not redistributed in this repository. See `data/README.md` for how to obtain the required file (`ori_pqal.json`) from the official source and where to place it.

### Running the notebook

1. Open `notebook/Clinical_RAG_Colab_T4.ipynb` in Google Colab and set the runtime to a T4 GPU.
2. Place the PubMedQA file in your Google Drive as described in `data/README.md`.
3. Run the cells top to bottom. The offline stage (corpus → chunks → embeddings → FAISS index) is checkpointed to Drive, so interrupted sessions can resume.
4. The evaluation and baseline cells write their outputs to the `results/` files, and the comparison cell prints the summary table.

The notebook is organised into clearly numbered sections mirroring the pipeline: environment, data, knowledge-base construction, retrieval, generation, explainability, evaluation, and the baseline comparisons.

---

## Reproducibility

- A fixed random seed produces the same 500-instance test split on every run.
- Every configuration is evaluated on the identical test set; only the embedding model changes between retrieval systems, so differences are attributable to the retriever.
- Long computations checkpoint to persistent storage and resume after interruption.

Because the generator runs under 4-bit quantisation and library versions evolve, exact metric values may differ slightly from the reference results above when re-run.

---

## Datasets and licensing

- **PubMedQA** is used under its own licence (MIT) and is cited as: Jin, Q., Dhingra, B., Liu, Z., Cohen, W. and Lu, X., 2019. *PubMedQA: A Dataset for Biomedical Research Question Answering*. In Proceedings of EMNLP-IJCNLP 2019.
- The dataset itself is **not** included in this repository; obtain it from the official source.
- The models used (BioBERT, all-MiniLM-L6-v2, S-PubMedBert-MS-MARCO, and the instruction-tuned generator) are obtained from their respective providers under their own licences.

## Intended use

This is a **research prototype** built to study explainable retrieval-augmented generation. It is **not** a clinical tool, does **not** provide medical advice or diagnosis, and must not be used as a substitute for professional medical judgement.

## Citation

If you refer to this work, please cite the thesis:

```
[Your Name], 2026. Explainable Retrieval-Augmented Generation for Clinical
Question Answering. MSc Thesis, Liverpool John Moores University.
```

## Usage and reuse

This repository accompanies an MSc thesis and is provided for reference and to document the work. It is not released under an open-source licence; all rights are reserved by the author. If you would like to reuse any part of the code, please contact the author. Datasets and pre-trained models referenced here are subject to their own licences.

## Acknowledgements

Developed as an MSc thesis project. Thanks to the maintainers of PubMedQA and the open-source models and libraries that made this work possible.
