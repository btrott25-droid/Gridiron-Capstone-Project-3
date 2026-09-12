# Gridiron Brief

## AI-Powered Fantasy Football League Chat Summarization

**Author:** Bennett Trott  
**Program:** Flatiron School Data Science  
**Project:** Capstone Project 3 — Large Language Models / Dialogue Summarization

## Project Overview

Fantasy football leagues generate fast-moving group chats about trades, waiver claims, injuries, lineup decisions, league votes, and commissioner announcements. When a user falls behind, important information can become buried among jokes, reactions, and side conversations.

**Gridiron Brief** is a proof-of-concept abstractive summarization system designed to condense long fantasy-football-style conversations into short summaries. The project uses the **SAMSum** dialogue dataset and a **BERT encoder-decoder architecture**, with evaluation focused on both ROUGE performance and qualitative factual reliability.

The final system improved substantially over the MVP baseline, but the human review also showed that speaker attribution and factual grounding remain significant limitations. The model should therefore be treated as a research prototype rather than a deployment-ready summarizer.

## Business Problem

A useful fantasy-football chat summarizer should help users quickly recover the most important information from missed conversations, including:

- trade proposals and decisions,
- waiver-wire plans,
- injury and lineup updates,
- commissioner announcements,
- league-rule discussions and votes.

The intended stakeholder is a fantasy-football platform or league-management product team evaluating whether automated chat summarization could improve the user catch-up experience.

## Dataset

The project uses the **SAMSum** dataset, a human-annotated corpus of everyday messenger-style dialogues and abstractive summaries.

Raw dataset size:

| Split | Rows |
|---|---:|
| Train | 14,731 |
| Validation | 818 |
| Test | 819 |
| **Total** | **16,368** |

A leakage audit identified exact dialogue overlap across the original splits. The cleaning process kept the test split unchanged, removed validation dialogues overlapping the test set, and removed training dialogues overlapping either cleaned validation or test data.

Final cleaned dataset:

| Split | Rows |
|---|---:|
| Train | 14,638 |
| Validation | 814 |
| Test | 819 |
| **Total** | **16,271** |

Repeated dialogues that appeared only within the training split were retained because many contained alternate valid human summaries.

## Preprocessing and Tokenization

Dialogue structure, speaker labels, punctuation, slang, and conversational style were intentionally preserved. Cleaning focused primarily on whitespace normalization and cross-split leakage prevention.

Token-length analysis supported the following limits:

- **Maximum input length:** 384 tokens
- **Maximum target length:** 64 tokens
- **Dialogue coverage at 384 tokens:** 96.98%
- **Summary coverage at 64 tokens:** 98.24%

Longer examples are truncated during model preprocessing.

## Modeling Approach

### MVP

The MVP used a compact BERT encoder-decoder configuration based on:

`google/bert_uncased_L-4_H-256_A-4`

The MVP validated the end-to-end training and evaluation pipeline, but its generated summaries frequently contained unrelated hallucinations and severe factual errors.

### Final Model

The final refinement retained the assignment-required BERT encoder-decoder design while starting from a summarization-oriented warm checkpoint:

`mrm8488/bert-small2bert-small-finetuned-cnn_daily_mail-summarization`

Key final-training settings:

- 3,000 training examples
- 300 validation examples
- 300 held-out test examples
- 2 epochs
- learning rate: `2e-5`
- train batch size: `2`
- evaluation batch size: `2`
- gradient accumulation: `8`
- weight decay: `0.01`
- early stopping enabled
- CPU-only training environment

The final run completed 376 training steps across two epochs.

### Generation Tuning

Generation settings were selected using validation data only. The best tested configuration used:

- 4-beam search
- maximum generation length of 64 tokens
- `no_repeat_ngram_size=3`
- `repetition_penalty=1.1`
- early stopping enabled

The held-out test set was not used for generation-setting selection.

## Results

### Quantitative Evaluation

| Metric | MVP | Final Model | Change | Original Target | Target Met? |
|---|---:|---:|---:|---:|:---:|
| ROUGE-1 | 13.99 | **31.71** | +17.72 | 35 | No |
| ROUGE-2 | 2.13 | **13.23** | +11.10 | 12 | **Yes** |
| ROUGE-L | 12.16 | **23.13** | +10.97 | 32 | No |

The final model substantially outperformed the MVP on all three ROUGE metrics and exceeded the original ROUGE-2 target.

### Human Review

Ten held-out test summaries were manually reviewed using strict binary criteria.

| Criterion | Result |
|---|---:|
| Main idea preserved | **60%** |
| Completely factually supported | **0%** |
| Readable | **60%** |

