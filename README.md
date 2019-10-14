# 因果推論を実践するために
## 因果推論でやりたいこと
- 背景要因(共変量)が同じデータ間でキャンペーンを経験したorしていないで比較したい
    - 背景：年齢、性別、ゲームどれだけやってるか、自社や他社のサービスどれだけやってるか、広告をどれだけ見たかなど
- テスト群：施策(z：キャンペーン)を実施したか、コントロール群：施策(z:キャンペーン)を実施していない、とする
- 共変量(x)が同じテストサンプル(z=1)とコントロールサンプル(z=0)を見つけて、施策効果(y)の差を計算

## 因果推論実施の条件
- 各要素の目的変数はに各要素に干渉せず、独立である
- *共*変量xが同じであれば、割り当てzと目的変数(y1,y0)(y1,y0)の同時分布は独立


## 手順
1. 共変量が似ているかどうかの算出
2. 共変量が似ているサンプル同士のマッチング
3. マッチングしたサンプル同士の結果変数の差を計算し介入の効果とする


### 1. 共変量が似ているかどうかの算出
- 傾向スコア(Propensity Score)
- マハラノビス距離
- IPW (Inverse Probability Weighting: IPW): 傾向スコアの逆数を重みとした重み付け方を用いた推定量
  - 割り当てzが共変量xからバイアスを受けている可能性があるから、普遍推定量を探してくる必要がある、それがIPW推定量
- Proximity: 気を構築する際にデータ同士が末端nodeで同じnodeに何回入ったか

今回は傾向スコア(Propensity Score)を活用する場合に着目。

####  傾向スコアの算出
- モデリング
- モデルの評価

##### モデリング
- ロジット、RF、GBMなど何でも良い

##### モデルの評価
###### AUC(c統計量)
- AUCとテスコン間の共変量の分布を見る
#### テスコン間の傾向スコアの分布: 左右対称になっているか確認
仮に、傾向スコアが正しく振れているならば
- 傾向スコア0.1 (テスト群のデータ：コントロール群のデータ) = 1 : 9
- 傾向スコア0.5 (テスト群のデータ：コントロール群のデータ) = 5 : 5
- 傾向スコア0.9 (テスト群のデータ：コントロール群のデータ) = 9 : 1
となるはず。だから、ヒストグラムを書いて確認する。
傾向スコアが同じ値をとるデータ間の共変量の分布」も見ておきたい。

#### 共変量に傾向スコアの逆数を掛けた際に、テスト / コントロール群の分布が平らに補正されるか
- 全ての共変量に対して確認するのは無理だから、重要な変数のみ確認で実践


### 2. 似ているサンプル同士のマッチング
- Nearest Neighbor: 一番傾向スコアが近いもの同士をマッチング
- caliper: マッチングする際にある一定以上傾向スコアに差がある場合にはマッチングしない
- 層別マッチング: 傾向スコアが「ある値以上ある値未満」のグループをK個作成し、そのグループ同士の目的変数yの値をの差の比較を行う
- テスト群のデータがコントロール群に比べて少ない問題設定では、1:1マッチを行うと多くのコントロール群のデータがマッチングに用いられないので、検出力が下がることは明らかです。また、テスト群とコントロール群で傾向スコアのヒストグラムの傾向が異なれば、マッチングができるデータ数が少なくなる

### 3. マッチングしたサンプル同士の結果変数の差を計算
計算する。







## 因果推論をする際に注意すべきこと
- そもそも、ランダム化比較試験(RCT)をしましょう、はだめなんでしたっけ？


### 背景要因を揃えられるかどうか？
- 共変量の次元が大きくなると類似度の高いテスト/コントロール群を抽出する事例もある。->探したい
- ランダム化比較試験(RCT：Randomized Control Trial)に対して施策の割り当てができることが理想(A/Bテスト)
- テスト/コントロール群のデータに偏りが生じている場合

### 仮想現実(counterfactual)
- 満たすべき2つの条件
  - SUTVA
  - 強く無視できる割り当て条件

#### SUTVA
- SUTVA(Stable Unit Treatment Value Assumption)条件、各要素の目的変数は各要素に干渉せず、独立である」
- あるプロダクトの売上がめちゃくちゃ高かった場合に、他のプロダクトの売上などに影響が出てはいけない

#### 強く無視できる割り当て条件
- 「共変量xが同じであれば、割り当てzと目的変数(y1,y0)の同時分布は独立」
- 割り当てがされる確率は、あくまでも共変量のみに依存し、目的変数には依存しない
-　例えば、年齢 / 性別 / 普段のTVの視聴時間などの共変量のみが、CMを見るかに依存する
-　つまり共変量(年齢性別など)が同じ個体であれば、割り当て(CMを見る)される確率P(z)は同じ。

同じ共編量xを持つデータは、割当zの有無のみで目的変数が変化する。
これが成り立っているかどうかはデータ見るしか無い。



「テスト / コントロール群で同じような傾向スコアを持つデータペアを作って、目的変数yの差の平均が効果」

## サマリ
- 傾向スコアe(x)：P(z=1|x)。xの元で割り当てっぽい確率
  - 実践としては、割当られているかどうかを目的変数に、xを説明変数として、統計モデルで確率出す
- 強く無視できる割り当て条件の元で、似た背景を持つデータであることの指標として傾向スコアを用いる
- 求めたい効果：テスト群のe(x)に近いコントロール群のデータのペアを作り、ペア間の目的変数yの差の期待値


** あるテスト群のデータに対し同じ傾向スコアを持つコントロール群データを「もし割り当てがなかったときのデータ」 **





### マッチングの評価
- 「このマッチングはどれくらい良いマッチングであるか」を評価する手法として standardized difference (SD)
- SDは「ペアリングに用いられたテスト / コントロール群のデータの共変量に差がないか」を表す指標






## 概念とか語彙とかもろもろ
- 相関関係：ある一方の変数の値が大きい時、他方の変数の値も大きい
- 因果関係：要因Xを変化させた時に要因Yも変化する
- 原因変数：原因を示す変数
- 結果変数(アウトカム)：結果を示す変数
- 介入/処置：要因を変化させること
- テスト群/コントロール群：施策を受けた群/受けなかった群
- 共変量：背景要因、つまり介入される以外の要因
- 割り当て(treat)：介入を受ける要因
- 交絡/交絡因子/疑似相関
- 傾向スコア：テスト/コントロール群のデータ同士をマッチングする際に使うスコア
- SUTVA
- 強く無視できる割り当て条件




## 参考
https://qiita.com/usaito/items/995bb9b6833f7d9e2178
https://qiita.com/deaikei/items/df3626486986566cb65c

### ドナルド・ルービンの因果モデル
- [central role of the propensity score in observational studies for causal effects | Biometrika | Oxford Academic
](https://academic.oup.com/biomet/article/70/1/41/240879)
- 潜在的なアウトカム(Potential outcome)
