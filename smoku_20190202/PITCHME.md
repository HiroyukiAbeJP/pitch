### 今日の成果

Python高速周り検証

---
### python関係ないけど

dockerのコンテナステータス見るのに
docker statsコマンド使ってたけど
ctop初めて知った。便利

---
### 計測方法

「%timeit」コマンドを利用 

or  

start_time - finish_time  

---

### 内包表記

Q.list生成

1. 単純なfor loop ▶ 652 ms ± 4.56 ms

2. 内包表記 ▶ 530 ms ± 5.53 ms

内包表記のほうが1.2 -1.3倍くらいはやい

---

### そもそもなるべくfor文使わない

Q.list生成

1.for ▶ 150 ms ± 2.61 ms

2.map  ▶ 406 ns ± 8.75 ns

3.numpy ▶ 46.3 ms ± 599 µs

forは極力避ける

---

### whileとfor

Q.list生成

1. while ▶ 8.81 ms ± 349 µs

2. for ▶ 4.9 ms ± 215 µs


forのほうが1.8 - 2.0倍くらいはやい


---

### joinによる文字列結合

1. + ▶ 18.2 µs ± 2.33 µs

2. .join ▶ 35.2 µs ± 1.28 µs


joinのほうが2倍くらい時間かかる

---

### yieldを使用したloop

Q. フィボナッチ数列計算

1. for loop

▶ 2.13 s ± 107 ms

1. yield

▶ 4.79 µs ± 187 ns


メモリの使用量が数分の１で済む


---


### numbaを用いた高速化

what's numba ?

JIT(実行時)コンパイラを使うためのpythonライブラリ
なるべくcython使いたくないけど、高速化したい人向け

```python
from numba import jit

@jit
def function()
```

---

### JITコンパイラ

JITコンパイラとは、プログラムの実行時に、あらかじめ用意された（実行環境に依存しない汎用的な）中間コードを、プログラムの実行時点でプロセッサが実行可能な機械語（ネイティブコード）にコンパイルすることである。

---

### どれくらい高速化されるか

Q.10000✖10000の行列計算

1. 通常

▶ Time:24.166[sec]

2. jitコンパイル

▶Time:0.516[sec]


---

### 関数内で使えないもの(例)

1. 内包表記
2. 空のlist


---

### その他

1. 配列の初期化は[None]*nで
2. グローバルでの処理最小化
3. multiprocessingによるマルチスレッド処理
4. cythonに助けを求める
5. juliaに逃げる
6. お金の力（クラウドGPU）に頼る


---

### おわり


---





### 今日の成果

- やったこと
	- ドキュメント読み仕様だいたい理解できた
	- 線形回帰とロジスティック回帰両方触ってモデル作成・検定・予測とやってみた


---
### 今日の成果
- 所感
	- SQLのみで非常に簡単に実装でき、各関数も結構使いやすかった
	- 線形回帰とロジスティック回帰のみだが、ビジネス上では意外と適用できる範囲はあるので、ビジネス職種の人が普段のちょっとした分析に使う分には非常に便利そうな感じがした
	- 同じML関連のツールでも、例えばdatarobotのようなある程度専門性がある人向けのものではない
---

### 公式ドキュメントのモデル作成クエリ

```sql
#standardSQL
CREATE MODEL `bqml_tutorial.sample_model`
OPTIONS(model_type='logistic_reg') AS
SELECT
  IF(totals.transactions IS NULL, 0, 1) AS label,
  IFNULL(device.operatingSystem, "") AS os,
  device.isMobile AS is_mobile,
  IFNULL(geoNetwork.country, "") AS country,
  IFNULL(totals.pageviews, 0) AS pageviews
FROM
  `bigquery-public-data.google_analytics_sample.ga_sessions_*`
WHERE
  _TABLE_SUFFIX BETWEEN '20160801' AND '20170630'
```

---


### モデル作成用のクエリ

```sql
# ロジスティック回帰（id昇順の上位５００人で学習）
create or replace model `database.dataset.sample_ml_logistic`
options
(model_type='logistic_reg', input_label_cols=['label']) as 
select
 Survived as label,
 Pclass,
 Age,
 Sex,
 SibSp,
 parch,
 Embarked,
 fare
 from  `database.dataset.tit_train`
 order by PassengerId asc
 limit 500
 ```

---


### モデルの統計情報

- 項目
	- トレーニングデータの損失：logless。今回の場合は[cross entropy（対数損失）](https://en.wikipedia.org/wiki/Cross_entropy#Cross-entropy_error_function_and_logistic_regression)
	- 評価データの損失：ホールドアウトのデータセットにおけるlogless
	- 学習率
	- 完了時刻（秒）	

---


###検証用のクエリ

```sql
select *
from ml.evaluate(model `database.dataset.sample_ml_logistic`,
(
select
 Pclass,
 Age,
 Sex,
 SibSp,
 parch,
 Embarked,
 fare,
 Survived as label
 from  `database.dataset.tit_train`
 order by PassengerId desc
 limit 300
))
```

---


### 各種検定値

- ロジスティック回帰を使用しているため、結果には次の列が含まれます。
- precision - 分類モデルの指標。陽性のクラスの予測でモデルが正しかった確率を表します。
- recall - 次の質問に回答する分類モデルの指標: すべての陽性のラベルの中でモデルが正しく識別したラベルの数
- accuracy - 分類モデルが正解した予測の割合。
- f1_score - モデルの精度を表す尺度。f1 値は適合率と再現率の調和平均を表します。最も精度の高い f1 値は 1 で、最も精度の悪い値は 0 です。
- log_loss - ロジスティック回帰で使用される損失関数。モデルの予測が正しいラベルからどのくらい離れているかを表します。
- roc_auc - ROC 曲線の下の面積。これは、無作為に選択した陽性のサンプルが陽性に分類される確率が、無作為に選択した陰性のサンプルが陽性に分類される確率よりも高い可能性を表します。詳細については、機械学習集中講座の分類をご覧ください。

---


### 他にも

- ML.ROC_CURVE
- ML.CONFUSION_MATRIX
- ML.WEIGHTS

なんかが用意されており、基本的な検定指標はぱっと見ることが可能
※予測用は「ml.evaluate」「ml.predict」に変更するだけ


なんかが用意されており、基本的な検定指標はぱっと見ることが可能
