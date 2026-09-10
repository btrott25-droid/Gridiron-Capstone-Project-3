# Gridiron Brief

## AI-Powered Fantasy Football League Chat Summarization

Gridiron Brief is a dialogue summarization proof of concept designed to condense lengthy fantasy football league conversations into short, readable summaries.

The project uses the SAMSum dialogue dataset and a BERT-based encoder-decoder architecture implemented with Hugging Face Transformers.

## MVP Approach

- SAMSum dialogue dataset
- Cross-split leakage detection and removal
- BERT tokenization
- Compact BERT encoder-decoder model
- 23.4 million parameters
- 3,000 training examples
- 300 validation examples
- 2 training epochs
- ROUGE evaluation
- Qualitative error analysis
- Fantasy football dialogue demonstration

## MVP Results

Validation loss improved from 4.4683 after Epoch 1 to 4.3434 after Epoch 2.

Held-out test performance on 300 examples:

- ROUGE-1: 13.99
- ROUGE-2: 2.13
- ROUGE-L: 12.16

## Key Finding

The end-to-end summarization pipeline was successfully implemented, but the MVP model showed substantial factual consistency problems. Generated summaries sometimes introduced unsupported people, locations, or events and repeated generic language across unrelated conversations.

The current system should therefore be viewed as a functional proof of concept rather than a deployment-ready application.

## Future Improvements

Future development could include additional training data, more training epochs, GPU-based training, generation tuning, comparison with summarization-focused architectures such as BART or T5, and fantasy-football-specific fine-tuning.

## Notebook

[View the Gridiron Brief MVP Notebook](Gridiron_Brief_MVP.ipynb)
