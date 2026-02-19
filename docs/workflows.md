# GitHub Actions ワークフロー設定ドキュメント

このドキュメントでは、`.github/workflows` ディレクトリに含まれる GitHub Actions ワークフローの設定について説明します。

## ワークフロー一覧

| ファイル名 | 説明 |
|---|---|
| `claude.yml` | Claude Code による Issue・PR への自動応答 |
| `claude-code-review.yml` | Claude Code による PR 自動コードレビュー |

---

## `claude.yml` — Claude Code 自動応答ワークフロー

### 概要

Issue や Pull Request のコメントで `@claude` とメンションすることで、Claude が自動的に応答・実装・コードレビューを行うワークフローです。

### トリガー条件

以下のイベントで起動します。

| イベント | 条件 |
|---|---|
| `issue_comment` | コメントが作成されたとき |
| `pull_request_review_comment` | PR レビューコメントが作成されたとき |
| `issues` | Issue が `opened` または `assigned` されたとき |
| `pull_request_review` | PR レビューが `submitted` されたとき |

ただし、コメント本文またはタイトルに `@claude` が含まれる場合のみ実行されます。

### 必要な権限

```yaml
permissions:
  contents: read
  pull-requests: read
  issues: read
  id-token: write
  actions: read
```

### 使用する Action

```yaml
uses: anthropics/claude-code-action@v1
```

### 必要なシークレット

| シークレット名 | 説明 |
|---|---|
| `CLAUDE_CODE_OAUTH_TOKEN` | Claude Code の認証トークン |

### 使い方

Issue または PR のコメントに `@claude` を含めてメッセージを書くと、Claude が自動的に応答します。

**例:**
```
@claude このバグを修正してください
@claude このコードをレビューしてください
@claude テストを追加してください
```

### カスタマイズ

`claude.yml` では以下の設定が可能です（コメントアウトされた設定を参照）。

- **`prompt`**: カスタムプロンプトを設定し、Claude に特定の動作をさせる
- **`claude_args`**: 許可するツールの制限など、Claude Code の詳細設定
- **`additional_permissions`**: CI 結果の読み取りなど追加の権限付与

---

## `claude-code-review.yml` — Claude Code 自動コードレビューワークフロー

### 概要

Pull Request が作成・更新されたときに、Claude が自動的にコードレビューを実施するワークフローです。

### トリガー条件

以下のプルリクエストイベントで起動します。

| イベントタイプ | 説明 |
|---|---|
| `opened` | PR が新規作成されたとき |
| `synchronize` | PR に新しいコミットがプッシュされたとき |
| `ready_for_review` | ドラフト PR がレビュー可能状態になったとき |
| `reopened` | クローズされた PR が再オープンされたとき |

### 必要な権限

```yaml
permissions:
  contents: read
  pull-requests: read
  issues: read
  id-token: write
```

### 使用する Action・プラグイン

```yaml
uses: anthropics/claude-code-action@v1
```

コードレビュー機能はプラグイン経由で提供されます。

```yaml
plugin_marketplaces: 'https://github.com/anthropics/claude-code.git'
plugins: 'code-review@claude-code-plugins'
prompt: '/code-review:code-review ${{ github.repository }}/pull/${{ github.event.pull_request.number }}'
```

### 必要なシークレット

| シークレット名 | 説明 |
|---|---|
| `CLAUDE_CODE_OAUTH_TOKEN` | Claude Code の認証トークン |

### カスタマイズ

`claude-code-review.yml` では以下の設定が可能です（コメントアウトされた設定を参照）。

- **対象ファイルパスの絞り込み**: `paths` を指定して、特定のファイルが変更された場合のみレビューを実行
  ```yaml
  paths:
    - "src/**/*.ts"
    - "src/**/*.tsx"
  ```

- **PR 作成者によるフィルタリング**: `if` 条件を使って特定のユーザーや contributor タイプの PR のみを対象にする
  ```yaml
  if: |
    github.event.pull_request.user.login == 'external-contributor' ||
    github.event.pull_request.author_association == 'FIRST_TIME_CONTRIBUTOR'
  ```

---

## セットアップ手順

1. **シークレットの設定**
   - GitHub リポジトリの Settings > Secrets and variables > Actions に移動
   - `CLAUDE_CODE_OAUTH_TOKEN` を追加（Claude Code の OAuth トークンを設定）

2. **ワークフローファイルの配置**
   - `.github/workflows/claude.yml` — Claude Code 自動応答
   - `.github/workflows/claude-code-review.yml` — 自動コードレビュー

3. **動作確認**
   - Issue に `@claude テスト` とコメントして、Claude が応答することを確認

## 参考リンク

- [claude-code-action ドキュメント](https://github.com/anthropics/claude-code-action)
- [Claude Code CLI リファレンス](https://code.claude.com/docs/en/cli-reference)
- [使用方法ガイド](https://github.com/anthropics/claude-code-action/blob/main/docs/usage.md)
