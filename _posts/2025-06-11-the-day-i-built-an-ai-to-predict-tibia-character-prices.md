---
layout: post
title:  "The Day I Built an AI to Predict Tibia Character Prices"
date:   2025-06-11 01:00:00 -0300
categories: [ai, machine-learning]
tags: [en-US]
permalink: /posts/the-day-i-built-an-ai-to-predict-tibia-character-prices
---

A story about TCAQS: Tibia Character Automated Quotation System

> Portuguese version: [Link](https://www.alvaroinckot.com/posts/o-dia-em-que-construi-uma-ia-para-prever-precos-de-personagens-de-tibia)

## The spark (late‑night 2022)

Picture a rainy Sunday in 2022. I’m procrastinating on real‑life chores by scrolling [Tibia’s brand‑new auction house](https://www.tibia.com/news/?subtopic=latestnews), looking at characters priced like beachfront property. _Surely_ there’s a pattern here, I thought. Fifteen years of playing Tibia had wired my brain to sniff under‑ or over‑priced deals the way other people smell fresh coffee.

So I opened VS Code and wrote the first line of what would become TCAQS: `async def scrap(url): ...`  
Mission: scrape every single auction record, teach a model to predict fair prices, and maybe—just maybe—save a few souls from paying 6k TC in a skill-less knight in Antica.


## Three weeks, 650 000 HTML files, one battered laptop

Tibia’s rate‑limiter is the digital equivalent of an ankle weight. Three requests every second isn’t a suggestion—it’s the law. Multiply that by half a million pages and you get **six days** of constant crawling (add constant networks drops, and some Tibia website downtime as well). When the `results/` folder finally hit 650 k files, I zipped it, patted myself on the back, and… promptly moved to the United States.

Life happened: visas, boxes, a Florida heatwave. `tcaqs/` collected dust in a backup drive.

## Rediscovery (May 2024)

Fast‑forward to a spring cleaning session. I’m Marie‑Kondo‑ing old drives when a 14 GB archive labeled `tibia_auctions_2022.zip` stares back. Does it spark joy? _Absolutely._ And AI tooling has exploded since 2022, so why not finish the job?

## Death by a thousand <div>s

Parsing those HTML relics felt like fighting 1 000 paper cuts with a wooden shield. I wrote a tiny utility, **html‑scalpel**, that sliced each page into a dict, dumped it to Parquet, and batched inserts into SQLite. Seven hours later I had:

- 650 k rows
- 40+ raw fields (level, vocation, outfits, mounts, quest flags…)
- Two crucial targets: `final_bid`, `bid_status` (sold vs. minimum‑bid‑unsold)

![cpu goes brrrr](https://github.com/alvaroinckot/tcaqs/blob/main/assets/cpu-usage-result.png?raw=true)


## Model season: XGBoost vs. my ego

I promised myself **90 % R²** on test data, enough to pass my own Tibian sniff‑test. Here’s how that duel unfolded:

### v1: Baseline and Reality Check

I began with the obvious stats: level, vocation, world type, and a handful of other numeric attributes, thirty columns in total. A quick grid-search on plain XGBoost pushed training R² past 0.94, but test R² stalled at 0.89 while MAPE bounced all over, especially on low-level characters. The model was memorising the training set and learning little about real-world pricing, useful only as a floor for future work.

### v2: Packing the Lore

Next I stuffed in the bragging-rights features that really drive Tibia prices, such as quest flags, rare mounts, outfits, hirelings, and server type, all one-hot encoded and rebalanced. Bayesian hyper-parameter tuning lifted test R² to 0.94 and smoothed predictions for mid- to high-level chars. The new pain point, though, was low-level characters on freshly opened servers. They start overpriced because early adopters have excess gold, then drop rapidly as the server economy matures, so their actual values drift faster than my static scrape could capture, keeping MAPE high in that slice.

### v3: Purging Poisoned Labels

The real breakthrough came from cleaner targets, not extra features. Many auctions ending at the minimum bid had no buyer, so their listed prices were wishful thinking. Filtering those rows, log-transforming gold values, and retuning XGBoost flattened the error curve: overall MAPE fell to 0.42 (under 0.30 for sub-100 chars) and test R² settled at 0.935. CatBoost scored about the same but drank more VRAM, so XGBoost is still the champion for now.

![Top Features](https://github.com/alvaroinckot/tcaqs/blob/main/assets/top-features.png?raw=true)

## A few results

The feature engineering evolution in numbers:

| Model       | Split    | MSE        | RMSE     | MAE      | MAPE   | R2     | Explained_Variance|
|-------------|----------|------------|----------|----------|--------|--------|-------------------|
| results-v1  | Training | 2289526.25 | 1513.12  | 703.37   |        | 0.9393 |                   |
| results-v1  | Testing  | 4209464.5  | 2051.7   | 857.07   |        | 0.8874 |                   |
| results-v2  | Training | 543840.75  | 737.46   | 376.31   |        | 0.9856 |                   |
| results-v2  | Testing  | 2172790.25 | 1474.04  | 582.8    |        | 0.9419 |                   |
| results-v2-es | Training | 333324.59 | 577.3427 | 315.0486 | 0.4379 | 0.9912 | 0.9912           |
| results-v2-es | Testing  | 2140074.75 | 1462.8994 | 577.0405 | 0.5454 | 0.9427 | 0.9427         |
| results-v3  | Training | 49717.34   | 222.9739 | 149.1993 | 0.3372 | 0.9954 | 0.9954            |
| results-v3  | Testing  | 725773.25  | 851.9233 | 358.3552 | 0.4243 | 0.9349 | 0.9349            |

## A live crystal ball

- **Demo**: <https://huggingface.co/spaces/alvaroinckot/tcaqs>  
- **Code**: <https://github.com/alvaroinckot/tcaqs>

Fill out character info and get an estimate. No more guesswork before you splash the guild’s treasury.

## What’s next?

1. **Fresh crawl** – New vocation (Monk) & inflation‑adjusted gold means the 2022 snapshot is aging. A distributed crawler on SQS (or any other queue) should cut scraping from weeks to days.
2. **Market context** – Inject rolling world‑level CPI so the model tracks gold inflation.
3. **Model bake‑off** – LightGBM, TabPFN, maybe a tiny tabular transformer—because why not?
4. **Public dashboard** – Daily fair‑price index per vocation/world.

## Closing XP

Shipping **TCAQS** reminded me that AI glamour rests on unglamorous foundations: stubborn scraping, vicious cleaning, and lots of waiting bars. But bringing together two long‑time passions: Tibia and machine learning, was worth every `<div>`.
