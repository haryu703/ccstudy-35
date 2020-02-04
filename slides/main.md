---
marp: true
---

# Avalanche の紹介

### haryu703

---

# 自己紹介

[haryu703](https://twitter.com/haryu703)

興味
- Bitcoin Cash
- Rust

---
# Snowflake to Avalanche: A Novel Metastable Consensus Protocol Family for Cryptocurrencies
- Team Rocket
- t-rocket@protonmail.com
- Revision: 05/16/2018 21:51:26 UTC

---

# Abstract

- 新しい BFT プロトコル作った
- このペーパーでは 3 つのプロトコルからなるプロトコルファミリーについて
  - プロトコルの保証を分析する
  - 使い方を説明する
- 1300tps, 4sec/confirmation 出せる

---

# 1. Introduction

- 従来のコンセンサスプロトコルについて
  - 参加者全員で合意をとる
  - $O(n^2)$ のコミュニケーションが必要
  - 多数の参加者では難しい
- Nakamoto コンセンサスについて
  - Bitcoin で使われている
  - マイニングが必要
  - スケーラビリティの課題

---

- 提案するプロトコルファミリーについて
  - [ゴシップアルゴリズム](https://ja.wikipedia.org/wiki/%E3%82%B4%E3%82%B7%E3%83%83%E3%83%97%E3%83%97%E3%83%AD%E3%83%88%E3%82%B3%E3%83%AB)に影響を受けている
    - 参加者間で繰り返し情報を交換する手法
  - 参加者をネットワークからランダムに繰り返しサンプリングする
  - PoW しない
  - $O(kn \text{log} n)$ から $O(kn)$ のコミュニケーションで済む
    - $k$ はセキュリティパラメータ
  - ただし、トランザクションの「正しさ」は保証される必要がある
    - 暗号資産の場合、発行者は署名で保証されている
    - 不正なトランザクションが発行されても安全性に影響はない

---

# 2. Approach

1. Slush(雪泥) (non-Byzantine protocol)から始まり、
1. Snowflake(雪片)
   ↓
1. Snowball(雪玉)
   ↓
1. Avalanche(雪崩)
   という構成になっている

---

- 3 つの前提がある
  - Bitcoin のような確率的な安全性
    - 従来と同じで、パラメータ次第
  - 全てのトランザクションの順番は決めない
    - トランザクションに関連する部分だけ決める
  - 不正なノードの動作は保証しない
    - 最終的には正常なノードが動作する

---

## 2.1. Model, Goals, Threat Model

- 定義
  - $\mathcal N$ ... ノード
  - $\mathcal C$ ... 正直ノード
  - $\mathcal B$ ... 不正ノード
- Bitcoin の UTXO モデルで考える
  - 不正なトランザクションは署名によって防がれる
  - 衝突するトランザクションは発行できる（二重支払い）
    - $\mathcal C$は発行しないけど$\mathcal B$は発行する
- Avalanche は以下の保証を高確率で保証する
  - Safety ... 競合するトランザクションを受け入れる 2 つの$\mathcal C$がない
  - Liveness ... $\mathcal C$が発行したトランザクションは全ての$\mathcal C$が受け入れる
- $1-\epsilon$の確率で安全性が維持される
  - $\epsilon$はシステム上のセキュリティパラメータ

---

## 2.2. Slush: Introducing Metastability

- non-Byzantine protocolだが、この後のプロトコルの説明に役立つ
- 競合するトランザクションを「Red(R)」と「Blue(B)」の 2 色に置き換えて説明する
  - 初期状態は無色

### 手順

1. ネットワークから$k$個のノードをランダムに抽出する
1. 抽出したノードに「私の色は(R|B)だけどあなたの色は？」と問い合わせる
1. 問い合わせられたノードは自身の色を返す
   - 自身が無色だったら、受け取った色で更新して返す
1. $k$個のうち、$\alpha k$個のノードが同じ色なら、自身の色をその色に更新する
   - $\alpha > 0.5$はプロトコルのパラメータ
1. 1-4 を$m$回繰り返して色を決める

---

![h:700](https://raw.githubusercontent.com/haryu703/ccstudy-35/master/images/slush_protocol.png)

---

- 利点
  - 省メモリ
  - コミュニケーション対象のノードが少ない
  - 最初の色の割合が 5:5 だとしても、ランダムな抽出のおかげで片方に偏っていく
  - $m$が十分大きければ全てのノードは高確率で同じ色になる
- $m$はノード全体の数に対して対数的に増える
- $\mathcal B$は問い合わせに対して自分に有利な色を返すことで攻撃できる

---

## 2.3. Snowflake: BFT

- Slush の拡張
- 「問い合わせ結果が連続して自分と同じ色だった回数」$cnt$を追加する
  - 結果が自分と違う色だったらリセットする
- $cnt > \beta$になるまでラウンドを繰り返す
  - $\beta$はセキュリティパラメータ
- これで safety と liveness を満たせる
- $\mathcal B$はある時点から色を制御できなくなる

---

![h:700](https://raw.githubusercontent.com/haryu703/ccstudy-35/master/images/snowflake_protocol.png)

---

## 2.4. Snowball: Adding Confidence

- Snowflake の拡張
- $cnt$は状態として短命
- 「問い合わせ結果の色の出現回数」$confidence\ counters$を追加する
- 色の出現回数が片方を上回ったら自分の色を更新する
  - 更新するだけで、受け入れは Snowflake と同じ
- 攻撃耐性の強化だけでなく、プロトコルに一般化しやすくなる

---

![h:700](https://raw.githubusercontent.com/haryu703/ccstudy-35/master/images/snowball_protocol.png)

---

## 2.5. Avalanche: Adding a DAG

1. Snowball を一般化する
1. Snowball の$confidence\ counters$の代わりに DAG を使う

- 効率面
  - 一つの頂点への投票が暗黙的に頂点までのパス全てへの投票になる
- セキュリティ面
  - 過去の決定を取り消しづらくなる
- 定義
  - $ancestor\ set$ ... トランザクションの全ての先祖トランザクション
  - $progeny$ ... トランザクションの全ての子孫トランザクション
- 「トランザクションの衝突」はアプリケーション次第
  - 我々のアプリケーションでは「二重支払い」のこととする

---

- ノードはトランザクションの問い合わせに対して$ancestor\ set$込みで判定し応答する
- 閾値$\alpha k$以上の投票があったらトランザクションの$chit$($c_{uT}$)を 1 にする
- トランザクションの$progeny$の$chit$を合計した値($confidence$)を計算する
- ノードはトランザクションを一度だけ問い合わせ、$progeny$に追加して$confidence$を作っていく
- 最初に見つかったトランザクション次第で DAG の関係は壊れる（？）

---

![h:700](https://raw.githubusercontent.com/haryu703/ccstudy-35/master/images/avalanche_protocol.png)

---

![h:700](https://raw.githubusercontent.com/haryu703/ccstudy-35/master/images/avalanche_tx_gen.png)

---

![h:700](https://raw.githubusercontent.com/haryu703/ccstudy-35/master/images/avalanche_voting.png)

---

## 2.6. Avalanche: Specification

- $\mathcal T_u$ ... ノード$u$が持つ全てのトランザクションのセット
- $\mathcal P_T$ ... $T \in \mathcal T_u$ のコンフリクトセット
- $T' \leftarrow T$ ... $T$は$T'$の親トランザクション
- $T' \xleftarrow{*} T$ ... $T$は$T'$の先祖トランザクション
- Avalance は Bitcoin と同様に、トランザクションの確定はしない

---

![h:700](https://raw.githubusercontent.com/haryu703/ccstudy-35/master/images/avalanche_dag.png)

---

# 3. Analysis

- Slush、Snowflake、Snowball、Avalanche の分析を行う

---

## 3.1. Analysis of Slush

- Slush はビザンチンノードがいない状態で動作する
- 以下の 3 つを示す
  - R1) ほぼ確実に全てのノードが有限時間で同じ色に同意する状態に収束する
  - R2) 収束の速度の閉形式を提供する
  - R3) $1-\epsilon$の確率で合意するための最小ステップ数を特徴付ける
- マルコフ連鎖を使って解析する

---

## 3.2. Safety Analysis of Snowflake

- Slush との違い 3 つ
  - ビザンチンノードがいる
  - 各ノードは連続して同じ色だった回数を覚える
  - $\mathcal A$という関数を導入する
    - ノード全体の空間と時間$t$に選ばれたノード$u$を受け取る
    - $\mathcal B$のセットを任意の色の構成に変更する
- ビザンチンノードは、50/50 に分裂した状態を抜ける閾値を補正する機能を持つ
- $s_{ps} > s_{c/2}$になるとビザンチンノードの制御を失う

---

- Snowflake の安全性を保証する 2 つの状態を分析する
  - C1) $s_{ps+\Delta}$を超えたらビザンチンノードが存在しても状態が覆らない必要がある
  - C2) システムが高確率で覆らない状態になったときにのみ、プロセスが色にコミットできるメカニズムを構築する必要がある
- 分析の前にサンプルサイズ$k$の特徴を紹介する
  - 超幾何分布が十分大きなネットワークサイズの二項分布に似る
    - Snowflake と後続のプロトコルに影響がある
    - $k$が十分大きい場合、システムのセキュリティは全てのネットワークサイズで等しくなる
  - $k$が$n$に近づくと$s_{ps}$は指数関数的に$s_{c/2+b/2}$に近づく

- 適切なパラメータのもとで$C1$と$C2$が満たされれば、2 つのノードが違う色を選ぶ確率は$\epsilon$未満になる

---

# 3.3. Safety Analysis of Snowball

- Snowball が Snowflake よりも安全であることを示す
- Snowball は R と B の$confidence$を増加させて持つ点が Snowflake との主な違い
  - ランダムな不安定さやビザンチンノードの影響が制限される

---

- 具体例を挙げて説明する
  - 下記のネットワークパラメータで考える
    - ノード数: $n$
    - ビザンチンノード数 $b$: $(1/5)n$
    - 自身の色を更新する閾値 $\alpha$: $(4/5)k$
    - 正しいノードは最低$(3/5)n$が R を選ぶ
  - ビザンチンノードが R を選ぶと$(4/5)n$は R を支持する
  - ビザンチンノードが B を選ぶと$(3/5)n$は R を支持する
  - 正しいノードは$(1/5)n$〜$(2/5)n$が B を支持することになる
  - $(3/5)n$以上の正しいノードは R の$confidence$を伸ばす
  - R の$confidence$は B よりも成長しやすくなる

---

- Snowflake の安全性に必要な 2 つの状態について
  - C2 は Snowball でも同じ
  - C1 は再分析が必要

### Security of Snowball.

- C1 については、Snowflake よりも厳密にセキュリティが高い
- R と B の$confidence$の差は数ステップで広がっていき、覆ることがほぼなくなる
- Snowflake で適用したパラメータは Snowball にも適用できる

---

## 3.4. Safety Analysis of Avalanche

- トランザクション$T_i$の DAG に問い合わせをする
- $T_i$の祖先全体に暗黙的に問い合わせている
- Avalanche は Snowball のインスタンスにマップされる
- liveness は Snowball とは異なる
  - 正しいトランザクションかどうかは、その親に依存している
  - 次の 2 つのセクションで説明する

---

## 3.5. Safe Early Commitment

- 色を確定するためのパラメータについて
- 11 ノードくらい同じ色だったら確定とみなせる（？）

---

## 3.6. Liveness

### Slush

- Slush は BFT プロトコルではない
- 有限数のラウンドでほぼ確実に終了する

### Snowflake & Snowball

- カウンターを使い、より投票された方を追跡する
- ビザンチンノードは衝突するトランザクションを偽造できない
- ビザンチンノードが取れる不正行動は、問い合わせの回答を拒否すること
- 問い合わせがタイムアウトするとノードが選び直される
- 有限数のラウンドでほぼ確実に終了する

---

### Avalanche

- DAG 構造を使っている
- 未確定のトランザクションを親に選ぶと、親が不正だった場合が危ない

---

- 以下のメカニズムで改善できる
  - Eventually good ancestry
    - トランザクションは親を変更してリトライできる
  - Sufficient chits
    - 正しいトランザクションに nop トランザクションを子孫として発行する
    - 決定された親を持つ正しいトランザクションが、十分な$chit$を受け取るために必要
    - nop トランザクション
      - アプリケーションに副作用がない
      - どのノードからも発行できる
      - ビザンチンノードから悪用されることはない（？）
  - 上記２つのメカニズムがあれば、最悪でも Snowball 並み

---

## 3.7. Churn and View Updates

- 今までは簡単のため、ノードの追加や離脱については考えていなかった
- ノードの追加や離脱があっても Avalanche が安全に動くことを示す
- $\gamma$だけノードの追加と、$\bar{\gamma}$だけノードの削除ができる
  - $\gamma$と$\bar{\gamma}$はシステムのパラメータ
- ネットワークが切り替わる間について考慮する必要がある
  - 簡単な方法は、次のノード全体の状態が安全になるまで今の状態を確定しない（？）
    - 今の状態が安全と判断できたら一つ前の状態を確定させる？
    - この方法は確定までが遅い
  - 正確な方法はおいといて、代わりにメンバーシップオラクルに依存する

---

## 3.8. Communication Complexity

- 複雑性については正しいトランザクションのみ考える
  - liveness は不正なトランザクションについて保証しないため
- Snowflake と Snowball は$O(kn \text{log} n)$メッセージで終わる
- Avalanche は$O(kn)$

---

# 4. Implementation

- Bitcoin のトランザクションを Avalanche に移植した
- Avalanche で値を転送することについて焦点をあてる

---

## UTXO Transactions

- 二重支払いを Avalanche で防ぐ
- TxIn が一つの場合、衝突するのも一つ
- TxIn が複数ある場合、全ての TxIn について衝突しないことを確認する
- 複数のトランザクションをまとめて問い合わせできるように実装する

---

## Parent Selection

- 優れた親決定アルゴリズムは、DAG を安定した幅で成長させる
  - 木のように発散したり、チェーンのように収束したりしない
- 親の選択アルゴリズムにはトレードオフがある
  - 受け入れられた親を選択するとサポートされやすい
  - 投票希釈（マイノリティがいなくなること？）につながる
- 新しい親を選択すると、親が不正だった場合にトランザクションがスタックする

---

- 簡単な方法は、最も優先されているトランザクションからランダムに親を選択すること
  - 古いトランザクションが親に選ばれやすくなる
  - DAG が発散する
  - 新しいトランザクションの$confidence$が育たない
- 新しいトランザクションを選択して DAG の形状を維持するには別の問題がある
  - 新しいと十分な$confidence$がない
  - ネットワークに十分に分散されてない
- 最適な親は新しいトランザクションの近くかつ、古すぎないところにある
- いい感じのアルゴリズムは、DAG の先端からさかのぼっていい親を探す

---

## Optimizations

- システムのスケールのためにいくつか最適化をした
- DAG の遅延更新を行う
  - $confidence$の再帰的な定義で、DAG を横断する必要があるため
- 各コンフリクトセットのコンテナデータ構造を保持する代わりに、各 UTXO からコンフリクトセットの代表としての優先トランザクションへのマッピングを作成する（？）
  - 不正なノードは大量のコンフリクトセットを生成できるため
  - 将来の衝突やクエリへの応答を高速化できる
- $\alpha k$の閾値に達したら、$k$個の応答を待たずにクエリを終了する
  - クエリを高速化できる

---

# 5. Evaluation

- 5000 行くらい C++で実装した
- Amazon AWS で大規模に展開して、下記について他のシステムと比較する
  - スループット
  - スケーラビリティ
  - レイテンシ

---

## 5.1. Setup

- 数百〜数千の EC2 インスタンスで実験した
  - AWS は最大 2Gbps 出るけど、Avalanche は最大 36Mbps だけ使う
- トランザクションとか secp256k1 とかは Bitcoin 0.16 ベース
- 各ノードがウォレットを持ってトランザクションを生成することでシミュレートする
- トランザクションサイズは 600 バイトくらい
- 10 個のトランザクションをまとめて問い合わせる
- 一部のトランザクションはレイテンシが非常に長くする

---

## 5.2. Throughput

- 1800tps 以上出た
- Bitcoin 0.16 の場合
  - 生成は 2400tps
  - 検証は 12300tps

---

## 5.3. Scalability

- ノード数を 125 から 2000 まで変化させた
- 10%くらい落ちた
- 全てのトランザクションを線形化する従来の BFT より平行性がいい（？）

---

## 5.4. Latency

- 送信された瞬間から受け入れられたことが確認されるまでにかかる時間のこと
- 全てのトランザクションが約 1 秒以内に確認されている

---

## 5.5. Misbehaving Clients

- 不正なトランザクションがレイテンシにどのように影響するか調べる
- 0〜25%の pending のトランザクションが衝突するようシミュレートする
- 1000 ノードで実験した
- 影響はわずかだった
- 不正なトランザクションが増加するとレイテンシが低下した
  - 全体的な実効スループットが低下して、システムの負荷が下がるため

---

## 5.6. Geo-replication

- ノードが地理的に分散した状態をエミュレートして評価した
- 2000 ノードを実際の Bitcoin ノードが多い都市と同じくらいの通信速度にした
- 平均 1312tps 出た
- レイテンシの中央値は 4.2 秒、最大値は 5.8 秒

---

## 5.7. Batching

- バッチ処理の効果を確認した
  - デフォルトで 10 トランザクションをバッチ処理している
  - バッチ処理を無効化して比較した
- バッチ処理した場合でもスループットは 2 倍にしかなっていなかった
  - 現状の実装ではトランザクションの検証がボトルネックになっているため
  - `verify()`関数をコメントアウトしたら 8000tps に上がった
  - 検証に GPU 使ったりしたら改善できる

---

## 5.8. Comparison to Other Systems

- Algorand（PoS）と Bitcoin（PoW）と比較する
- リーダー
  - Algorand: リーダーベース
  - Avalanche: リーダーレス
- 20%のビザンチンノードがいる状態での安全性保証の確率
  - Algorand: $5 \times 10^{-9}$
  - Avalanche: $10^{-9}$

---

- スループット
  - Bitcoin: 3-7tps
  - Algorand: 364tps(10MB ブロック)、159tps(2MB ブロック)
  - Avalanche: 1300tps
- レイテンシ
  - Bitcoin: 10〜60 分
  - Algorand: 50 秒（10MB ブロック）、22 秒（2MB ブロック）
  - Avalanche: 4.2 秒

---

# 6. Related Work

- Bitcoin とか BFT とかの紹介

# 7. Conclusions

- まとめ

---

# Bitcoin Cash で提案されている使用方法

## Pre-Consensus

- 0-conf のトランザクションを安全にする
- 2 つのトランザクションの衝突を防ぎたい
- 直近 100 ブロックをマイニングしたノードが投票者となり、Avalanche プロトコルでトランザクションを確定する
- ネットワークに衝突するトランザクションがなければ今までと全て同じ
- 無効と判定されたトランザクションをマイニングするとブロックが orphan される
- 非マイニングノードには関係ない

---

## Post-Consensus

- reorg 攻撃耐性に Avalanche を使う
- 現状の Bitcoin ABC は、10 ブロック以上の reorg を拒否するようになっている
- その代わりに、Avalanche で reorg の拒否に関してノード間で合意を取る

---

# 参考資料など
- 論文
https://ipfs.io/ipfs/QmUy4jh5mGNZvLkjies1RWM4YuvJh5o2FYopNPVYwrRVGV
- Pre-Consensus
https://medium.com/@chrispacia/avalanche-pre-consensus-making-zeroconf-secure-ddedec254339
- Post-Consensus
https://medium.com/@Mengerian/avalanche-post-consensus-making-bitcoin-cash-indestructible-2464b1ae0382
https://www.avalabs.org/
