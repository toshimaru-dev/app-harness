# harness

Claude Code のサブエージェント機能を使った自動開発パイプライン。短いプロダクトアイデアを入力するだけで、企画 → 実装 → 検証のサイクルが自動で回り続ける。

## 概要

```
あなた（1〜4行のアイデア）
    ↓
Planner  ── /docs/spec.md を生成
    ↓
Generator ── スプリント単位で実装 ── /docs/progress.md に記録
    ↓
Evaluator ── Playwright で実動作テスト ── /docs/feedback/sprint-N.md に結果出力
    ↓ 合格
次のスプリントへ（Generator に戻る）
    ↓ 不合格
Generator に差し戻し（修正後に再テスト）
```

## エージェント

| エージェント | 役割 | モデル |
|---|---|---|
| **Planner** | アイデアを製品仕様書（`/docs/spec.md`）に展開する | Opus |
| **Generator** | 仕様書を読み、1スプリントずつ実装する | Opus |
| **Evaluator** | Playwright MCP でアプリを実際に動かしてテストする | Opus |

## 使い方

### 1. 新しいプロジェクトを始める

Claude Code を起動し、Planner エージェントにアイデアを渡す：

```
planner エージェントを使って以下のアイデアを仕様書にしてください：
「シンプルなタスク管理アプリ。チームで共有できて、期限と優先度を設定できる」
```

Planner が `/docs/spec.md` を生成したら、Generator を呼び出す：

```
generator エージェントでスプリント 1 を実装してください
```

Generator の実装が終わったら、Evaluator でテストする：

```
evaluator エージェントでスプリント 1 を評価してください
```

合格後は Generator → Evaluator を繰り返す。

### 2. ファイル構成（自動生成）

```
docs/
  spec.md             # Planner が生成する製品仕様書
  progress.md         # Generator が記録する実装進捗
  feedback/
    sprint-1.md       # Evaluator が出力する評価結果
    sprint-2.md
    ...
```

## 評価基準

| 基準 | 合格閾値 |
|---|---|
| 機能完全性 | 4/5 以上 |
| 動作安定性 | 4/5 以上 |
| UI/UX品質 | 3/5 以上 |
| エラーハンドリング | 3/5 以上 |
| 回帰なし | 5/5（必須） |

1つでも閾値を下回ればスプリント不合格、Generator に自動差し戻し。

## 必要な MCP サーバ

- **Playwright MCP** — Evaluator がブラウザ操作テストに使用

`.claude/agents/evaluator.md` に設定済み。`npx @playwright/mcp@latest` で自動起動する。

## 絶対ルール

- **Planner は実装しない。Generator は仕様を変更しない。Evaluator はコードを修正しない。**
- スプリントは Sprint 1 → 2 → 3 の順に実装する（スキップ禁止）。
- 各スプリント完了時にアプリが正常に起動・動作していること。
- Generator は新スプリント着手前に、前スプリントの不合格フィードバックを先に修正する。
