# Railsのメモ

---

# ImageUploader の役割について
- CarrierWave（画像アップロードを扱うgem）の`Uploader`クラスは、簡単に言うと「画像をどう扱うかのルールブック」
- ImageUploaderの役割は以下
 - 画像をどこ（どのディレクトリ）に保存するか
 - 画像のサイズをリサイズするか、するならどんなサイズに変換するか
 - どんな拡張子（jpg, png など）のファイルだけ許可するか

実際に`ImageUploader`に記述されているコードの意味 
``` Ruby
# 「アップロードしたファイルを、サーバーの中のファイルとして保存する」という設定
storage :file

# 画像をどこに保存するか
def store_dir
"uploads/#{model.class.to_s.underscore}/#{mounted_as}/#{model.id}"
end

# 「まだ画像が投稿されていないときに表示する、デフォルトの画像」の設定
def default_url(*args)
  ActionController::Base.helpers.asset_path("default.png")
end

# 「アップロードを許可する拡張子」を配列で指定
def extension_allowlist
  %w[jpg jpeg gif png]
end
```

---

# rails g model / rails g migration / rails db:migrate の違いについて
| コマンド | 役割 |
|---|---|
| `rails g model` | モデルファイル（`app/models/○○.rb`）と、それに対応する マイグレーションファイル（設計図） を生成する |
| `rails g migration` | マイグレーションファイル（設計図）だけを生成する（モデルファイルは作らない） |
| `rails db:migrate` | マイグレーションファイル（設計図）の内容を実際にデータベースに反映させる |

## イメージで考えると
- `rails g model` や `rails g migration` は「**こういうテーブルを作りたい/変更したい** という設計図を書く」作業
- `rails db:migrate` は「その設計図をもとに、実際に工事（データベースへの反映）を行う」作業
つまり、設計図を書いただけでは、まだ実際のデータベースには何も変化が起きていない状態

## 実際にマイグレート状態を確かめてみる
- `rails db;migrate:status`を実行することで、生成されたマイグレーションファイルがマイグレートされているか（up/down）が確認できる

```
 Status   Migration ID    Migration Name
--------------------------------------------------
   up     20250421055009  Create users
   up     20250422050023  Create memories
  down    20260908022548  Create memory images
```

- 確認してみると`rails g model`を行った際に生成されたマイグレーションファイルは`down`になっている

つまり、`rails g model`、`rails g migration`の後マイグレーションファイルをDBに反映（up）にするには、`rails db:migrate`が必要になる

---

# permit（ストロングパラメータ）にシンボルとハッシュを渡す時の違い
```Ruby
def memory_params
  params.require(:memory).permit(:title, :body, :public_flag, :image, multiple_images: [])
end
```
このコードをよく見ると、途中まではシンボル（`:title`など）が並んでいて、最後だけ`multiple_images: [] `という「キー: 値」の形になっています
- `:image`のようにシンボル単体で渡すのは、「 1つの値（文字列や数値など）を許可する」という意味
- `multiple_images: []`のようにハッシュで渡すのは、「配列やネストされたデータ構造を許可する」という意味
## なぜこんな区別が必要か
- `params`の中に入ってくるデータは、実際には「単純な1つの値」なのか「複数の値をまとめた配列」なのか「さらに別のハッシュ」なのか、様々な形がありえる
- もし`permit`がシンボルだけしか受け取れなかったら、配列やネストしたデータをどう許可すればいいか、Rails 側は判断できなくなってしまう
- この区別はストロングパラメータがセキュリティを守るための仕組みだからこそ必要なものになっている
 - 安全のためには、「配列の中に何が入ってくるか」まで含めて、開発者が明示的に許可を与える必要があるということ
## 【番外編】`require(:memory)`と`permit(...)`はそれぞれ何をしているの
- `require(:memory)`
 - データの中から特定のモデル名（キー）を指定し、必須のオブジェクトを絞り込む
 - 指定したキーが存在しないならエラーを出してくれる

- `permit`メソッド
 - 絞り込んだデータから変更・保存を許可するカラム（キー）を指定する
 - リストに含まれていない余計なパラメータは無視（フィルター）される

# `memory.memory_images.first&.image&.url`を解説

## `memory.memory_images`
Memoryモデルには`has_many :memory_images`という関連付け（アソシエーション）が定義されています。これは「1つのMemoryに対して、複数のMemoryImageが紐づいている」という関係を表すRailsの機能です。

`memory.memory_images`と書くと、「このmemoryに紐づいているMemoryImageのレコードを全部ちょうだい」という意味になり、結果は配列のようなもの（`ActiveRecord::Associations::CollectionProxy`という、配列っぽく扱えるオブジェクト）で返ってきます。

## `.first`
配列の先頭の要素を取り出すメソッドです。ここでは「一番最初に保存されたMemoryImage」を取得しています。

ただし注意点があります。もしそのmemoryにまだ画像が1枚も保存されていなければ、`memory_images`は空の配列になり、`.first`の結果はnilになります。

## 1つ目の`&.`（なぜ必要か）
&.は「ぼっち演算子」や「セーフナビゲーション演算子」と呼ばれるRubyの機能です。

もし普通に`memory.memory_images.first.image`と書いてしまうと、画像が1枚もない場合に.firstがnilを返し、そのnilに対して.imageを呼び出そうとして

```
NoMethodError: undefined method 'image' for nil:NilClass
```

というエラーになります。

&.を使うと、「呼び出す相手（レシーバー）がnilだったら、メソッドを呼ばずにそのままnilを返す」という安全な動きになります。つまり`.first`がnilだった場合、`&.image`は.imageを実行せずにnilを返して処理を続けます。

## .image
MemoryImageモデルには`mount_uploader :image`, ImageUploaderという記述があります。これはCarrierWaveというgem（画像アップロード機能を提供するライブラリ）の機能で、「このMemoryImageが持っている画像ファイルを扱うためのオブジェクト（アップローダー）」を返すメソッドを自動的に作ってくれています。

つまり`memory_image.image`は、画像ファイルそのものではなく「画像ファイルを操作するための入れ物」のようなオブジェクトが返ってくる、とイメージしてください。

## 2つ目の&.（なぜ必要か）
先ほど3で説明した通り、もし`.first`がnilだった場合、`&.image`は`.image`を呼ばずにnilを返します。そのまま何もせず.url（普通のドット）を呼んでしまうと、またnilに対してメソッドを呼ぶことになり、同じようにエラーになってしまいます。

&.はその1回の呼び出しだけを守る仕組みなので、チェーン（`.first`→`.image`→`.url`のようにメソッドを連続でつなげる書き方）の各段階で、nilになる可能性がある箇所には毎回&.をつける必要があります。

## .url

アップローダーオブジェクト（4で説明したもの）に対して呼ぶメソッドで、実際にブラウザで表示できる画像のURL（文字列）を返します。

## まとめ
|部分|意味|
|`memory.memory_images`|このMemoryに紐づく画像レコード全部|
|`.first`|先頭の1件（なければnil）|
|`&.image`|先頭がnilでなければ、アップローダーオブジェクトを取得|
|&.url|それがnilでなければ、実際の画像URLを取得|

つまり全体としては「画像が1枚もなければnil、あれば先頭画像のURL文字列」を安全に取得する1行、ということになります。
