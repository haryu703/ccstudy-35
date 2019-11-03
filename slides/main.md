---
marp: true
---

# Avalanche

### haryu703

---
# 自己紹介
[haryu703](https://twitter.com/haryu703)

最近の興味
- Bitcoin Cash
- Rust
- クラウド開発環境

---
# Avalanche の概要
- すごい多数決
- 多数決への参加資格は決めていない
  - PoS や PoW で決めることができる

---
# Abstract
- 新しい BFT プロトコル作った
- このペーパーでは Avalanche について
  - 3つのプロトコルに分けて説明する
  - プロトコルの保証を分析する
  - 使い方を説明する
- 1300tps, 4sec/confirmation 出せるらしい

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
- Avalance について
  - [ゴシップアルゴリズム](https://ja.wikipedia.org/wiki/%E3%82%B4%E3%82%B7%E3%83%83%E3%83%97%E3%83%97%E3%83%AD%E3%83%88%E3%82%B3%E3%83%AB)に影響を受けている
    - 参加者間で繰り返し情報を交換する手法
  - 参加者をネットワークからランダムに繰り返しサンプリングする
  - PoW しない
  - $O(kn \text{log} n)$ から $O(kn)$ のコミュニケーションで済む
    - $k$ はセキュリティパラメータ
  - ただし、トランザクションの「正しさ」は保証される必要がある
      - 暗号資産の場合、発行者は署名で保証されている
      - Double Spend はありうるが、大きな影響はない（？）

---
# 2. Approach
- Slush(雪泥) → Snowflake(雪片) → Snowball(雪玉) → Avalanche(雪崩)
- プロトコルを定義する
- 証明と分析はこの次

---
- 3つの前提がある
  - Bitcoinのような確率的な安全性
    - 従来と同じで、パラメータ次第
  - 全てのトランザクションの順番は決めない
    - トランザクションに関連する部分だけ決める
  - 不正なノードの動作は保証しない
    - 最終的には正常なノードが動作する

---
## 2.1 Model, Goals, Threat Model
- 定義
  - $\mathcal N$ ... ノード
  - $\mathcal C$ ... 正直ノード
  - $\mathcal B$ ... 不正ノード
- Bitcoin の UTXO モデルで考える
  - 不正なトランザクションは署名によって防がれる
  - 競合するトランザクションは発行できる（二重支払い）
    - $\mathcal C$はしないけど$\mathcal B$はする
- Avalanche は以下の保証を高確率で保証する
  - Safety ... 競合するトランザクションを受け入れる2つの$\mathcal C$がない
  - Liveness ... $\mathcal C$が発行したトランザクションは全ての$\mathcal C$が受け入れる
- $1-\epsilon$の確率で安全性が維持される
  - $\epsilon$はシステム上のセキュリティパラメータ

---
# 3. Analysis

---
# 4. Implementation

---
# 5. Evaluation

---
# 6. Related Work

---
# 7. Conclusions

---
# BCH で提案されている使用方法

---
# まとめ

---
# 参考資料など
https://ipfs.io/ipfs/QmUy4jh5mGNZvLkjies1RWM4YuvJh5o2FYopNPVYwrRVGV
https://medium.com/@chrispacia/avalanche-pre-consensus-making-zeroconf-secure-ddedec254339
https://medium.com/@Mengerian/avalanche-post-consensus-making-bitcoin-cash-indestructible-2464b1ae0382
https://www.avalabs.org/
