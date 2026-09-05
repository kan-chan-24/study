# prevの仕組み
## 1. スプレッド構文の上書きルール
- オブジェクトのスプレッド構文` { ...obj, key: value } `では、後に書いたプロパティが優先される
- 検証例:` { a: 1, b: 2 } `に a: 3 を後から書くと` { a: 3, b: 2 } `になる

```
const obj = { a: 1, b: 2 }
const newObj = { ...obj, a: 3 }
```

- 検証結果の理由: 同じキーが複数回出てきた場合、JavaScript は「最後に書かれた値」で上書きする
- この`...obj`が、`...prev`と同じ役割だと思っていい

## 2.`multiple_images: selectedFiles`で「選び直し」が実現できていた理由

``` 
  const handleFileChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    const files = e.target.files
    if(!files || files.length === 0) return

    const selectedFiles = Array.from(files)

    if (selectedFiles.length === 1) {
      setFormData((prev) => ({
        ...prev,
        image: selectedFiles[0],
      }))
    } else {
      setFormData((prev) => ({
        ...prev,
        multiple_images: selectedFiles,
        image: undefined,
      }))
    }
    console.log(selectedFiles)
    console.log(formData)
  }
```

- `{ ...prev, multiple_images: selectedFiles, image: undefined }`という記述の場合
- `...prev`の中にも古い`multiple_images`が入っているが、その直後に`multiple_images: selectedFiles`と明示的に書いているため上書きされる
- 具体的なトレース
  - 1回目：複数ファイル`[A,B,C]`を選択 
    - -> `multiple_images: [A, B, C]`
  - 2回目：複数ファイル`[D,E,F]`を選択
    - -> `...prev`の中の`[A, B, C]`より後に書かれた`[D, E, F]`が優先
    - -> `multiple_images: [D, E, F]`

## 3. `...prev`がないとどうなる？

```
// もしこう書いたら？
setFormData({
  multiple_images: selectedFiles,
  image: undefined,
})
```

- 考えるヒント：`...prev`が担っている役割は「前回の`formData`の中身を丸ごと引き継ぐこと」
- 質問：`title`や`description`など、フォームの他の項目は消えてしまわないか？
- 考察：...prev がないと、明記されている`multiple_images`と`image`の処理だけが行われてしまう。title や description は消えてしまう。
- 解説：`setFormData`に渡したオブジェクトが「新しい`formData`そのもの」になってしまうので、書いていないキーはすべて失われます。

## 4.【番外編】なぜ`image.undefined`も一緒に書いているのか

```
  setFormData((prev) => ({
    ...prev,
    image: selectedFiles[0],
  }))
} else {
  setFormData((prev) => ({
    ...prev,
    multiple_images: selectedFiles,
    image: undefined,
  }))
}
```

- 考えるヒント：「1枚だけ選んで`formData.image`に値が入っている状態」から「複数枚を選び直した」場合を想像する
- 質問：`image`の値をそのままにしておくと、送信時にどんな不整合が起きそうか？
- 考察：`image`に入っている画像を消さないと、複数選択したときに`image`と`multiple_images`の両方に画像が残ってしまう。
- 解説：「単数用の入れ物(`image`)」と「複数用の入れ物(`multiple_images`)」が両方とも値を持ってしまう

