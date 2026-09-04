# `<input>`で複数ファイルを選べるようにする
`<input>`要素の type="file" 型は、ユーザーが一つまたは複数のファイルを端末のストレージから選択することができるようにします。

ここにmultiple属性を追加すると、複数ファイル選択ができます。
```
<input
  id="image"
  type="file"
  name="image"
  multiple
  accept="image/*"
  onChange={onFileChange}
  className="w-full rounded-md border border-gray-300 p-3 focus:ring-2 focus:ring-blue-500 focus:outline-none"
/>
```
