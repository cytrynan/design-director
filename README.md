# design-director

「Claudeが作った」と見抜かれないUIを作るための Claude Code プラグイン。
エージェント `design-director` とスキル `design-tokens` の2つが入っている。

## できること

- UIの方向性を6種のメニューから1つだけ強制選択させる
- デザイントークンを `DESIGN.md` に具体値で固定し、全画面の単一の正にする
- AI製の「指紋」ブラックリストを出荷前QAで照合し、1つでも該当したら不合格にする
- 可読性ガードレール（本文16px以上・行長60〜80字・コントラスト・タップ領域44px以上）を方向性に関わらず常時かける。「個性的だが読みにくい」は不合格

## インストール

### A. プラグインとして入れる（推奨）

Claude Code の対話セッションで:

```
/plugin marketplace add cytrynan/design-director
```

```
/plugin install design-director@cytrynan-design
```

### B. ファイルを直接置く

プラグイン機構を使わない場合はコピーするだけでも動く。

```bash
git clone https://github.com/cytrynan/design-director.git
cp design-director/agents/design-director.md ~/.claude/agents/
cp -r design-director/skills/design-tokens ~/.claude/skills/
```

プロジェクト単位で使いたいなら `~/.claude/` の代わりに
リポジトリ内の `.claude/` に置く。

## 使い方

### 1. UIを作る/作り直すとき

エージェントは説明文のトリガー語（`UI設計` / `デザイン` / `見た目` / `restyle` /
`landing page` / `画面デザイン` など）で自動的に呼ばれる。明示的に呼ぶなら:

```
design-director エージェントで、このアプリのトップ画面をデザインし直して
```

エージェントは次の順で動く。

```
Step 0  DESIGN.md があるか確認（あればそれが正・実装へ直行）
Step 1  対象ユーザー・用途・トーンを把握
Step 2  方向性を1つ選ぶ（収束禁止ルールあり）
Step 3  トークンを具体値で固定し DESIGN.md に記録
Step 4  レイアウト構造から方向性を反映して実装
Step 5  ブラックリスト照合＋スクイントテスト＋コントラスト＋全状態チェック
```

### 2. 方向性メニュー（Step 2）

| 方向性 | 定義的特徴 | 向くプロダクト |
|---|---|---|
| エディトリアル/スイス | 文字が主役。大胆な見出し、厳格グリッド、余白で分節 | 情報系・メディア・ポートフォリオ |
| ソフト・ネオブルータリズム | 太い境界線、ベタ塗り面、硬い影、角丸ほぼ無し | ツール・学習系・若年向け |
| インダストリアル/モノ | 等幅or幾何学サンセリフ、罫線区切り、高データ密度 | ダッシュボード・開発者向け・金融 |
| オーガニック/ナチュラル | アースカラー、紙/布テクスチャ、丸みと手仕事感 | 健康・食・ライフスタイル |
| ラグジュアリー/端正 | 細身セリフ+広letter-spacing、2色、余白過剰 | 高級感・ブランドサイト |
| レトロ/クラフト | 特定年代モチーフを1つ決めて徹底 | 個性優先の企画・キャンペーン |

収束禁止ルール: 迷ったときに毎回ミニマルSaaS風へ逃げない。
同じ作者の別プロジェクトでは別の方向を選ぶ。

### 3. `DESIGN.md` を先に作りたいとき

実装より先に方向性だけ決めておきたい場合はスキルを直接呼ぶ。

```
design-tokens スキルで DESIGN.md を作って
```

雛形は [`skills/design-tokens/template.md`](skills/design-tokens/template.md)。

### 使わなくていい場面

- 自分専用の使い捨てツール・CLIスクリプト（見た目が問題にならない）
- 既に確立したデザインシステムがあるプロダクト（`DESIGN.md` にそれを書けば従う）

## 技術の詳細

### 中身

| 種別 | 名前 | 役割 |
|---|---|---|
| エージェント | `design-director` (model: opus) | UI設計・実装の実行役。方向性選択 → トークン固定 → 実装 → 自己QA まで一気に担当 |
| スキル | `design-tokens` | プロジェクトルートに `DESIGN.md` を作る/更新するガイド＋テンプレート |
| ドキュメント | `docs/why-ai-slop.md` | なぜこの設計なのかの根拠と参照元 |

エージェントとスキルは併用前提。エージェントが方向性を決め、その結果を
`DESIGN.md`（スキルのテンプレート形式）に書き出し、以降の全画面がそれに従う。

### 設計の考え方

AIが生成するUIが毎回同じ顔になるのは、指示が無いとモデルが学習データの統計的平均
（Inter・紫グラデ・中央ヒーロー・3枚カード…）に落ちるから（分布収束）。
だから「もっと良くして」では直らない。方向性の強制選択・トークン固定・ブラックリストの3つを機械的に強制するのはこのため。
根拠と参照元は [`docs/why-ai-slop.md`](docs/why-ai-slop.md)。

### AI製の「指紋」ブラックリスト（抜粋）

出荷前QAで機械的に照合する。1つでも該当したら不合格。
全リストは [`agents/design-director.md`](agents/design-director.md) を参照。

- 紫→青グラデーション、白背景+紫アクセント、ラベンダー
- Inter / Roboto / Poppins / Space Grotesk / Geist の見出し使い
- 中央寄せヒーロー+H1直上badge、「1·2·3」ステップ行、横並び3枚featureカード
- hero→3カード→ロゴ帯→pricing→FAQ→footer の雛形直送
- 全カードに1px灰色枠、色付き左ボーダー、無加工の `rounded-2xl shadow-lg p-6`
- 理由なきダークモード既定、ネオングロー枠、意味なきglassmorphism、3Dブロブ
- 全要素一律 fade-in、巨大丸角アイコンを見出し上に置く定型

### 補足

- エージェントの指示は日本語。フォント推奨に和文フォント（Zen Kaku Gothic New /
  BIZ UDPGothic / Shippori Mincho 等）を含むため、日本語UIで特に効く。
  英語UIでも方向性メニューとブラックリストはそのまま使える。
- Anthropic公式の `frontend-design` スキルと併用可能（本プラグインはその具体化版）。

## ライセンス

MIT
