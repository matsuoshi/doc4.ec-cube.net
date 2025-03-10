---
layout: single
title: 4.0本体バージョンアップ
keywords: howto update
tags: [quickstart, getting_started]
permalink: update
summary : 4.0.0から4.0.6-p1への本体バージョンアップ手順について記載します。
---

本番環境でバージョンアップを行う前に、テスト環境で事前検証を必ず行ってください。
{: .notice--danger}
この手順では、ec-cube.netからダウンロードしたEC-CUBEのパッケージを利用していることを想定しています。
{: .notice--danger}
EC-CUBE本体のコード(app/config/eccube, app/DoctrineMigrations, bin, src, htmlディレクトリ)をカスタマイズしている場合、ファイルが上書きされてしまうため、この手順ではバージョンアップできません。[各バージョンでの変更差分](#各バージョンでの変更差分)を確認して必要な差分を取り込んでください。
{: .notice--danger}
EC-CUBE 4.0.6 以前には[危険度「高」の脆弱性](https://www.ec-cube.net/info/weakness/index.php?level=0&version=4.0)が含まれています。必ず EC-CUBE 4.0.6-p1 以降のバージョンへバージョンアップをお願いします。
{: .notice--danger}

## アップデートプラグインを利用したバージョンアップ方法

- ４系のEC-CUBEのバージョンアップには、アップデートプラグインをご利用いただけます。
- アップデートプラグインは、お使いのEC-CUBEの管理画面の「オーナーズストア/プラグインを探す」から「EC-CUBEアップデートプラグイン」で検索を行い、ご利用ください。

アップデートプラグインでバージョンアップを行なった場合は、以下の手順は必要ありません。
{: .notice--info}

## 作業の流れ
1. サイトのバックアップ
1. メンテナンスモードを有効にする
1. EC-CUBEのソースファイルをバージョンアップしたものに置き換え
1. 個別ファイル差し替え
1. composer.json/composer.lockの更新
1. スキーマ更新/マイグレーション
1. プロキシの再生成
1. フロントテンプレートファイルの更新
1. メンテナンスモードを無効にする
1. その他

## 手順詳細

### 1. サイトのバックアップ

EC-CUBEのインストールディレクトリ以下をすべてバックアップしてください。

お使いのデータベースも全てバックアップしてください。

### 2.メンテナンスモードを有効にする（バージョン4.0.1以降）

EC-CUBEの管理画面へアクセスし、「コンテンツ管理」の「メンテナンス管理」から、メンテナンスモードを有効にしてください。

または、EC-CUBEのルートディレクトリに「.maintenance」ファイルを設置することでメンテナンスモードを有効にすることもできます。

```
[root]
  │
  ├──.maintenance
  │
```

※ メンテナンスモード使用時は、管理画面以外のページにアクセスするとメンテナンス画面が表示されます。

※ この機能は、EC-CUBEのバージョンが「4.0.1」以上でないと使用できません。

### 3. EC-CUBEのソースファイルをバージョンアップしたものに置き換え

EC-CUBEのソースファイルについて、ディレクトリごとにそれぞれバージョンアップしたソースファイルに置き換えていきます。置き換える対象のディレクトリとなるのは、今回のバージョンアップで変更となったものとなります。
（`app/config/eccube` `app/DoctrineMigrations` `bin` `src` `html` `vendor` など）

#### `vendor` ディレクトリ以外の置き換え

置き換える対象となるディレクトリを削除し、バージョンアップするEC-CUBEのバージョンのディレクトリにそれぞれ置き換えてください。

（対象ディレクトリの上書きではなくディレクトリの中のすべてのファイルを置き換える必要があります。古いファイルが残ってしまうと、予期せぬ動作となる恐れがありますので、必ず置き換える対象となるディレクトリごとにすべてのファイルを置き換えるようにしてください。）

```
[root]
  │
  ├──[app/config/eccube]
  ├──[app/DoctrineMigrations]
  ├──[bin]
  ├──[src]
  ├──[html]
  │
```

#### `vendor` ディレクトリの置き換え

`vendor` ディレクトリは削除せず、バージョンアップするEC-CUBEのバージョンの `vendor` ディレクトリで上書きしてください。

（ Web API プラグインなど Symfony bundle を利用の場合に `vendor` ディレクトリを完全に削除してしまうと以降の手順でエラーが発生します。ステップ(5) の `bin/console eccube:composer:require-already-installed` コマンドで `vendor` ディレクトリ内の不要なファイルは削除されます。）

```
[root]
  │
  ├──[vendor]
  │
```

### 4. 個別ファイル差し替え

対象となるバージョンごとに、個別のファイル差し替えが必要です。

下記から差し替え対象ファイルを確認して最新のファイルで上書きしてください。

| バージョンアップ対象 | 差し替え対象ファイル                                                                              |
|----------------------|---------------------------------------------------------------------------------------------------|
| 4.0.0 → 4.0.1        | composer.json<br>composer.lock<br>.htaccess<br>index.php<br>maintenance.php|
| 4.0.1 → 4.0.2        | composer.json<br>composer.lock|
| 4.0.2 → 4.0.3        | composer.json<br>composer.lock<br>.htaccess<br>index.php|
| 4.0.3 → 4.0.4        | composer.json<br>composer.lock<br>.htaccess<br>app/Customize/Resource<br>var/.htaccess|
| 4.0.4 → 4.0.5-p1        | composer.json<br>composer.lock<br>.htaccess<br>robots.txt<br>app/template/plugin|
| 4.0.5 → 4.0.6        | composer.lock<br>symfony.lock<br>.htaccess|
| 4.0.6 → 4.0.6-p1     | .htaccess|

- ※ 差し替え対象に、composer.json/composer.lockがある場合は 上書き後、`composer.json/composer.lockの更新の手順`を実施してください。
- ※ `4.0.0 → 4.0.2` のように複数バージョンをまたぐバージョンアップを行う場合は、`4.0.0 → 4.0.1`→`4.0.1 → 4.0.2` のように段階的なバージョンアップを行ってください。
- ※ FTP等でファイルをアップロードするとパーミッションが変更される可能性があります。[パーミッションの設定について](/permission)を参考にパーミッションの確認をお願いします。
- ※ [公式サイトの脆弱性リスト](https://www.ec-cube.net/info/weakness/)を確認をして該当バージョンで検索し、必要なパッチを適用してください。

上書き後、以下のコマンドでキャッシュの削除を行ってください。

```
bin/console cache:clear --no-warmup
```

### 5. composer.json/composer.lockの更新

この手順は、以下の条件をすべて満たす場合に必要です。そうでなければスキップしてください。

- `個別ファイル差し替え`の差し替え対象に、composer.json/composer.lockが含まれている
- プラグインをインストールしている

以下のコマンドを実行してください。

```
bin/console eccube:composer:require-already-installed
```

packagist等の外部ライブラリを独自にインストールしている場合は、再度requireしてください。

例えば、psr/http-messageをインストールしている場合は、以下のコマンドを実行してください。

```
composer require psr/http-message
```

### 6. スキーマ更新/マイグレーション

スキーマ更新およびマイグレーション機能を利用して、データベースのバージョンアップを行います。

以下のコマンドを実行してください。

**※ 4.0.0 → 4.0.1, 4.0.1 → 4.0.2, 4.0.4 → 4.0.5-p1, 4.0.5 → 4.0.6, 4.0.6 → 4.0.6-p1 へのアップデートでは、スキーマ更新は必要ありません。**

参考：[プラグインが無効の状態だと Doctrine SchemaTool でエンティティ拡張が認識されない](https://github.com/EC-CUBE/ec-cube/issues/4056){:target="_blank"}


スキーマ更新

```
bin/console doctrine:schema:update --force --dump-sql
```

マイグレーション

```
bin/console doctrine:migrations:migrate
```

### 7. プロキシの再生成

4.0.3へのバージョンアップのみ、プロキシの再生成が必要です。そうでなければスキップしてください。

プロキシファイルを削除
```
rm -rf app/proxy/entity/*
```

autoloadファイルの再生成
```
composer dump-autoload
```

プロキシファイルを再生成
```
bin/console eccube:generate:proxies
```

キャッシュファイルの再生成
```
bin/console cache:warmup --env=prod
```

### 8. フロントテンプレートファイルの更新

対象となるバージョンごとに、フロントテンプレートファイル(twig)の更新が必要です。

管理画面のコンテンツ管理もしくは店舗設定＞メール設定から、該当するページ/ブロック/メールテンプレートを編集してください。

変更対象の差分は、以下リンクからご確認いただくが[各バージョンでの変更差分](#各バージョンでの変更差分)からご確認いただけます。

#### 4.0.0 → 4.0.1

|ページ名                               |ファイル名|
|--------------------------------------|---------------|
|会員登録(入力ページ)                     |<a href="../documents/updatedoc/4.0.1/Contact_index_twig.htm" target = "_blank">Contact/index.twig</a>|
|会員登録(入力ページ)                     |<a href="../documents/updatedoc/4.0.1/Entry_index_twig.htm" target = "_blank">Entry/index.twig</a>|
|MYページ/会員登録内容変更(入力ページ)     |<a href="../documents/updatedoc/4.0.1/Mypage_change_twig.htm" target = "_blank">Mypage/change.twig</a>|
|MYページ/お届け先追加                   |<a href="../documents/updatedoc/4.0.1/Mypage_delivery_edit_twig.htm" target = "_blank">Mypage/delivery_edit.twig</a>|
|商品購入                               |<a href="../documents/updatedoc/4.0.1/Shopping_index_twig.htm" target = "_blank">Shopping/index.twig</a>|
|非会員購入情報入力                      |<a href="../documents/updatedoc/4.0.1/Shopping_nonmember_twig.htm" target = "_blank">Shopping/nonmember.twig</a>|
|商品購入/お届け先の追加                  |<a href="../documents/updatedoc/4.0.1/Shopping_shipping_edit_twig.htm" target = "_blank">Shopping/shipping_edit.twig</a>|
|商品購入/お届け先の複数指定(お届け先の追加)|<a href="../documents/updatedoc/4.0.1/Shopping_shipping_multiple_edit_twig.htm" target = "_blank">Shopping/shipping_multiple_edit.twig</a>|

#### 4.0.1 → 4.0.2

|ページ名                               |ファイル名|
|--------------------------------------|---------------|
|MYページ/購入履歴詳細                    |<a href="https://github.com/EC-CUBE/ec-cube/pull/4008/files" target = "_blank">Mypage/history.twig</a>|
|注文受付メール                          |<a href="https://github.com/EC-CUBE/ec-cube/pull/4060/files" target = "_blank">Mail/order.twig</a>|
|注文受付メール(HTML)                    |<a href="https://github.com/EC-CUBE/ec-cube/pull/4060/files" target = "_blank">Mail/order.html.twig</a>|

#### 4.0.2 → 4.0.3

<a href="https://github.com/EC-CUBE/ec-cube/pulls?q=is%3Apr+label%3Aaffected%3Atemplate+is%3Aclosed+milestone%3A4.0.3" target = "_blank">フロントテンプレートファイルの差分</a>

#### 4.0.3 → 4.0.4

<a href="https://github.com/EC-CUBE/ec-cube/pulls?q=is%3Apr+label%3Aaffected%3Atemplate+is%3Aclosed+milestone%3A4.0.4" target = "_blank">フロントテンプレートファイルの差分</a>

#### 4.0.4 → 4.0.5-p1

<a href="https://github.com/EC-CUBE/ec-cube/pulls?q=is%3Apr+label%3Aaffected%3Atemplate+is%3Aclosed+milestone%3A4.0.5" target = "_blank">フロントテンプレートファイルの差分</a>

#### 4.0.5-p1 → 4.0.6、4.0.6 → 4.0.6-p1

フロントテンプレートの更新はありません。

### 9.メンテナンスモードを無効にする（バージョン4.0.1以降）

EC-CUBEの管理画面へアクセスし、「コンテンツ管理」の「メンテナンス管理」から、メンテナンスモードを無効にしてください。

または、EC-CUBEのルートディレクトリに「.maintenance」ファイルを削除することでメンテナンスモードを無効にすることもできます。

※ この機能は、EC-CUBEのバージョンが「4.0.1」以上でないと使用できません。

### 10. その他

#### 4.0.0 -> 4.0.1

4.0.1で実装された[メンテナンス機能](https://github.com/EC-CUBE/ec-cube/pull/3998){:target="_blank"}を利用する場合, .envに以下を記載するか、環境変数として以下の値を定義する必要があります。

```
ECCUBE_LOCALE=ja
ECCUBE_ADMIN_ROUTE=admin
ECCUBE_TEMPLATE_CODE=default

※設定値は記載例です。環境にあわせて変更してください。
```

#### 4.0.1 -> 4.0.2

- [faviconパスの変更](https://github.com/EC-CUBE/ec-cube/pull/4075){:target="_blank"}を利用する場合,  [差分](https://github.com/EC-CUBE/ec-cube/commit/50fcbdea4c66e5eabc03ba1e38ce7952a53ca97d#diff-7cefac9fd3759d999afb711a36b6dad9){:target="_blank"}を適用後、ファイル管理からfaviconファイルをアップロードすることでfaviconの変更を行うことができます。
- [CSS管理](https://github.com/EC-CUBE/ec-cube/pull/4083){:target="_blank"} を利用する場合, [差分](https://github.com/EC-CUBE/ec-cube/commit/7994bd00de19399d7c6a8e22dd280791478b9435#diff-7cefac9fd3759d999afb711a36b6dad9){:target="_blank"}の適用が必要です。
- [Javascript管理](https://github.com/EC-CUBE/ec-cube/pull/4084){:target="_blank"}を利用する場合, [差分](https://github.com/EC-CUBE/ec-cube/commit/008236d28633d803d18e15abecf5a04224d0a4f4#diff-7cefac9fd3759d999afb711a36b6dad9R50){:target="_blank"}の適用が必要です。

#### 4.0.2 -> 4.0.3

- [軽減税率制度に関する対応](https://github.com/EC-CUBE/ec-cube/issues/4183){:target="_blank"}を利用する場合, [差分](https://github.com/EC-CUBE/ec-cube/pulls?q=is%3Apr+is%3Aclosed+label%3A%E8%BB%BD%E6%B8%9B%E7%A8%8E%E7%8E%87%E5%AF%BE%E5%BF%9C){:target="_blank"}の適用が必要です。
  ※合わせて[注意点](/update/4_0_3)もご確認下さい。
- [PDFのロゴ画像変更](https://github.com/EC-CUBE/ec-cube/pull/4216){:target="_blank"} を利用する場合, [差分](https://github.com/EC-CUBE/ec-cube/commit/e8f2952925dda75db5b02ab52bf357f59343ecef){:target="_blank"}の適用が必要です。
- [複数ファイルのアップロード](https://github.com/EC-CUBE/ec-cube/pull/4235){:target="_blank"}を利用する場合, [差分](https://github.com/EC-CUBE/ec-cube/commit/fe5cae800fa4e7f09cbb905e1ef3632b34e41489){:target="_blank"}の適用が必要です。

EC-CUBEのバージョンアップ手順は以上です。

## 各バージョンでの変更差分

バージョンごとの詳細な変更差分は、以下のリンク先で確認することができます。

| バージョン      | 差分ページ                                                                                                             |
|-----------------|------------------------------------------------------------------------------------------------------------------------|
| 4.0.0 → 4.0.1   | [https://github.com/EC-CUBE/ec-cube/compare/4.0.0...4.0.1](https://github.com/EC-CUBE/ec-cube/compare/4.0.0...4.0.1?w=1#files_bucket){:target="_blank"}   |
| 4.0.1 → 4.0.2   | [https://github.com/EC-CUBE/ec-cube/compare/4.0.1...4.0.2](https://github.com/EC-CUBE/ec-cube/compare/4.0.1...4.0.2?w=1#files_bucket){:target="_blank"}   |
| 4.0.2 → 4.0.3   | [https://github.com/EC-CUBE/ec-cube/compare/4.0.2...4.0.3](https://github.com/EC-CUBE/ec-cube/compare/4.0.2...4.0.3?w=1#files_bucket){:target="_blank"}   |
| 4.0.3 → 4.0.4   | [https://github.com/EC-CUBE/ec-cube/compare/4.0.3...4.0.4](https://github.com/EC-CUBE/ec-cube/compare/4.0.3...4.0.4?w=1#files_bucket){:target="_blank"}   |
| 4.0.4 → 4.0.5-p1   | [https://github.com/EC-CUBE/ec-cube/compare/4.0.4...4.0.5-p1](https://github.com/EC-CUBE/ec-cube/compare/4.0.4...4.0.5-p1?w=1#files_bucket){:target="_blank"}   |
| 4.0.5-p1 → 4.0.6   | [https://github.com/EC-CUBE/ec-cube/compare/4.0.5-p1...4.0.6](https://github.com/EC-CUBE/ec-cube/compare/4.0.5-p1...4.0.6?w=1#files_bucket){:target="_blank"}   |
| 4.0.6 → 4.0.6-p1   | [https://github.com/EC-CUBE/ec-cube/compare/4.0.6...4.0.6-p1](https://github.com/EC-CUBE/ec-cube/compare/4.0.6...4.0.6-p1?w=1#files_bucket){:target="_blank"}   |

