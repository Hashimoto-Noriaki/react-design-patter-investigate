# Compound Component パターン

## 概要

複数の子コンポーネントが協調して動作する、柔軟で再利用可能なコンポーネントを作る設計パターン。
親コンポーネントが内部状態を管理し、子コンポーネントは Context 経由でその状態にアクセスする。
`<select>` と `<option>` の関係に似ており、個々は独立したコンポーネントだが、組み合わせて初めて意味を持つ。

## 通常の props 渡しとの比較

```tsx
【通常：props で全てを制御（硬直的）】
<TaskCard
  title="タスク名"
  description="説明"
  badge="urgent"
  footer={<button>完了</button>}
/>

【Compound Component：使う側がレイアウトを自由に組み立て】
<TaskCard>
  <TaskCard.Badge>urgent</TaskCard.Badge>
  <TaskCard.Title>タスク名</TaskCard.Title>
  <TaskCard.Description>説明</TaskCard.Description>
  <TaskCard.Footer>
    <button>完了</button>
  </TaskCard.Footer>
</TaskCard>
```

## ファイル構成

```
src/components/<ComponentName>/
  <ComponentName>.tsx          # 親コンポーネント（Context 定義・状態管理）
  <ComponentName>.Title.tsx    # 子コンポーネント
  <ComponentName>.Description.tsx
  <ComponentName>.Badge.tsx
  <ComponentName>.Footer.tsx
  index.ts
```

## コード規約

### 親コンポーネント（`<ComponentName>.tsx`）
- Context を定義し、Provider で子を囲む
- 内部状態はここで管理する
- 子コンポーネントを static プロパティとして付与しない（index.ts で合成する）

```tsx
import { createContext, useContext } from 'react'

type TaskCardContextType = {
  isCompleted: boolean
  onToggle: () => void
}

const TaskCardContext = createContext<TaskCardContextType | null>(null)

export const useTaskCardContext = () => {
  const context = useContext(TaskCardContext)
  if (!context) throw new Error('useTaskCardContext must be used within TaskCard')
  return context
}

export const TaskCard = ({ children }: { children: React.ReactNode }) => {
  const [isCompleted, setIsCompleted] = useState(false)
  const onToggle = () => setIsCompleted((v) => !v)

  return (
    <TaskCardContext.Provider value={{ isCompleted, onToggle }}>
      <div className="task-card">{children}</div>
    </TaskCardContext.Provider>
  )
}
```

### 子コンポーネント（`<ComponentName>.<Part>.tsx`）
- `useXxxContext()` で親の状態にアクセスする
- 単独では使えないことをコメントで明示しなくてよい（Context のエラーで自明）

```tsx
// TaskCard.Title.tsx
import { useTaskCardContext } from './TaskCard'

export const TaskCardTitle = ({ children }: { children: React.ReactNode }) => {
  const { isCompleted } = useTaskCardContext()
  return (
    <h3 style={{ textDecoration: isCompleted ? 'line-through' : 'none' }}>
      {children}
    </h3>
  )
}
```

### index.ts
- 親コンポーネントに子コンポーネントをプロパティとして付与して export する

```ts
import { TaskCard as _TaskCard } from './TaskCard'
import { TaskCardTitle } from './TaskCard.Title'
import { TaskCardDescription } from './TaskCard.Description'
import { TaskCardBadge } from './TaskCard.Badge'
import { TaskCardFooter } from './TaskCard.Footer'

export const TaskCard = Object.assign(_TaskCard, {
  Title: TaskCardTitle,
  Description: TaskCardDescription,
  Badge: TaskCardBadge,
  Footer: TaskCardFooter,
})
```

## このパターンを使う場面

- 使う側がレイアウトや子の順序を自由に変えたいコンポーネント（カード、モーダル、タブ等）
- 一部の子だけ表示したい・追加したいケースが複数ある場合
- props が増えすぎて柔軟性が失われてきた既存コンポーネントの改善

## 注意点

- 子コンポーネントは必ず対応する親の Provider 内で使う必要がある
- 親の状態を持ちすぎると責務が大きくなるため、複雑なロジックはカスタムフックに切り出す
- Provider パターンと組み合わせる場合、スコープが異なることに注意（Compound は局所的、Provider はアプリ全体〜ページ単位）
