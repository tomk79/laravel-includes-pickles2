# laravel-includes-pickles2
Laravel 5.7 の中に Pickles 2 の静的なHTMLコンテンツを同居させるサンプルです。

## 目標

[Pickles 2](https://pickles2.pxt.jp/) は、静的なHTMLを書き出すウェブサイト制作ツールですが、 静的なHTMLを書き出すという性格上、 動的なアプリケーション機能を出力するのが苦手です。

この課題を解決する方法として、 [Paprika Framework](https://github.com/pickles2/px2-paprika) のようなプラグインも提供されていますが、 ここでは別の方法として、 Laravel フレームワークと組み合わせて使う方法を模索してみたいと思います。

## 方針

Laravel 5.7 のプロジェクトをベースに、 Pickles 2 のソース環境を埋め込みます。

Pickles 2 からパブリッシュする先のディレクトリは、 `/public` にします。

Laravel の関連ファイルと競合しない形でパブリッシュできたら、 同居運用が可能なはず！


## やったこと

### Laravel をインストール

Laravel 5.7 をインストールしました。
現時点で最新場は 5.7.15 のようです。

```
$ composer create-project --prefer-dist laravel/laravel
```

### composer.json に Pickles 2 の関連パッケージを追加していく

Pickles 2 の標準的な雛形パッケージ [Get Start Pickles 2 !](https://github.com/pickles2/preset-get-start-pickles2)の composer.json を参考に、 Pickles 2 を動かすために必要なパッケージをインストールしていきます。

```
$ composer require broccoli-html-editor/broccoli-module-fess
$ composer require broccoli-html-editor/broccoli-module-plain-html-elements
$ composer require pickles2/px-fw-2.x
$ composer require pickles2/px2-dec
$ composer require pickles2/px2-multitheme
$ composer require pickles2/px2-path-resolver
$ composer require pickles2/px2-px2dthelper
$ composer require pickles2/px2-remove-attr
$ composer require pickles2/px2-sitemapexcel
$ composer require pickles2/px2-publish-ex
```


### Pickles 2 のソース(プレビュー)環境を構築していく

Pickles 2 のソースは、 `/pickles2_src/*` に置くことにします。このディレクトリをドキュメントルートとします。

実行スクリプト `/pickles2_src/.px_execute.php` を設置します。

こちらも Get Start Pickles 2 ! からそのまま持ってきますが、 Composer が作成する vendor フォルダが1つ上の階層に置かれることになるので、 そこだけ書き換えています。

```php
<?php
chdir(__DIR__);
@require_once( '../vendor/autoload.php' );
new picklesFramework2\pickles('./px-files/');
```
