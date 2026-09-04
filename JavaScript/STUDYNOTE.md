# 📝prevの仕組み
1. スプレッド構文の上書きルール
- オブジェクトのスプレッド構文` { ...obj, key: value } `では、後に書いたプロパティが優先される
- 検証例:` { a: 1, b: 2 } `に a: 3 を後から書くと` { a: 3, b: 2 } `になる
```
const obj = { a: 1, b: 2 }
const newObj = { ...obj, a: 3 }
```
- 理由: 同じキーが複数回出てきた場合、JavaScript は「最後に書かれた値」で上書きする
2.`multiple_images: selectedFiles`で「選び直し」が実現できていた理由
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

