![スクリーンショット 2020-01-18 10.30.34.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/281085/f26a20c5-ec73-95ea-86af-e8f32126b56c.png)

[PHP フレームワーク Laravel 入門](https://www.amazon.co.jp/dp/4798060992/)を学習中にデータベースにアクセスできない問題が発生しました。

エラーメッセージは『Database does not exist.(SQL:PRAGMA foreign_keys = ON;)』との表示。

`.env`ファイルの`DB_DATABASE=database.sqlite`をコメントアウトすることで解決したのですが、この記事では詳しい原因と対処法をお伝えします。

## エラーメッセージの意味

『Database does not exist.(SQL:PRAGMA foreign_keys = ON;)』

こちらのメッセージ、意味は「**データベースが存在しません。**」です。
本では誤植があったようで、実際にはデータベースファイル（`database.sqlite`）のパスを指定する必要がありましたが、`DB_DATABASE=database.sqLite`と入力していたため、「データベースが存在しない」というエラーが発生したようです。

## さらなる問題と対処法

誤植に気付き、ファイルパスを絶対パスで記入したところ、`The environment file is invalid!`のエラー。
詳細は`Failed to parse dotenv file due to unexpected whitespace.`（予期しない空白のため、dotenv ファイルの解析に失敗しました。）とのこと。
おそらくファイルパスに日本語が含まれていたため、うまくいかなかったのでしょう。

相対パスでの表記もうまくいかず、対処法を探したところ`.env`ファイルの`DB_DATABASE=〜`をコメントアウトすることでうまくいくとのこと。

```shell:.envビフォー
# 前略
DB_DATABASE=〜
# 後略
```

```shell:.envアフター
# 前略
# DB_DATABASE=〜
# 後略
```

【参考】[laravel にて database.sqlite が存在しない（does not exist）と表示される｜ teratail](https://teratail.com/questions/183959)

実際に書き換えて、サーバーを立ち上げ直したところ、きちんと動作しました。

## 対処法解説

では、なぜ`DB_DATABASE=〜`をコメントアウトすることで、きちんと動作するようになったのでしょうか？

Laravel ではデータベースを指定する際に`config/database.php`から設定を読み込みます。
この`database.php`ではデータベースの指定に以下のようなコードが書かれています。

```php:database.php
# 前略
'database' => env('DB_DATABASE', database_path('database.sqlite')),
# 後略
```

まず、`env()`から見ていきましょう。

### グローバルヘルパー関数 env()

ここで使われている`env()`は Laravel に用意されているグローバルヘルパー関数の１つで、環境変数の値を取得します。
取得できない場合はデフォルト値を返します。

```php
$env = env('APP_ENV');

// APP_ENVがセットされていない場合、第二引数がデフォルト値（'production'）として返る
$env = env('APP_ENV', 'production');
```

`database.php`に書かれている`env('DB_DATABASE', database_path('database.sqlite'))`は、「**環境変数`DB_DATABASE`に保存されている値を取得する！なければ database_path('database.sqlite')の値を使う！**」ということだったんですね。

【参考】[ヘルパ 5.5 Laravel( env() )](https://readouble.com/laravel/5.5/ja/helpers.html#method-env)

では、`DB_DATABASE`をコメントアウトすることで、取得するようになる第二引数`database_path('database.sqlite')`はどういう関数なのでしょうか？

### グローバルヘルパー関数 database_path()

`database_path()`もグローバルヘルパー関数のひとつです。
`database/`ディレクトリの完全パスを返します。
`database/`ディレクトリ  内の指定ファイルへの完全パスを生成することもできます。

```php
$path = database_path();

// databaseディレクトリ内のfactories/UserFactory.phpへの完全パスを生成
$path = database_path('factories/UserFactory.php');
```

`env('DB_DATABASE', database_path('database.sqlite'))`で使われていた`database_path('database.sqlite')`は「**`database/`ディレクトリの`database.sqlite`の完全パスを取得する！**」ということだったんですね。

【参考】[ヘルパ 5.5 Laravel（ database_path() ）](https://readouble.com/laravel/5.5/ja/helpers.html#method-database-path)

### ２つの関数をまとめると

それぞれの関数でやっていることがわかったので、`database.php`に書かれている

```php:database.php
# 前略
'database' => env('DB_DATABASE', database_path('database.sqlite')),
# 後略
```

が何をしているかをまとめると、

「**環境変数`DB_DATABASE`に保存されている値を取得する！なければ`database/`ディレクトリの`database.sqlite`の完全パスを取得する！**」

ということになります。

対処法として行った「`.env`ファイルの`DB_DATABASE=database.sqlite`をコメントアウトする」というのは、**`database_path()`で取得した完全パスをデータベースとして指定するようにする**ということだったんですね！

## その他の対処法

そうなると、「`.env`ファイルの`DB_DATABASE=database.sqlite`をコメントアウトする」以外にも対処法が見えてきますね！

`.env`ファイルの`DB_DATABASE=database.sqlite`をコメントアウトせずに

```php:database.php
# 前略
'database' => env('DB_DATABASE', database_path('database.sqlite')),
# 後略
```

を

```php:database.php
# 前略
'database' => database_path('database.sqlite'),
# 後略
```

のように変更して、環境変数の読み込みをなくして直接`database.sqlite`の完全パスを指定してもきちんと動作するようになりました。

## まとめ

Laravel で『Database does not exist.』のエラーが出た際は、データベースのパスの指定が間違っている可能性があるので、

- `.env`ファイルの`DB_DATABASE=database.sqlite`をコメントアウト

```shell:.env
# 前略
# DB_DATABASE=〜
# 後略
```

もしくは

- 環境変数の読み込みをなくして直接`database.sqlite`の完全パスを指定

```php:database.php
# 前略
'database' => database_path('database.sqlite'),
# 後略
```

を試してみましょう！

## 参考まとめ

- 学習中の書籍
  - [PHP フレームワーク Laravel 入門](https://www.amazon.co.jp/dp/4798060992/)
- 参考サイト
  - [laravel にて database.sqlite が存在しない（does not exist）と表示される｜ teratail](https://teratail.com/questions/183959)
  - [ヘルパ 5.5 Laravel( env() )](https://readouble.com/laravel/5.5/ja/helpers.html#method-env)
  - [ヘルパ 5.5 Laravel（ database_path() ）](https://readouble.com/laravel/5.5/ja/helpers.html#method-database-path)
