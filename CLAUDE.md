# React Design Pattern Project

このプロジェクトは **Qiita クローン**を題材に、React のデザインパターンを学習・実装するためのリポジトリです。
機能は1つずつ段階的に追加していく。

## 採用するデザインパターン

| パターン | ルールファイル | 概要 |
| --- | --- | --- |
| Container/Presenter | [.claude/rules/container-presenter.md](.claude/rules/container-presenter.md) | ロジックとUIの分離 |
| Provider | [.claude/rules/provider.md](.claude/rules/provider.md) | Context APIによるグローバル状態管理・props バケツリレーの解消 |
| Compound Component | [.claude/rules/compound-component.md](.claude/rules/compound-component.md) | 複数の子コンポーネントが協調する柔軟なレイアウト構造 |

## 実装する機能（実装順）

| # | 機能 | 説明 |
| --- | --- | --- |
| 1 | 記事一覧 | トップページに記事カードを並べる |
| 2 | 記事詳細 | Markdown レンダリング・タグ表示 |
| 3 | 認証 | ログイン・ログアウト・ユーザー登録 |
| 4 | 記事投稿・編集 | Markdown エディタで記事を作成・編集 |
| 5 | いいね | 記事へのいいね（カウント表示） |
| 6 | タグ | タグ一覧・タグ絞り込み |
| 7 | フォロー | ユーザーフォロー・フォロワー |

## ページ構成

```bash
/                    # 記事一覧（トップ）
/articles/:id        # 記事詳細
/articles/new        # 記事投稿
/articles/:id/edit   # 記事編集
/login               # ログイン
/signup              # ユーザー登録
/users/:id           # ユーザープロフィール
/tags/:tag           # タグ別記事一覧
```

## ディレクトリ構成

```bash
src/
  pages/
    TopPage.tsx                      # ページコンポーネント（ルーティングの単位）
  components/
    <ComponentName>/
      <ComponentName>Presenter.tsx   # UI担当
      <ComponentName>Container.tsx   # ロジック担当
      use<ComponentName>.ts          # カスタムフック（ロジック）
      index.ts                       # 再エクスポート
  providers/
    <ContextName>Provider.tsx        # Context + Provider の定義
    use<ContextName>.ts              # Context を使うカスタムフック
  types/
    index.ts                         # 共通の型定義（Article, User, Tag など）
  mocks/
    articles.ts                      # モックデータ
```
