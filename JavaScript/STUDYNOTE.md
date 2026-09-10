# JavaScript 学習ノート
---
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

# forEachの書き方

``` javascript
// 配列の各要素を表示
const numbers = [1, 2, 3, 4, 5];
numbers.forEach(num => {
    console.log(num);
});
//出力：
//1 2 3 4 5
```

``` javascript
// インデックスを含めて出力
const fruits = ["りんご", "バナナ", "ぶどう"];
fruits.forEach((fruit, index) => {
    console.log(`${index}: ${fruit}`);
});
//出力：
//0: りんご
//1: バナナ
//2: ぶどう
```

## `forEach`と`for`ループの違い

|比較項目|forEach|for ループ|
| ---- | ---- | ---- |
|記述の簡潔さ|◎ 短く書ける|△ やや長くなる|
|ループの途中終了|✖ break できない|◎ break 可能|
|return の扱い|✖ return できない|◎ return 可能|

``` javascript
const numbers = [1, 2, 3, 4, 5];

// forEach（ループを途中で抜けられない）
numbers.forEach(num => {
    if (num === 3) {
        return; // ループは止まらない（単なる関数のreturnで、ループには影響しない）
    }
    console.log(num);
});
// 出力：
// 1
// 2
// 4
// 5

// for ループ（break で終了可能）
for (let i = 0; i < numbers.length; i++) {
    if (numbers[i] === 3) {
        break; // ループ終了
    }
    console.log(numbers[i]);
}
// 出力：
// 1
// 2
```
## mapとの違い
- `forEach`と`map`は似ていますが、`map`は新しい配列を返すのに対して、`forEach`は配列を変更せずに処理を実行します。

``` javascript
const numbers = [1, 2, 3, 4, 5];

// forEach（結果を返さない）
numbers.forEach(num => num * 2);
console.log(numbers); // [1, 2, 3, 4, 5]

// map（新しい配列を返す）
const doubled = numbers.map(num => num * 2);
console.log(doubled); // [2, 4, 6, 8, 10]
```
- データを変換したいなら`map`、処理を実行するだけなら`forEach`

# `import`する時の{}の有無の違い

```
import { Memory } from '../../../types/memory'
```

{}の中身は「変数名」ではなく「取ってくる名前」
{}の中に書くのは、変数を保存するための"名前"ではなく、インポート元のファイルが外に公開している名前をそのまま指定する部分です。

memory.tsを見ると、こう書かれています。

```
export type Memory = { ... }
export type MemoryDetail = { ... }
```

`export`が付いているものは、他のファイルから使える状態になります。つまり`memory.ts`は「Memoryという名前」と「MemoryDetailという名前」の、2種類の型を公開しているわけです。

importする側は、その中から「どれが欲しいか」を選ぶだけ

```
import { Memory } from '...'       // Memoryという型が欲しい
import { MemoryDetail } from '...' // MemoryDetailという型が欲しい
```

これは「Memoryという名前の入れ物を作って、そこに何かを保存する」のではなく、「公開されている型のうちMemoryという名前のものを使わせてください」という指名に近いイメージです。

## {}がない場合の意味

```
import MemoryCard from './_components/memory-card'
```

{}なし = そのファイルの「代表選手」を受け取ること
デフォルトexport(default export) →importするとき{}は不要

memory-card.tsxの中はおそらくこうなっています。

```
const MemoryCard = () => { ... }
export default MemoryCard
```

`export default`は「このファイルが公開する物は、これが代表(メイン)です」という宣言です。1つのファイルにつき`export default`は1個だけしか書けません。

`import`する側は代表選手を受け取るだけなので、名前を指定して選ぶ必要がなく、{}も要りません。

```
import MemoryCard from './_components/memory-card'
```

## まとめると

|書き方|exportする側|importする側|名前は自由に変えられる?|
|{}あり|export const X = ...| import { X } from {...}|✕(元の名前と一致させる必要あり)|
|{}なし|export default X|import X from {...}|○(好きな名前を付けられる)|

memory.tsが{}必須だったのは、`Memory`と`MemoryDetail`という複数の型を名前付きで`export`していたからです。一方`memory-card.tsx`のようなReactコンポーネントのファイルは、1ファイル1コンポーネントが基本なので、デフォルト`export`がよく使われます。

# Props・親子関係のまとめ

## オブジェクトとは

- 変数:値を1つだけ入れる箱(例:const name = 'Taro')
- 関数:処理をまとめたもの。呼び出すと動く
- オブジェクト:複数の値を、名前(キー)付きでひとまとめにした箱(例:{ name: 'Taro', age: 20 })。中には値だけでなく関数も入れられる

## Propsの正体

- <MemoryForm formData={formData} onChange={handleChange} />という書き方は、裏側では「{ formData: formData, onChange: handleChange }という1つのオブジェクトを組み立てて、MemoryFormという関数に渡している」のと同じ
- つまりPropsは特別な仕組みではなく、ただのオブジェクト。複数の値・関数を1つにまとめて、コンポーネント(関数)に渡しているだけ

## 関数を値として渡すということ

- JavaScriptでは関数も数値や文字列と同じ「ただの値」として扱える
- handleChange(カッコなし)=「実行はまだしない、関数そのものを渡す」
- handleChange()(カッコあり)=「今すぐ実行する」
- onChange={handleChange}は前者。「必要なタイミングで、渡した先(MemoryForm)が代わりに実行してね」という意味

## export/importとの違い

||誰が使えるか|方向性|
|---|---|---|
|export/import|プロジェクト内の誰でも（importすれば）|使う側が能動的に取りに行く|
|props|親が名指しした、その子コンポーネントだけ|親から子への一方通行の受け渡し|

exportしていない関数(const handleChange = ...)は他ファイルから直接使えない。propsで渡すことで、その1回の描画に限り、名指しされた子コンポーネントだけがそれを使えるようになる。

## 親子関係を作っているコードはどこか

- 親たらしめるコード:親コンポーネントのreturn(描画結果)の中に、<MemoryForm ... />のように別コンポーネントを埋め込んで書くこと、それ自体
- 子たらしめるコード:memory-form.tsx側には存在しない。「誰かのreturnの中に埋め込まれる」という受動的な立場そのものが「子」の意味であり、能動的な宣言は不要
- 証拠:同じMemoryFormが、edit/index.tsxから呼ばれれば「editの子」、new/index.tsxから呼ばれれば「newの子」になる。ファイル自身は自分がどちらの子かを知らない、その場限りの関係
- formData={...}などのprops指定は、親子関係を作っているのではなく、すでにできている親子関係の上に荷物を追加で積んでいるだけ
