# Railsのメモ
---
# ImageUploader の役割について
- CarrierWave（画像アップロードを扱うgem）の`Uploader`クラスは、簡単に言うと「画像をどう扱うかのルールブック」
- ImageUploaderの役割は以下
 - 画像をどこ（どのディレクトリ）に保存するか
 - 画像のサイズをリサイズするか、するならどんなサイズに変換するか
 - どんな拡張子（jpg, png など）のファイルだけ許可するか

``` Ruby
// 画像をどこに保存するか
  def store_dir
    "uploads/#{model.class.to_s.underscore}/#{mounted_as}/#{model.id}"
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
