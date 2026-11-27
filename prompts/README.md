# Prompts

All prompts used in the paper, copied verbatim from the appendices.

## `generation/` — summary generation with LLMs

| File | Name in paper | Type | Appendix | Placeholders |
|---|---|---|---|---|
| [`P-Basic.txt`](generation/P-Basic.txt) | P-Basic | Baseline, zero-shot | A, Fig. 8 | `{document}` |
| [`P-Examples.txt`](generation/P-Examples.txt) | P-Examples | Baseline, few-shot (1 example summary) | A, Fig. 9 | `{example}`, `{document}` |
| [`P-Oracle.txt`](generation/P-Oracle.txt) | P-Oracle | Few-shot + abstractive triples from the reference summary | B, Fig. 10 | `{example}`, `{oracle_triples}`, `{document}` |
| [`P-Graph.txt`](generation/P-Graph.txt) | P-Graph | Few-shot + triples selected from the triples graph | B, Fig. 11 | `{example}`, `{graph_triples}`, `{document}` |

Used with: Llama 3.1 8B, Mixtral 8×7B, Mistral Large 2, Lawma 8B.

## `evaluation/` — LLM-as-a-judge, *global factuality* variants

For **document factuality** (factuality w.r.t. the source document) we use the
**original prompts released by the metric authors**, unchanged:

| Metric | Original prompt |
|---|---|
| G-Eval (consistency) | [`prompts/summeval/con_detailed.txt`](https://github.com/nlpyang/geval/blob/main/prompts/summeval/con_detailed.txt) |
| GPTScore (summarization, factuality aspect) | prompt templates in the [GPTScore paper](https://arxiv.org/abs/2302.04166) / [repo](https://github.com/jinlanfu/GPTScore) |
| SelfCheckGPT (prompt variant) | `SelfCheckAPIPrompt` in [`selfcheckgpt/modeling_selfcheck_apiprompt.py`](https://github.com/potsawee/selfcheckgpt/blob/main/selfcheckgpt/modeling_selfcheck_apiprompt.py) |

For **global factuality** (the judge may use its parametric memory: *is the
information true in an absolute sense?*) we adapted these prompts:

| File | Metric | Appendix | Placeholders |
|---|---|---|---|
| [`geval_global_factuality.txt`](evaluation/geval_global_factuality.txt) | G-Eval | C, Fig. 12 | `{{Document}}`, `{{Summary}}` |
| [`gptscore_global_factuality.txt`](evaluation/gptscore_global_factuality.txt) | GPTScore | C, Fig. 13 | `{Text}`, `{Summary}` |
| [`selfcheckgpt_global_factuality.txt`](evaluation/selfcheckgpt_global_factuality.txt) | SelfCheckGPT | C, Fig. 14 | `{sentence}` |

Judge model for all three metrics: `mistralai/Mistral-7B-Instruct-v0.2`.
