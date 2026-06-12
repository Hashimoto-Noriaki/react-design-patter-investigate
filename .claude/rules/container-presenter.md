# Container/Presenter パターン

## 概要

コンポーネントを「見た目の表示を担当する Presenter」と「ロジックを担当する Container」に分離するパターン。
さらにカスタムフックを使うことで、ロジックを Container からも切り出し、独立した関数として再利用・テストできる。

## 責務の分離

| | Presenter | Container | カスタムフック |
|---|---|---|---|
| 役割 | UIの描画 | Presenterへの橋渡し | ロジック・状態管理 |
| state | 持たない | 持たない | 持つ |
| props | 受け取って表示するだけ | フックの結果をPresenterに渡す | なし（引数のみ） |
| 副作用 | なし | なし | あり（fetch、イベント等） |
| テスト | propsを渡すだけで完結 | ほぼテスト不要 | Reactコンポーネント不要でテスト可能 |

## ファイル構成

```
src/components/<ComponentName>/
  <ComponentName>Presenter.tsx
  <ComponentName>Container.tsx
  use<ComponentName>.ts       # カスタムフック（ロジック）
  index.ts
```

## コード規約

### Presenter
- `type Props` を同ファイルで定義する
- state・副作用（useEffect, fetch等）は持たない
- スタイルやマークアップのみを記述する

```tsx
type Props = {
  count: number
  onIncrement: () => void
}

export const CounterPresenter = ({ count, onIncrement }: Props) => {
  return (
    <div>
      <p>{count}</p>
      <button onClick={onIncrement}>+</button>
    </div>
  )
}
```

### カスタムフック（`use<ComponentName>.ts`）
- state・副作用・ビジネスロジックをここに集約する
- 戻り値は Presenter の Props に渡せる形にする
- React コンポーネントに依存しないため単体テストが容易

```ts
export const useCounter = () => {
  const [count, setCount] = useState(0)
  const onIncrement = () => setCount((c) => c + 1)
  return { count, onIncrement }
}
```

### Container
- カスタムフックを呼び出し、結果を Presenter に渡すだけ
- 自前でロジックや state を持たない
- JSX は Presenter の呼び出しのみ

```tsx
export const CounterContainer = () => {
  const props = useCounter()
  return <CounterPresenter {...props} />
}
```

### index.ts
- 外部に公開するのは Container のみ

```ts
export { CounterContainer as Counter } from './CounterContainer'
```

## このパターンを使う場面

- API fetch やビジネスロジックを含むコンポーネント
- Storybook でUIだけ確認したいコンポーネント
- ロジックとUIを独立してテストしたい場合
- 同じロジックを複数の Presenter で使い回したい場合
