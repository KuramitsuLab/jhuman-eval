# JHumanEval: Japanese Hand-Translated HumanEval

This is a Japanese translated version of HumanEval, an evaluation harness for the HumanEval problem solving dataset described in the paper "Evaluating Large Language Models Trained on Code".

LLMのコード生成能力の標準ベンチマーク HumanEval の日本語翻訳版です。

機械翻訳(DeepL, GPT4)の翻訳結果を全て人手によって再修正し、
訳文を日本人のプログラマが読んで理解してコードが書けそうかチェックしてあります。
ただし、英語版 HumanEval の間違いは、修正せずに残して、
HumanEval同様に不完全なドキュメントからの生成能力を見るようになっています。

日本語LLMのベンチマークとしてお使いください。

# 使い方

データはJSONL形式で配布しています。

HuggingFaces のデータセットで公開する前は、ファイルから読み込んでください。

```py
from datasets import load_dataset
ds = load_dataset('json', data_files='jhuman-eval.jsonl.gz', split='train')
```
```
Dataset({
    features: ['task_id', 'prompt_en', 'prompt', 'entry_point', 'canonical_solution', 'test'],
    num_rows: 164
})
```
注意: 不具合の報告がなければ HuggingFace datasets から公開する予定です。

## データ形式

データ形式は、本家HuggingFace と同じです。

- `task_id`: identifier for the data sample
- `prompt`: input for the model containing function header and docstrings
- `canonical_solution`: solution for the problem in the prompt
- `test`: contains function to test generated code for correctness
- `entry_point`: entry point for test

英文プロンプトは`prompt_en`に残してあります。

## 評価方法

HumanEval と同じく、HuggingFace evaluate を使うといいでしょう。

```python
import os
os.environ["HF_ALLOW_CODE_EVAL"] = "1"

from evaluate import load
code_eval = load("code_eval")
pass_at_k, results = code_eval.compute(references=test_cases, predictions=candidates, k=[1])
print(pass_at_k)
```

テスト用: 参照コードでpass@1を計算するためには：

```python
candidates = []
test_cases = []
for d in ds:
    # FIXME: 参照コードをそのまま入れているが、予測コードに置き換えるべき
    candidates.append([d['prompt']+d['canonical_solution']])
    # テストケースを作る実行可能な形式にする
    test_cases.append(d['test']+f"\n\ncheck({d['entry_point']})\n")
```