The factual-support criterion was intentionally strict: every generated claim and speaker attribution had to be supported by the source dialogue. Every reviewed summary contained at least one unsupported, contradictory, repetitive, or incorrectly attributed detail.

This **0% strict factual-support result should not be treated as a direct measurement of the original pitch's 85% “major factual-error-free” target**, because the final review used a different and stricter rubric. The proposed 4.0/5 usefulness score, catch-up-time reduction, and important-information recall metrics were also not formally evaluated and remain future user-study goals.

## Fantasy-Football Domain Test

The refined model was also tested on custom conversations involving:

- a trade proposal,
- an injury update,
- a league vote,
- waiver-wire planning,
- a mixed fantasy-football discussion.

Compared with the MVP, the final model generally remained on-topic and recognized the central fantasy-football concepts. However, it still confused speakers, repeated information, and occasionally assigned actions to the wrong participant.

These results support the project as a **functional proof of concept**, but not as a production-ready summarization system.

## Key Limitations

- Training was constrained by CPU-only hardware.
- Only a subset of SAMSum was used for final fine-tuning.
- SAMSum is general conversational data rather than fantasy-football-specific data.
- ROUGE measures lexical overlap and cannot guarantee factual correctness.
- Abstractive generation remains vulnerable to hallucination and attribution errors.
- The human-review sample contained only 10 examples.
- Product-level usefulness, catch-up time, and important-information recall were not formally tested.

## Future Work

Recommended next steps include:

1. Fine-tune on more SAMSum examples using GPU resources.
2. Compare the BERT encoder-decoder model against summarization-focused architectures such as BART and T5 under the same evaluation protocol.
3. Add fantasy-football-specific training data after the general summarizer is more reliable.
4. Expand factual-consistency evaluation with a larger human-rated sample.
5. Test catch-up time, usefulness, and important-information recall with real users.
6. Preserve links to the original conversation so users can verify generated claims.

## Repository Structure

```text
Gridiron-Capstone-Project-3/
├── Gridiron_Brief_Final_Project.ipynb   # Final executed project notebook
├── Gridiron_Brief_MVP.ipynb             # Earlier MVP notebook
├── Gridiron_Brief_Project_Pitch.pdf     # Original project pitch
├── README.md                            # Project overview and results
├── requirements.txt                    # Python dependencies
└── .gitignore                          # Excludes generated checkpoints/models
```

Generated model weights and training checkpoints are intentionally excluded from Git. The final notebook saves the trained model locally in Hugging Face-compatible format and verifies that it can be reloaded for independent inference.

## Reproducing the Project

1. Clone the repository.
2. Create and activate a Python 3.10 environment.
3. Install dependencies:

```bash
pip install -r requirements.txt
```

4. Open `Gridiron_Brief_Final_Project.ipynb` in Jupyter or VS Code.
5. Run the notebook from top to bottom.

The submitted run used **Python 3.10.20** and **PyTorch 2.14.0+cpu** on Windows with CUDA unavailable. The final training cell took about one hour on that CPU-only setup, so runtime will vary substantially by hardware.

## Saved Model

The final notebook saves the trained model to:

```text
gridiron_brief_final_model/
```

The directory contains the Hugging Face model/tokenizer files needed for later inference. It is ignored by Git because generated model artifacts are large and can be reproduced from the notebook.

## References

- Devlin, J., Chang, M.-W., Lee, K., & Toutanova, K. (2019). *BERT: Pre-training of Deep Bidirectional Transformers for Language Understanding*. NAACL-HLT.
- Gliwa, B., Mochol, I., Biesek, M., & Wawer, A. (2019). *SAMSum Corpus: A Human-annotated Dialogue Dataset for Abstractive Summarization*. ACL.
- Lin, C.-Y. (2004). *ROUGE: A Package for Automatic Evaluation of Summaries*. ACL Workshop.
- Rothe, S., Narayan, S., & Severyn, A. (2020). *Leveraging Pre-trained Checkpoints for Sequence Generation Tasks*. Transactions of the Association for Computational Linguistics.
- Hugging Face Transformers documentation.
- Hugging Face SAMSum dataset: `knkarthick/samsum`.
- Warm-start summarization checkpoint: `mrm8488/bert-small2bert-small-finetuned-cnn_daily_mail-summarization`.

## Bottom Line

Gridiron Brief demonstrates a complete dialogue-summarization workflow from business framing and leakage-aware preprocessing through transformer fine-tuning, validation-based decoding selection, held-out evaluation, human review, domain testing, and model persistence.

The final model produced major ROUGE gains over the MVP and crossed the ROUGE-2 target, while the human review showed that factual accuracy and speaker attribution still require significant improvement. That combination of measurable progress and transparent limitation analysis is the central result of the project.
