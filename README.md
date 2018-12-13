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

次のファイルはそのままコピーして設置します。

- .htaccess
- .gitignore
- common/*
- px-files/*
- sample_pages/*
- index.html


### composer.json に px2package 情報を追加する

今回、`/pickles2_src` ディレクトリに Pickles 2 をセットアップするので、 `path` と `path_homedir` にこのディレクトリパスを設定しておきます。

Laravel ももともと `extra` を設定しているようなので、ここに並べて書きます。

```json
{
    "name": "tomk79/laravel-includes-pickles2",

    ...中略

    "extra": {
        "laravel": {
            "dont-discover": []
        },
        "px2package": {
            "name": "Laravel includes Pickles 2",
            "type": "project",
            "path": "pickles2_src/.px_execute.php",
            "path_homedir": "pickles2_src/px-files/"
        }
    },

    ...中略

}
```


### Pickles 2 のパブリッシュ先のディレクトリを設定する

今回の目的は、 Laravel と同居することでした。 Laravel の `/public` には、静的なファイルを置くことができます。

従って、 Pickles 2 からパブリッシュされる静的なファイルが `/public` に出力されるように設定します。

```php
/** パブリッシュ先ディレクトリパス */
$conf->path_publish_dir = '../public/';
```

ただし、もとからある Laravel 関連のファイルを上書きしたり、消してしまうと困ります。
これらのファイルを Pickles 2 が変更しないように、除外ファイル指定を設定しておきます。

```php
	$conf->paths_proc_type = array(
		'/.htaccess' => 'ignore' ,
		'/.px_execute.php' => 'ignore' ,
		'/px-files/*' => 'ignore' ,
		'*.ignore/*' => 'ignore' ,
		'*.ignore.*' => 'ignore' ,
		'/composer.json' => 'ignore' ,
		'/composer.lock' => 'ignore' ,
		'/README.md' => 'ignore' ,
		'/vendor/*' => 'ignore' ,
		'*/.DS_Store' => 'ignore' ,
		'*/Thumbs.db' => 'ignore' ,
		'*/.svn/*' => 'ignore' ,
		'*/.git/*' => 'ignore' ,
		'*/.gitignore' => 'ignore' ,

		// ↓これらは Laravel が管理するファイル
		'/css/*' => 'ignore',
		'/js/*' => 'ignore',
		'/svg/*' => 'ignore',
		'/favicon.ico' => 'ignore',
		'/index.php' => 'ignore',
		'/robots.txt' => 'ignore',
		'/web.config' => 'ignore',
		'/index.html' => 'ignore',

		'*.html' => 'html' ,
		'*.htm' => 'html' ,
		'*.css' => 'css' ,
		'*.js' => 'js' ,
		'*.png' => 'pass' ,
		'*.jpg' => 'pass' ,
		'*.gif' => 'pass' ,
		'*.svg' => 'pass' ,
	);
```