# Evaluation metrics

Metrics are computed **with respect to the reference summaries** and/or
**with respect to the source documents** (reference-free).

## Libraries and implementations

| Metric | Group | Evaluated against | Paper | Implementation |
|---|---|---|---|---|
| ROUGE-L | Lexical | Reference | [Lin, 2004](https://aclanthology.org/W04-1013) | [google-research/rouge](https://github.com/google-research/google-research/tree/master/rouge) (`pip install rouge-score`) |
| BLEU | Lexical | Reference | [Papineni et al., 2002](https://aclanthology.org/P02-1040/) | [mjpost/sacrebleu](https://github.com/mjpost/sacrebleu) |
| BERTScore | Semantic | Reference | [Zhang et al., 2020](https://arxiv.org/abs/1904.09675) | [Tiiiger/bert_score](https://github.com/Tiiiger/bert_score) |
| FactCC | Faithfulness (entailment) | Reference + source | [Kryscinski et al., 2020](https://aclanthology.org/2020.emnlp-main.750/) | [salesforce/factCC](https://github.com/salesforce/factCC) — HF checkpoint [`manueldeprada/FactCC`](https://huggingface.co/manueldeprada/FactCC) |
| QuestEval | Faithfulness (QA) | Reference + source | [Scialom et al., 2021](https://aclanthology.org/2021.emnlp-main.529/) | [ThomasScialom/QuestEval](https://github.com/ThomasScialom/QuestEval) |
| Entity-level F / FH / non-factual FH | Hallucination (IE) | Reference | [Nan et al., 2021](https://aclanthology.org/2021.eacl-main.235/), [Bendahman et al., 2025b](https://aclanthology.org/2025.naacl-long.275/) | Original entity-level metrics: [amazon-science/fact-check-summarization](https://github.com/amazon-science/fact-check-summarization); adapted (see formulas below) |
| Triple-level F / FH / non-factual FH | Hallucination (IE) | Reference | This paper | See formulas below |
| Coverage (entities, triples) | Hallucination (IE) | Source | This paper | See formulas below |
| G-Eval | LLM-as-a-judge | Source / world knowledge | [Liu et al., 2023](https://aclanthology.org/2023.emnlp-main.153/) | [nlpyang/geval](https://github.com/nlpyang/geval) |
| GPTScore | LLM-as-a-judge | Source / world knowledge | [Fu et al., 2023](https://arxiv.org/abs/2302.04166) | [jinlanfu/GPTScore](https://github.com/jinlanfu/GPTScore) |
| SelfCheckGPT | LLM-as-a-judge | Source / world knowledge | [Manakul et al., 2023](https://aclanthology.org/2023.emnlp-main.557/) | [potsawee/selfcheckgpt](https://github.com/potsawee/selfcheckgpt) (`pip install selfcheckgpt`) |
| Spearman's ρ (metric correlation) | Analysis | — | — | [`scipy.stats.spearmanr`](https://docs.scipy.org/doc/scipy/reference/generated/scipy.stats.spearmanr.html) |

LLM judge for G-Eval, GPTScore and SelfCheckGPT:
[`mistralai/Mistral-7B-Instruct-v0.2`](https://huggingface.co/mistralai/Mistral-7B-Instruct-v0.2)
(open-source and different from all generation models). Each is run twice:
*document factuality* (original authors' prompt) and *global factuality*
(adapted prompt, see [`prompts/evaluation/`](../prompts/evaluation/)).

## Triple-oriented metrics (defined in this paper)

Notation, for a document *dᵢ*, its reference summary *rᵢ* and generated summary *gᵢ*:

- *T_gᵢ*: triples in the generated summary
- *T_Frᵢ*: faithful triples of the reference summary (also present in *dᵢ*)
- *T_Ainrᵢ*: abstractive triples of the reference summary that appear in at least one document of the collection
- *T_rᵢ*: triples in the reference summary
- *T_dᵢ*: triples in the source document

A triple is *(legal entity, relation, legal entity)* with relation ∈
{amend, repeal, correct, supplement, extend, recast, replace}.

**Faithfulness rate**

R(F) = |T_gᵢ ∩ T_Frᵢ| / |T_Frᵢ|  P(F) = |T_gᵢ ∩ T_Frᵢ| / |T_gᵢ|

**Factual hallucination rate** (abstractive content also produced by experts)

R(FH) = |T_gᵢ ∩ T_Ainrᵢ| / |T_Ainrᵢ|  P(FH) = |T_gᵢ ∩ T_Ainrᵢ| / |T_gᵢ|

**Non-factual hallucination rate** (lower is better)

P(NFH) = |T_gᵢ \ (T_rᵢ ∪ T_dᵢ)| / |T_gᵢ|

**Coverage rate** (reference-free)

R(C) = |T_gᵢ ∩ T_dᵢ| / |T_dᵢ|  P(C) = |T_gᵢ ∩ T_dᵢ| / |T_gᵢ|

The same four metrics are also computed at the **entity** level (replace
triples by entities). Note: the non-factual hallucination formula for entities
is slightly modified w.r.t. Bendahman et al. (2025b) — an entity is a
non-factual hallucination only if it appears neither in the source document
nor in the reference summary.
