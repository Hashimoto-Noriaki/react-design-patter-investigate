# React Design Pattern Project

このプロジェクトは React のデザインパターンを学習・実装するためのリポジトリです。

## 採用するデザインパターン

| パターン | ルールファイル | 概要 |
|---------
| Container/Presenter | [.claude/rules/container-presenter.md](.claude/rules/container-presenter.md) | ロジックとUIの分離 |
| Provider | [.claude/rules/provider.md](.claude/rules/provider.md) | Context APIによるグローバル状態管理・props バケツリレーの解消 |
| Compound Component | [.claude/rules/compound-component.md](.claude/rules/compound-component.md) | 複数の子コンポーネントが協調する柔軟なレイアウト構造 |

## ディレクトリ構成

```bash
src/
  components/
    <ComponentName>/
      <ComponentName>Presenter.tsx   # UI担当
      <ComponentName>Container.tsx   # ロジック担当
      index.ts                       # 再エクスポート
  providers/
    <ContextName>Provider.tsx        # Context + Provider の定義
    use<ContextName>.ts              # Context を使うカスタムフック
  components/
    <ComponentName>/
      <ComponentName>.tsx            # 親コンポーネント（内部状態を管理）
      <ComponentName>.<Part>.tsx     # 子コンポーネント群
      index.ts                       # 再エクスポート（親に子をプロパティとして付与）
```
