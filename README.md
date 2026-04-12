# tf-workflows

トランスフィード Firebase アプリ群の AI レビューシステム（共通 Reusable Workflows）

## 概要

非エンジニア主体の開発体制でも品質を担保するため、Claude + Gemini による AI クロスレビューを GitHub Actions で自動実行する。

**基本方針**: 速度をあまり落とさず、致命傷は必ず 2 重で止める

## リポジトリ構成

```
.github/workflows/
├── ai-review-claude.yml    # Claude レビュー（ロジック・ドメイン知識）
├── ai-review-gemini.yml    # Gemini レビュー（セキュリティ・構造）
├── weekly-scan.yml          # 週次全量スキャン
└── security-check.yml       # firestore.rules 変更時の追加チェック

prompts/
├── claude-review-prompt.md  # Claude 用システムプロンプト
├── gemini-review-prompt.md  # Gemini 用システムプロンプト
└── checklist.md             # 共通チェックリスト（Phase 1-3）
```

## 対象リポジトリ

| リポ | レビュー強度 |
|------|-------------|
| tf-nagai/susshiro | 最強（Claude + Gemini + セキュリティチェック + 週次スキャン） |
| tf-nagai/quickloda-lp | 強（Claude + 週次スキャン） |
| tf-nagai/quickpass | 中（Claude 軽量） |
| tf-nagai/BizFlowBuilder | 中（Claude 軽量） |
| tf-nagai/waresight | 中（Claude 軽量） |

## 各リポでの使い方

各アプリリポの `.github/workflows/review.yml` に以下を配置：

```yaml
name: AI Review
on:
  pull_request:
    branches: [main]

jobs:
  claude-review:
    uses: tf-nagai/tf-workflows/.github/workflows/ai-review-claude.yml@main
    with:
      review_level: "full"  # または "light"
      phase: "1"            # チェックリスト Phase
    secrets:
      ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}

  # 「最強」レベルのリポのみ追加
  gemini-review:
    uses: tf-nagai/tf-workflows/.github/workflows/ai-review-gemini.yml@main
    with:
      phase: "1"
    secrets:
      GEMINI_API_KEY: ${{ secrets.GEMINI_API_KEY }}

  security-check:
    uses: tf-nagai/tf-workflows/.github/workflows/security-check.yml@main
    secrets:
      ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
```

## 必要な Secrets

各アプリリポの Settings → Secrets and variables → Actions に設定：

| Secret 名 | 説明 |
|-----------|------|
| `ANTHROPIC_API_KEY` | Claude API キー |
| `GEMINI_API_KEY` | Gemini API キー（クロスレビュー対象リポのみ） |

## 誤検知バイパス

1. **PR ラベル**: `override-ai-review` を付与 → マージブロック解除
2. **コード内コメント**: `// ai-review-ignore: [理由]` → 該当行の指摘を除外

## チェックリスト Phase

- **Phase 1**（導入直後〜2週間）: 致命 8 項目のみ
- **Phase 2**（3〜4週目）: + 警告 11 項目
- **Phase 3**（2ヶ月目〜）: + 情報 12 項目

Phase は各リポの workflow 呼び出しで指定。段階的に上げていく。
