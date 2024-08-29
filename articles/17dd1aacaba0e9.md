---
title: "react-hook-form + zod でエラーが発火しないケースの覚書"
emoji: "😺"
type: "tech"
topics:
  - "reacthookform"
  - "zod"
published: true
published_at: "2024-03-15 10:09"
---

# 開発環境
```json
"react-hook-form": "^7.45.4"
"zod": "^3.22.2"
```
# 実現したいこと
hoge と fuga 2つのテキストフィールドがあり、両方入力されている、もしくは両方からの時は true、片方のみ入力されており、もう片方がからの時は false とする。
# 発生ケース
片方（hoge の方と仮定）のみ変更を行った際に、もう片方にエラーが表示されない。
fuga のテキストフィールドを変更するとエラーメッセージは無事に表示される。
# 原因
react-hook-form はデフォルトがフィールドの変更をバリデーションのトリガーにしているため、発生していない風に見えている。（fuga 用のエラーメッセージが生成されていない。）
# 解決方法
テキストフィールドのフォーカスが外れた場合などに、両方のバリデーションが発火するための処理を追加する。
具体的なコード↓
```tsx
export const schema = z.object({
  hoge: z.string().nullable(),
  fuga: z.string().nullable(),
})
.superRefine((val, ctx) => {
  if(...) {
    ctx.addIssue({
      code: 'custom',
      path: ['hoge'],
      message: '🐖🐖🐖🐖🐖',
    });

    ctx.addIssue({
      code: 'custom',
      path: ['fuga'],
      message: '🐎🐎🐎🐎🐎',
    });
  }
})

const component = () => {
  const {
    …,
    trigger,
  } = useForm(resolver: zodResolver(schema),)

  const handler = () => {
    if (…) {
      trigger('hoge')
      trigger('fuga')
    }
  }

  return (
    <>
      <TextField {…register('hoge')} onBlur={handler} />
      <TextField {…register('fuga')} onBlur={handler} />
    </>
  )
}
```

以上！