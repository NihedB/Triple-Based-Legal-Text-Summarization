# Enhancing Legal Text Summarization with Triple-Based Generation

Companion repository for the article *Enhancing Legal Text Summarization with
Triple-Based Generation*. It gathers the resources needed to reproduce the
experiments: datasets, models, hyper-parameters, evaluation metrics (with links
to their implementations) and all the prompts used.

## Overview

Expert-written summaries of legal documents contain *abstractive* triples —
(entity, relation, entity) facts that are absent from the source document but
factually correct ("factual hallucinations"). We guide summary generation with
relational triples retrieved from a corpus-level triples graph, in order to
(1) reduce **non-factual hallucinations** and (2) induce **factual
hallucinations** mirroring the experts' contextualisation. Triples are injected
either into a fine-tuned PLM (LED, BART) or into LLM prompts.

## Repository structure

```
.
├── README.md
├── docs/
│   └── metrics.md                 # metrics, libraries, triple-level formulas
└── prompts/
    ├── README.md                  # index of prompts
    ├── generation/                # Appendices A and B
    │   ├── P-Basic.txt            # zero-shot baseline
    │   ├── P-Examples.txt         # few-shot baseline
    │   ├── P-Oracle.txt           # few-shot + oracle triples
    │   └── P-Graph.txt            # few-shot + graph-selected triples
    └── evaluation/                # Appendix C (global factuality)
        ├── geval_global_factuality.txt
        ├── gptscore_global_factuality.txt
        └── selfcheckgpt_global_factuality.txt
```

## Datasets

| Dataset | Description | Link |
|---|---|---|
| EUR-Lex-Sum (Aumiller et al., 2022) | 1,504 EU legal acts (≈12k tokens) with expert summaries (≈800 tokens). Split: 1,316 train / 188 test. | [HF: `dennlinger/eur-lex-sum`](https://huggingface.co/datasets/dennlinger/eur-lex-sum) · [paper](https://aclanthology.org/2022.emnlp-main.519/) |
| EUR-Lex-Triples (Bendahman et al., 2025a) | EUR-Lex-Sum annotated with legal relational triples: 14,396 triples (9,193 distinct). |[HF: : `dennlinger/eur-lex-triples`](https://huggingface.co/datasets/nihedb/)EUR-Lex-Triples · [paper](https://doi.org/10.1007/978-3-032-05409-8_25) |

Relations: `amend`, `repeal`, `correct`, `supplement`, `extend`, `recast`, `replace`.

## Triples graph

Two graphs are built: one on the training documents (G_DT) and one on the test
documents (G_DI).

- **Nodes**: documents (attribute: CELEX id), legal entities (attributes: type
  — regulation, decision, directive… — and code), persons, locations,
  organisations, subjects, keywords.
- **Edges**: document–document semantic similarity (cosine on
  [`facebook/bart-large-mnli`](https://huggingface.co/facebook/bart-large-mnli)
  embeddings, edge kept if w > 0.92), entity-in-document, subject–document,
  keyword–document, and legal relations between legal entities (the triples).
- **Triple selection**: triples connected to dᵢ by at most 1, 2 or 3 hops.

## Models

### Proposed approach (PLM fine-tuning)

| Model | Checkpoint |
|---|---|
| LED (Longformer-Encoder-Decoder) | TODO: [`allenai/led-base-16384`](https://huggingface.co/allenai/led-base-16384) or [`allenai/led-large-16384`](https://huggingface.co/allenai/led-large-16384) |
| BART Large | [`facebook/bart-large`](https://huggingface.co/facebook/bart-large) |

Input format: `[TRIPLETSET] t1 | t2 | … | t15 [DOCUMENT] <source document>`

| Hyper-parameter | Value |
|---|---|
| Epochs | 6 (epoch with lowest loss kept) |
| Learning rate | 2e-5 |
| Batch size | 1 |
| Warmup steps | 300 |
| Decoding | beam search, `num_beams = 4` |
| Max input length | 16,384 tokens (truncation) |
| Min summary length | 1,024 tokens |
| Triples injected | 15 |

Training settings (at inference, all settings use 15 graph-selected triples):

| Setting | Triples given during training |
|---|---|
| S0 | none (source document only) |
| S1 | ≤ 15 abstractive triples present in the reference summary and in ≥ 1 other corpus document |
| S2 | abstractive triples (as in S1) completed with graph-selected triples, balanced across triple types (15 total) |
| S3 | 15 graph-selected triples |
| Oracle | 15 abstractive triples at both training and inference (upper bound) |

### Baselines

| Baseline | Link |
|---|---|
| ECC — Entity Coverage Control (Zhang et al., 2022) | [paper](https://aclanthology.org/2022.findings-naacl.40/) |
| CTRLsum (He et al., 2022) | [salesforce/ctrl-sum](https://github.com/salesforce/ctrl-sum) |
| Entity-guided LED / BART (Bendahman et al., 2025b) | [paper](https://aclanthology.org/2025.naacl-long.275/) |

### LLMs (prompting)

| Model | Checkpoint |
|---|---|
| Llama 3.1 8B | [`meta-llama/Llama-3.1-8B-Instruct`](https://huggingface.co/meta-llama/Llama-3.1-8B-Instruct) |
| Mixtral 8×7B | [`mistralai/Mixtral-8x7B-Instruct-v0.1`](https://huggingface.co/mistralai/Mixtral-8x7B-Instruct-v0.1) |
| Mistral Large 2 | [`mistralai/Mistral-Large-Instruct-2407`](https://huggingface.co/mistralai/Mistral-Large-Instruct-2407) |
| Lawma 8B | [`ricdomolm/lawma-8b`](https://huggingface.co/ricdomolm/lawma-8b) |

Prompts: P-Basic, P-Examples (baselines), P-Oracle, P-Graph (triples) — see
[`prompts/`](prompts/).

## Evaluation

| Group | Metrics |
|---|---|
| Lexical / semantic | ROUGE-L, BLEU, BERTScore |
| Faithfulness | FactCC, QuestEval |
| Hallucination (entities & triples) | Faithfulness rate, Factual hallucination rate, Non-factual hallucination rate, Coverage rate |
| LLM-as-a-judge (judge: Mistral-7B-Instruct-v0.2) | G-Eval, GPTScore, SelfCheckGPT — *document* and *global* factuality |
| Metric agreement | Spearman's ρ across 22 system configurations |

Libraries, GitHub links and formulas: [`docs/metrics.md`](docs/metrics.md).

## Citation

```bibtex
@article{TODO,
  title   = {Enhancing Legal Text Summarization with Triple-Based Generation},
  author  = {TODO},
  journal = {TODO},
  year    = {TODO}
}
```

Related work by the authors:

- Bendahman et al. (2025a). *EUR-Lex-Triples: A Legal Relation Extraction Dataset from European Legislation.* TPDL 2025. [doi:10.1007/978-3-032-05409-8_25](https://doi.org/10.1007/978-3-032-05409-8_25)
- Bendahman et al. (2025b). *Not all Hallucinations are Good to Throw Away When it Comes to Legal Abstractive Summarization.* NAACL 2025. [ACL Anthology](https://aclanthology.org/2025.naacl-long.275/)

## License

TODO
