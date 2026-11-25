# Sentiment Analysis Tool

<p align="center">
  <a href="https://amatya27.github.io/sentiment-analysis-tool/"><b>Try it live</b></a> &nbsp;·&nbsp; runs fully in your browser, no server, no API key
</p>

<p align="center">
  <a href="https://amatya27.github.io/sentiment-analysis-tool/">
    <picture>
      <source media="(prefers-color-scheme: dark)" srcset="media/screenshot-dark.png">
      <img src="media/screenshot.png" width="520" alt="Sentiment Analysis Tool scoring a mixed hotel review overall and sentence by sentence">
    </picture>
  </a>
</p>

Paste any text and it tells you whether it reads positive, negative or neutral, how confident it is and, for longer text, how each sentence scores on its own. I built it while exploring how text analysis could help with hiring workflows, like reading candidate feedback or interview notes.

## Two models, and how I picked the default

The tool runs Hugging Face models in the browser with [Transformers.js](https://github.com/xenova/transformers.js), so after the first download everything happens on your own device.

My first version used DistilBERT fine-tuned on SST-2. It's small and fast, but it was trained only on positive vs negative movie reviews, so it has no idea what "neutral" is. It called "The meeting has been moved to Thursday at 3pm" **negative with 99.5% confidence**.

So I benchmarked it against a RoBERTa model trained with a real neutral class (`cardiffnlp/twitter-roberta-base-sentiment-latest`), using 200 random labeled examples from each of three public datasets:

| Test data | RoBERTa (default) | DistilBERT |
|---|---|---|
| Tweets: positive / neutral / negative | **78.5%** | 46.5% |
| Product reviews: positive / neutral / negative | **71.5%** | 64.5% |
| Movie review sentences: positive / negative only | 79.5% | **85.0%** |

RoBERTa wins wherever "neutral" is a real option, which is most everyday text, so it's now the default ("Accurate"). DistilBERT stays as the "Fast" option. It's a smaller download and still the better pick for clearly opinionated text. I also tried blending the two models and different confidence cut-offs for DistilBERT's neutral; none beat just picking the right model for the job.

You can re-run the benchmark in your own browser at [`/eval`](https://amatya27.github.io/sentiment-analysis-tool/eval/). It pulls the data live from the Hugging Face datasets API with a fixed seed.

## Features

- Overall label with a bar for each class
- Sentence-by-sentence highlights, so mixed reviews show which parts are positive or negative
- Switch between the Accurate and Fast models; each downloads once, with a progress bar
- Example texts to try in one click, `Ctrl + Enter` to analyze
- Light and dark theme (follows your system by default)

## Running it locally

It's a single `index.html`. Because it loads the models as ES modules, open it through a local server instead of double-clicking the file:

```bash
python -m http.server 8000
# then open http://localhost:8000
```

## Changelog

**2026**
- New design with light and dark themes.
- RoBERTa 3-class model as the default, after benchmarking it against DistilBERT.
- Sentence highlights, a download progress bar, per-class bars and a benchmark page.
- Fixed a broken comment above `<!DOCTYPE html>` that pushed browsers into quirks mode.
- Removed a duplicate import of the library and updated Transformers.js from 2.0.1 to 2.17.2.
