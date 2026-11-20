# bilan extraction with VLMs

TP3 of the Transformers & LLMs course at ENSAM Casablanca (group 6).

The task was to extract the actif and passif tables of moroccan balance sheets (CGNC format) from PDFs and scans into JSON with a vision language model, once through an API and once locally.

Models:
- qwen2.5-vl 7b, local with ollama on a kaggle T4
- nemotron 3.5 super vl, with the openrouter free tier

## Notebooks

- `01_benchmark.ipynb`: tries different prompts on our bilan (6), then runs both models on 7 bilans
- `02_qwen_strips.ipynb`: qwen kept putting numbers on the wrong line, so here we cut each page in 2, ask the model to just copy the rows, and match them to the CGNC lines in python

Both were run on kaggle (GPU T4, internet on). The PDFs from `data/bilans` go in a kaggle dataset, plus `data/gold` for the second notebook. The first one also needs an `OPENROUTER_API_KEY` secret.

## Results

On bilan 6 (17 lines we checked by hand):

| setup | qwen | nemotron |
|---|---|---|
| base | 71% | 94% |
| one_shot | 82% | 100% |
| template (list of lines) | 29% | 100% |
| text (pdf text added) | 6% | 100% |

Nemotron balances all 7 bilans (actif = passif), qwen only 2. Giving qwen the list of lines or the pdf text made it skip a lot of lines.

With the cut + copy + match approach, qwen gets 100% on bilan 3, 89.6% on bilan 10, 84.2% on bilan 6 and 80.3% on bilan 8 (88.5% avg), checked on every value using `data/gold`. One page of bilan 8 got cut because the answer was too long. The phone scan (bilan 2) passes all its sums, the blurry one (bilan 4) doesn't.

We forced the JSON schema in ollama and openrouter, and 85 of 86 pages came back valid.

Everything is in `results/results.csv`.
