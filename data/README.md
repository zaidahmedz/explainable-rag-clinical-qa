# Data

This project uses **PubMedQA** (Jin et al., 2019). The dataset is **not** redistributed in this repository; please obtain it from the official source.

## What you need

At minimum, the expert-labelled subset:

- `ori_pqal.json` — 1,000 expert-labelled instances (used as the evaluation gold standard)

Optionally, for a larger retrieval corpus:

- `ori_pqaa.json` — artificially-labelled subset (large; corpus only)
- `ori_pqau.json` — unlabelled subset (corpus only)

## Where to get it

Obtain the files from the official PubMedQA release (the labelled `ori_pqal.json` is available in the dataset's public repository; the larger subsets are linked from there).
https://github.com/pubmedqa/pubmedqa

## Where to put it

Place the file(s) in the dataset folder read by the notebook. When running on Google Colab with Google Drive mounted, the notebook expects:

```
MyDrive/clinical_rag/
├── Dataset/
│   └── ori_pqal.json        # required
└── artifacts/               # created automatically by the notebook
```

The notebook checks that the required file is present and will stop with a clear message if it is missing.

## Licence and citation

PubMedQA is released under the MIT License. Please cite:

> Jin, Q., Dhingra, B., Liu, Z., Cohen, W. and Lu, X., 2019. *PubMedQA: A Dataset for Biomedical Research Question Answering.* In Proceedings of EMNLP-IJCNLP 2019.
