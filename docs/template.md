# マークダウンの拡張記述について

## ローカルでのテストについて

このドキュメントの一番上のディレクトリ(/lambda)をターミナルで開いて、

```sh
npm run docs:dev
```

と入力するとローカルでテストができます。

テストページは編集して保存するたびに自動で更新されます。

## 絵文字

```
:tada: :v: :o: :x: :100: :sob: :nauseated_face: :rofl: :exploding_head: :thinking: :thumbsup: :white_check_mark: :shrug: :cry: :ok: :ng: :ok_hand:
```

:tada: :v: :o: :x: :100: :sob: :nauseated_face: :rofl: :exploding_head: :thinking: :thumbsup: :white_check_mark: :shrug: :cry: :ok: :ng: :ok_hand:

## 目次

```
[[toc]]
```

[[toc]]

## カスタムメッセージ

```
::: info
これは情報ボックスです
:::

::: tip
これはTipボックスです
:::

::: warning
これは警告ボックスです
:::

::: danger
これは危険ボックスです
:::

::: details クリックして展開
これは詳細ボックスです
:::
```

::: info
これは情報ボックスです
:::

::: tip
これは Tip ボックスです
:::

::: warning
これは警告ボックスです
:::

::: danger
これは危険ボックスです
:::

::: details クリックして展開

```js
console.log("Hello, VitePress!");
```

:::

## テキストのハイライト、注釈

`このテキストはハイライトされます`

```
`このテキストはハイライトされます`
```

---

注釈表示です

---

```
---

注釈表示です

---
```

## コードのハイライト、追加、削除

この行は普通の行です。<br>
この行をハイライトします // [!code hl]<br>
この行を追加します // [!code ++]<br>
この行を削除します // [!code --]

```diff
この行は普通の行です。
この行をハイライトします // [!code hl]
この行を追加します // [!code ++]
この行を削除します // [!code --]
```

## html タグ

<strong>強調表示</strong>

```html
<strong>強調表示</strong>
```

<div style="text-align:center;">このテキストは中央寄せです</div>

```
<div style="text-align:center;">このテキストは中央寄せです</div>
```

## リンク、画像の埋め込み

### 画像の埋め込み

![ランダムな画像](https://picsum.photos/300/200 "ランダムな画像")

```
![ランダムな画像](https://picsum.photos/300/200 "ランダムな画像")
```

※リンクの後の"ランダムな画像"の部分は任意ですが、アクセシビリティーのためにあったほうが良いとされます。

### リンクの読み込み

[amazon](https://www.amazon.co.jp/)

```
[amazon](https://www.amazon.co.jp/)
```
