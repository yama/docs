デプロイ
########

アプリケーションが一度完成したら、または、完成する前でさえも、デプロイしたいと
思うでしょう。CakePHP アプリケーションをデプロイするにあたり、いくつかのことを
しなければなりません。

ファイルの移動
==============

git commit とあなたのサーバー上で commit やリポジトリーの pull や clone を作成し、
``composer install`` を実行することを奨励されます。git に関する幾つかの知識や
``git`` や ``composer`` のインストールの知識が必要とされますが、このプロセスは、
ライブラリーの依存関係やファイルやフォルダーのパーミッションについて扱います。

FTP 経由でデプロイするとき、少なくともファイルやフォルダーのパーミッションを
修正しなければならないことを理解してください。

ステージングやデモサーバー (試作品) をセットアップし、あなたの開発環境と同期を保つための
デプロイ技術を使用することもできます。

config/app.php の調整
=====================

app.php、特に ``debug`` の値を調整することは非常に重要なことです。debug を
``false`` に変更することにより、開発に関連する部分で、決して広くインターネットに
晒されるべきでない部分を無効にすることができます。debug を無効とすることにより、
以下の種類のことが変更されます。

* :php:func:`pr()` 、　:php:func:`debug()` 及び :php:func:`dd()` により
  生成されたデバッグメッセージが、無効化されます。
* CakePHP コアのキャッシュが、開発時の 10 秒ごとの代わりに毎年 (約365日ごとに)
  破棄されるようになります。
* エラービューの情報量は少なくなり、一般的なエラーメッセージしか表示されなくなります。
* PHP エラーは表示されなくなります。
* 例外のスタックトレースは無効化されます。

上記に加え、多くのプラグインとアプリケーションの拡張機能は、自らの振る舞いを
修正するために、 ``debug`` を使用します。

環境間でデバッグレベルを動的にセットするため、環境変数に対してチェックを
かけることができます。このことにより、アプリケーションをデバッグ ``true`` の状態で
デプロイすることを避けることができるだけでなく、毎回本番環境にデプロイする度に
デバッグレベルを変更せずに済むこととなります。

例えば、Apache の設定にて、環境変数をセットすることができます。 ::

    SetEnv CAKEPHP_DEBUG 1

それから、**app_local.php** にてデバッグレベルを動的にセットすることができます。 ::

    $debug = (bool)getenv('CAKEPHP_DEBUG');

    return [
        'debug' => $debug,
        .....
    ];

It is recommended that you put configuration that is shared across all
of your application's environments in **config/app.php**. For configuration that
varies between environments either use **config/app_local.php** or environment
variables.

セキュリティのチェック
======================

もしあなたがウェブ上の荒野にアプリケーションを解き放とうとするなら、
何か抜け穴がないかを確認しておくことをお勧めします。

* :ref:`csrf-middleware` コンポーネントまたはミドルウェアを使用していることを確認して
  下さい。
* :doc:`/controllers/components/security` コンポーネントを有効化しておいた方が
  いいかもしれません。フォームの改ざんや一括代入 (mass-assignment) 脆弱性に関する
  問題の発生可能性を削減することができます。
* 各モデルにおいて、正しい :doc:`/core-libraries/validation` ルールが
  有効化されているかどうかを確認して下さい。
* ``webroot`` ディレクトリーのみが公開されており、その他の秘密の部分（ソルト値や
  セキュリティキー等）は非公開でかつユニークな状態となっていることを確認して下さい。

ドキュメントルートの指定
========================

アプリケーションでドキュメントルートを正しく指定することはコードをセキュアに、
またアプリケーションを安全に保つために重要なステップの内の一つです。
CakePHP のアプリケーションは、アプリケーションの ``webroot`` に
ドキュメントルートを指定する必要があります。これによってアプリケーション、
設定のファイルが URL を通してアクセスすることができなくなります。
ドキュメントルートの指定の仕方はウェブサーバーごとに異なります。
ウェブサーバー特有の情報については :ref:`url-rewriting` ドキュメントを見てください。

どの場合においても ``webroot/`` をバーチャルホスト（バーチャルドメイン）の
ドキュメントルートに設定すべきでしょう。これは webroot ディレクトリーの外側のファイルを
実行される可能性を取り除きます。

.. _symlink-assets:

アプリケーションのパフォーマンス改善
====================================

クラスローディングは、アプリケーションのプロセス時間の大部分を占めることがあります。
このような問題を避けるために、アプリケーションがデプロイされたら以下のコマンドを
本番サーバーにて走らせることを推奨します。 ::

    php composer.phar dumpautoload -o

プラグインの画像や JavaScript、CSS ファイルなどの静的なアセットを扱う場合、
``Dispatcher`` を通すことはかなり非効率です。本番環境においては、次のように
シンボリックリンクにすることを強くお勧めします。これは、 ``plugin`` シェルを
利用することで実行できます。 ::

    bin/cake plugin assets symlink

上記のコマンドは、アプリケーション内での ``webroot`` ディレクトリーの適切なパスに対して、
全てのロードされたプラグインの ``webroot`` ディレクトリーのシンボリックリンクします。

もし、あなたのファイルシステムがシンボリックリンクを作成できない場合、
ディレクトリーをシンボリックリンクする代わりにコピーします。また、以下を使用して、
明示的にディレクトリーをコピーすることができます。 ::

    bin/cake plugin assets copy

更新のデプロイ
==============

On each deploy you'll likely have a few tasks to co-ordinate on your web server. Some typical ones
are:

1. Install dependencies with ``composer install``. Avoid using ``composer
   update`` when doing deploys as you could get unexpected versions of packages.
2. Run database `migrations </migrations/>`__ with either the Migrations plugin
   or another tool.
3. Clear model schema cache with ``bin/cake schema_cache clear``. The :doc:`/console-commands/schema-cache`
   has more information on this command.

.. meta::
    :title lang=ja: デプロイ
    :keywords lang=ja: stack traces,application extensions,set document,installation documentation,development features,generic error,document root,func,debug,caches,error messages,configuration files,webroot,deployment,cakephp,applications
