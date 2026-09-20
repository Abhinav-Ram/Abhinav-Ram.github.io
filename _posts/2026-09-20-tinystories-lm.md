---
title: "TinyStoriesLM: A Replicated Study"
date: 2026-09-20
excerpt: "A replication of the TinyStories result: why a tiny language model can still generate coherent English, and what that says about scale, data, and model quality."
tags:
  - Artificial Intelligence
  - Language Models
  - Machine Learning
  - Research
  - Writing
---

## The hook

My mom runs a small business where she makes little 3D-modelled objects and raised-relief model objects. One of her things is to print them in white and bundle them with a small painting kit, marketing them as a kids' painting hobby. Having caught up with the latest trends, she uses quite a bit of AI here and there. More recently, she wanted to write small stories to go with the objects she sells, and was considering using AI to polish the story-writing and text. At first, I was like: "But what about human creativity? What about water resources?" But that got me thinking.

## Motivation

Forget creativity, do we really need billions of parameters to write just fluent, grammatical sentences? This was the same question asked in the influential paper by Eldan and Li [1]. The claim is not that tiny models match large ones in the task, but is more about the data. With a simple data distribution, models roughly 1000× smaller than GPT-2-scale systems can still produce fluent, grammatical stories. This project replicates that finding by training a model from scratch, with three settings of about 0.1M, 0.8M and 4.7M non-embedding parameters. In other words, the paper is essentially finding how much of that scale is "paying for" the model's ability to speak fluent English at all, versus paying for genuine world knowledge and reasoning?

## Our model

The main model is a decoder-only Transformer. Token IDs pass through token and position embeddings, then a stack of attention-plus-MLP blocks, then a LayerNorm and a tied linear head that outputs next-token probabilities.

We have a fresh byte-level BPE tokenizer with 4096 tokens. It starts from raw bytes and repeatedly merges the most frequent adjacent pair until the vocabulary is full. Every byte is valid, so there is no unknown token. GPT-2's 50,257-token vocabulary would mostly go unused on simple stories and would eat a tiny model's parameters. The paper instead reuses GPT-Neo's tokenizer, limited to its top 10K tokens.

For the data, only 20,000 of the 2.1M stories are used. That keeps runs to minutes, but it is also the biggest reason absolute numbers differ from the paper's.

## Training (self-supervised learning)

The model learns by predicting the next token in the text, so no labels are needed. Each step, it makes predictions on a batch, measures its error with cross-entropy loss, and updates its weights through backpropagation, using a gradually decaying learning rate, gradient clipping for stability, and the AdamW optimizer.

## Evaluation

We use two methods for evaluation:

1. **Perplexity** is *e* raised to the average cross-entropy loss. It's roughly how many equally likely tokens the model hesitated between. A score of 1 is perfect, and a score equal to the vocabulary size is random guessing. It is fast and automatic, but it is only comparable between models that share a tokenizer.
2. **LLM-as-a-judge** grades each completion from 1 to 10 on grammar, creativity, consistency and plot. Open-ended stories have no single right answer, so fixed-reference metrics like BLEU or ROUGE don't work. The trade-off is that the judge is a black box with its own biases. We use a local model, which is a weaker judge. Scores should be read as directional. Stories are generated at temperature 0.8 with top-k 50 for variety, and the judge runs at temperature 0 with JSON output for consistency.

## Results

Table 1 shows the model size, validation perplexity and LLM-judged scores (1 to 10) for the three configurations. We observe that validation perplexity decreases monotonically with model size, from 63.09 (tiny, ~100K non-embedding parameters) to 32.39 (small, ~793K) to 18.49 (medium, ~4.7M). Each increase in non-embedding parameters of roughly 6 to 8 times yields approximately a twofold reduction in perplexity (1.95x, then 1.75x). This reproduces the central finding of the TinyStories paper, which is that greater capacity reduces uncertainty over the next token on held-out stories, even at this small scale and with only 20K training stories. All three models share a single tokenizer, so the perplexity values are directly comparable.

| Config | Total params | Non-embedding params | Val. perplexity | Grammar | Creativity | Consistency | Plot | Avg |
|--------|-------------:|---------------------:|----------------:|--------:|-----------:|------------:|-----:|----:|
| tiny   | 378,624   | 100,096   | 63.09 | 6.37 | 4.53 | 7.47 | 5.47 | 5.96 |
| small  | 1,350,400 | 793,344   | 32.39 | 6.57 | 4.93 | 7.43 | 5.67 | 6.15 |
| medium | 5,853,184 | 4,739,072 | 18.49 | 8.60 | 6.33 | 8.27 | 6.80 | 7.50 |

*Table 1. Evaluation results for all configurations.*

### LLM-judged quality

Judge scores show a threshold effect rather than a smooth trend. Table 2 reports the change in mean score between consecutive model sizes.

| Dimension   | Tiny to Small | Small to Medium |
|-------------|--------------:|----------------:|
| Grammar     | +0.20 | +2.03 |
| Creativity  | +0.40 | +1.40 |
| Consistency | -0.04 | +0.84 |
| Plot        | +0.20 | +1.13 |
| Average     | +0.19 | +1.35 |

*Table 2. Change in mean judge score (1 to 10 scale) between consecutive model sizes.*

A finding here is that while perplexity improved by a similar relative amount at both steps, the LLM grader barely noticed any qualitative difference between tiny and small; however, medium was a step up across all dimensions. So we should remember that "good" perplexity doesn't automatically mean more coherent sentences. In this run, the 6-layer, 256-wide medium model is where text starts to read as coherent, so there's more of a threshold.

Another finding is that creativity is a difficult dimension to get right. It is poor across all the sizes (tiny, medium and small), further indicating that fluency is cheap but original variation needs capacity.

## Caveats to remember

These results come from a single run with a single seed, 30 holdout prompts and one completion per prompt, so the tiny to small gap (+0.19 avg) could partly be noise, while the small to medium gap (+1.35) is large enough to trust as a real effect. Mistral is a much weaker judge than the GPT-4 used in the paper, so absolute scores should be treated as directional, and the ranking is the trustworthy part.

## Back to the hook

Back to my mom's painting kits: her stories are simple, kid-level text, which is exactly the kind of data where small models are fluent, so a tiny model could plausibly handle the polishing at a fraction of the compute, which softens my water resources worry. Creativity was the weakest score at every size, though, so the original ideas behind her stories are still best left to her.

## References

[1] Eldan, R., and Li, Y. (2023). TinyStories: How small can language models be and still speak coherent English? arXiv preprint arXiv:2305.07759.
