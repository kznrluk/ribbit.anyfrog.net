---
title: "LLM周りで頻出する単語メモ"
date: 2023-03-27T15:00:00+09:00
draft: false
---
書きかけ。

https://note.com/kun1emon/n/n1533345d5d26

## LLM
大規模言語モデル

### GPT

### OPT

### LLaMA
FaceBook製LLM。

### Alpaca
LLaMAをスタンフォード大学がファインチューニングするプロジェクト。

[tatsu-lab/stanford\_alpaca: Code and documentation to train Stanford's Alpaca models, and generate the data.](https://github.com/tatsu-lab/stanford_alpaca)

LLaMAの7B程度のモデルでもtext-davinci-003(17.5B)と似た出力をする。

AlpacaはLLaMAをtext-davinci-003が生成したデータでSelf-Instructしたモデルであり、LLaMAのLoRAバージョンがAlpacaというわけではない。LLaMAのLoRAバージョンはAlpaca-LoRA。

[Alpaca まとめ｜npaka｜note](https://note.com/npaka/n/n1a0ab681dc70)

### Alpaca-LoRA
AlpacaをLoRAを用いて再現するプロジェクト。AlpacaがFine Tuningに利用したデータ `alpaca_data.json` を元に、LoRAで再現をする。

[Alpaca-LoRA まとめ｜npaka｜note](https://note.com/npaka/n/n88392b28dd2b)

### Alpaca-LoRAを日本語の文脈でよく見る謂れ
LLaMAやGPT, OPTは計算コストが高いため、個人で実験するには大変。LLaMAを軽量化したAlpaca、Alpacaをさらに軽量化したAlpaca-LoRAになってようやっと個人で実験が可能になってきた。

雑なイメージ、知識を大量に保有するでかい言語モデルであるLLaMAを、別のAIで生成した命令文を使って出力をよりよく、計算リソースも効率化しようとしたのがAlpaca...なはず。

### Self-Instruct
[[2212.10560] Self-Instruct: Aligning Language Model with Self Generated Instructions](https://arxiv.org/abs/2212.10560)

* Alpacaで使われている

## LoRA low-rank adaptation
低ランク適応。大規模なモデルの出力を使って小規模なモデルを学習(?)させ、計算に必要なコストを削減する手法。

* Alpaca-LoRAで使われている

ファインチューニングしやすい

## Fine Tuning
[実装方法から読み解くファインチューニングと転移学習の違いとは - DATAFLUCT Tech Blog](https://tech.datafluct.com/entry/20220511/1652194800)
