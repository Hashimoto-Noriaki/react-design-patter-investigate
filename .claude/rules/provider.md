# Provider パターン

## 概要

React の Context API を使い、深くネストされたコンポーネントに対して props のバケツリレーをせずにデータを渡すパターン。
カスタムフックで切り出したロジックをグローバルに共有したい場合に使う。

## props バケツリレーとの比較

```
【バケツリレー】                     【Provider パターン】
App                                  App
└─ PageA                             └─ CounterProvider
   └─ SectionB (count渡す)               └─ PageA
      └─ WidgetC (count渡す)                └─ SectionB
         └─ Counter ← やっと使える              └─ WidgetC
                                                  └─ Counter ← useCounterContext() で直接取得
```

## ファイル構成

```
src/providers/
  <ContextName>Provider.tsx   # Context の定義・Provider コンポーネント・カスタムフック
```

## コード規約

### Provider ファイル（`<ContextName>Provider.tsx`）

1つのファイルに Context・Provider・カスタムフックをまとめて定義する。

```tsx
import { createContext, useContext, useState } from 'react'

type CounterContextType = {
  count: number
  onIncrement: () => void
  onDecrement: () => void
}

const CounterContext = createContext<CounterContextType | null>(null)

// Provider コンポーネント
export const CounterProvider = ({ children }: { children: React.ReactNode }) => {
  const [count, setCount] = useState(0)
  const onIncrement = () => setCount((c) => c + 1)
  const onDecrement = () => setCount((c) => c - 1)

  return (
    <CounterContext.Provider value={{ count, onIncrement, onDecrement }}>
      {children}
    </CounterContext.Provider>
  )
}

// Context を使うカスタムフック（Provider 外からの利用を防ぐ）
export const useCounterContext = () => {
  const context = useContext(CounterContext)
  if (!context) throw new Error('useCounterContext must be used within CounterProvider')
  return context
}
```

### Provider の配置
- アプリ全体で使う場合は `main.tsx` またはルートコンポーネントで囲む
- 特定のページ・機能スコープで使う場合はその親コンポーネントで囲む

```tsx
// main.tsx
<CounterProvider>
  <App />
</CounterProvider>
```

### Context を使う側（任意のネスト深さのコンポーネント）

```tsx
const Counter = () => {
  const { count, onIncrement } = useCounterContext()
  return <button onClick={onIncrement}>{count}</button>
}
```

## Container/Presenter パターンとの組み合わせ

Provider パターンは Container/Presenter パターンと併用できる。
Container が `useXxxContext()` を呼び、Presenter に props として渡す。

```tsx
// CounterContainer.tsx
export const CounterContainer = () => {
  const props = useCounterContext()   // Context から取得
  return <CounterPresenter {...props} />
}
```

## このパターンを使う場面

- 複数の離れたコンポーネントで同じ状態・ロジックを共有したい場合
- props のバケツリレーが3階層以上になりそうな場合
- 認証情報・テーマ・言語設定など、アプリ全体に影響するグローバル状態

## 注意点

- Context の値が変わると Provider 配下の全コンポーネントが再レンダリングされる
- 頻繁に変わる値と静的な値は Context を分けることを検討する
