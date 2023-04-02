---
title: "Japanese-Alpaca-LoRAをGCPのVMインスタンスで試す"
date: 2023-04-03T01:00:00+09:00
draft: false
---

メモです。

## tl;dr
結論、とりあえず動かすなら `alpaca-lora` をクローンしてきて動かすのが簡単。

- https://github.com/tloen/alpaca-lora をクローンする
- `python generate.py --base_model='decapoda-research/llama-7b-hf' --lora_weights=="kunishou/Japanese-Alpaca-LoRA-7b-v0`
`:7860` ポートで待ち受けてた。クラウド使うときはファイアウォールが空いてるか確認して自前のPCからアクセスできる。

## 実験
とりあえず動かすところが目標です。

### インスタンス

```
NVIDIA T4 + n1-highmem-16 (16 vCPU, 104GBメモリ) + 256GB SSD

Debian 10 based Deep Learning VM with M105
Base CUDA 11.3, Deep Learning VM Image with CUDA 11.3 preinstalled.
```
asia-east1(台湾)リージョンが比較的安いためおすすめです。スポットインスタンスの条件付きだが月間216ドルで利用できる。一時間あたり0.30ドル。

FlexGenを試したときにめちゃくちゃメモリを利用した記憶があったため104GBを利用したが、TransformerではGPUメモリを主に利用するようで、物理メモリは3GB程度しか利用しなかった。CPUもほとんど利用していないので、コンピューティングリソースは割と節約できそう。

いろんなモデルを試すならストレージは大き目の方が良いです。LLaMA-65Bにもなるとモデルだけで243GB必要です。

イメージはGCPデフォルトのCUDA付きイメージを使用。最初さらに新しいイメージがあったのでそれを利用したが、バージョンの違いかCUDAのインストール後に再起動しなかったためかバージョンエラーが出て起動できず。

### まずは普通に起動させる
難しいことはありません。

```
pip install -r requirements.txt
python generate.py --base_model 'decapoda-research/llama-7b-hf' --lora_weights 'tloen/alpaca-lora-7b'
```

待ち受けポートが表示されるので、アクセスして英語版を触ってみてもいいかもしれません。GCPだとデフォルトでファイアウォールがONなので、通過する設定をする必要があります。

### Japanese-Alpaca-LoRAを使うように変更

引数を変えるだけで変更できます。

`python generate.py --base_model='decapoda-research/llama-7b-hf' --lora_weights=="kunishou/Japanese-Alpaca-LoRA-7b-v0`

### 日本語を7Bで試してみる
パラメーターはデフォルトでベースモデルは7Bを利用。ただし `Max tokens` はGPUのメモリが足りなくなるため100に下げています。
生成に掛かる時間はそれぞれ7秒弱でした。

```
> JavaScriptで配列の数字を二倍にして返す関数を作って。

function doubleArray(arr) {
  for (let i = 0; i < arr.length; i++) {
    arr[i] *= 2;
  }
  return arr;
}
```

```
> Goで配列の中から最大の数を返す関数を書いて。
def max_number(list):
    max_num = 0
    for num in list:
        if num > max_num:
            max_num = num
    return max_num
```

コード周りはまぁまぁ意味が伝わっていそうな回答が帰ってきます。二つ目はどう見てもGoじゃなさそうですが。

```
> フットボールとサッカーの違いは？

フットボールとサッカーの主な違いは、フットボールは11人当たりの球を使用し、サッカーは11人当たりの球を使用します。また、フットボールは2分間の間、サッカーは4分間の間で行われます。また、フットボ
```

```
> アルパカについて教えて

アルパカは、アルプスの中で最も高い山です。18,000フィート（5,500メートル）の最高峰があり、アルプスの中で最も高い山です。
```

```
> アルプスの中で最も高い山は？

アルプスの中で最も高い山は、エッセンス山です。
```

```
> アメリカの大統領は？

アメリカの大統領は、President of the United States（米国大統領）です。
```

```
> 2019年のアメリカの大統領は？

2019年のアメリカ大統領はドナルド・トランプです。
```

回答の精度はん？と思うこともありますが、7Bかつ日本語にしては割と良い返答がされている気がします。

### リソース状況

GPUメモリをめちゃくちゃ使ってますね。

```
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 510.47.03    Driver Version: 510.47.03    CUDA Version: 11.6     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|                               |                      |               MIG M. |
|===============================+======================+======================|
|   0  Tesla T4            Off  | 00000000:00:04.0 Off |                    0 |
| N/A   55C    P0    28W /  70W |  14229MiB / 15360MiB |      0%      Default |
|                               |                      |                  N/A |
+-------------------------------+----------------------+----------------------+

+-----------------------------------------------------------------------------+
| Processes:                                                                  |
|  GPU   GI   CI        PID   Type   Process name                  GPU Memory |
|        ID   ID                                                   Usage      |
|=============================================================================|
|    0   N/A  N/A     27621      C   python                          14227MiB |
+-----------------------------------------------------------------------------+
```

## 注意
どうせこういうのはモデルをあらかじめ落としておかないと使えないんだろ？と思って先に `decapoda-research/llama-7b-hf` のようなモデルを落とすと、明らかに分割された下みたいなファイルを入手できます。

`pytorch_model-00001-of-00033.bin ~ pytorch_model-00033-of-00033.bin`

このファイルはPythonで簡単なスクリプトを書けば結合させることができますが、そもそもtransformerで利用するLLaMAやAlpaca等のHuggingFace上でホスティングされているモデルについてはtransformerが自前で引っ張ってきてくれるので結合させたり、手動で読み込ませたりする必要はありません。

これに気がつくまでかなり時間がかかってしまったので、同じような思考をした人がこの記事に引っかかるように検索したワードを置いておきます。

```
join pytorch_model-00001-of-00033.bin
join pytorch_model-00001-of-00033.bin
ggml-model.bin
pytorch_model.bin.index.json
```

また、最初はWindowsで動かすよう頑張っていましたが、CUDAをうまく認識してくれず断念しました。どちらにしろこのメモリ利用量だと動かなかったと思います。

## Next

`alpaca.cpp` や `llama.cpp` 等、LLMを軽量で利用できそうなプロジェクトができてきていますので、近々それに挑戦していきたいと思います。

ChatGPTのそれとは違い、LLaMAのようなモデルはナマモノなので利用には注意が必要な部分はありますが、そういったモデルでもローカルや個人VMであれば楽しく利用することができます。