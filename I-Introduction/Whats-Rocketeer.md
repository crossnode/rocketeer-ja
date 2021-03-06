# Rocketeerとは

<!--original
# What's Rocketeer ?
-->

Rocketeerはモダンな開発者のための、速くて簡単なデプロイツールです。もし、*Capistrano* を過去に使っていて、それが何なのかという基本的な部分についてすでに詳しければ、このセクションはスキップしてもOKです。そうでなければ、しばらくお付き合いください。

<!--original
Rocketeer is a fast and easy deploying tool for modern developers. If you've already used *Capistrano* in the past you're already familiar with the gist of what it does, and can probably skip this section. The rest of you, bear with me.
-->

FTPや、SublimeのSFTPプラグイン、あるいは自作のデプロイスクリプトでデプロイしているかにかかわらず、Rocketeerはその負担を自動化してあなたを助けるためにあります。
Rocketeerには**SSH接続が必要**なことに気をつけて下さい — つまり、もし共有ホスティングを利用している場合(かつFTPでファイルをアップロードしている場合)、できることはありません。

<!--original
Whether you've always deployed manually via FTP, plugins like SFTP for Sublime or via custom-made deploy scripts, Rocketeer is here to help you and automate your burden.
Please note that Rocketeer **requires an SSH connection** - meaning if you're on shared hosting (and use FTP to upload files), I probably can't do anything for you.
-->

## タスクランナー

<!--original
## Task runner
-->

本質的には、RocketeerはベーシックなSSHタスクランナーです。サーバとサーバで実行するコマンドを定義して、それらを様々な状況に合わせて走らせます。Rocketeerを**タスク**の(定義された)ドキュメントのように使うこともできます。

<!--original
In its spirit, Rocketeer is a basic SSH task runner, it defines servers, commands to execute on said server, and run them according to various contexts. You can use Rocketeer as such, for this see the **Tasks** documentation.
-->

## デプロイ

<!--original
## Deployments
-->

Rocketeerは、リモートプロジェクトをデプロイし管理するための、ビルトインのタスクを少数提供しています。

<!--original
Rocketeer provides a handful of tasks built-in to deploy and manage your remote projects.
-->

### 基本的なフォルダー

<!--original
### Core folders
-->

基本的な部分で、このパッケージの戦略はCapistranoにインパイアされていますが、よりシンプルなものになっています。
あなたのサーバ上の`root_folder`でRocketeerに何か与えるまで、このフォルダはRocketeerだけの自己完結した小さな世界になっています — どんなものが入ってきたとしても。

<!--original
At its core, this package's strategy is inspired by Capistrano's and is relatively simple.
Before anything you'll provide Rocketeer with a `root_folder` on your server - this folder is Rocketeer's little self-contained world : whatever it does will be in that folder.
-->

それでは、あなたの`application_name`を与えます。これはRocketeerに複数のアプリケーションを同じサーバで実行できるようにするためです。アプリケーション名は、`root_folder`のサブフォルダーを作るのに使われます。*この* アプリケーションに関するすべては、このフォルダの中で起きます。
もし、あなたの`root_folder`が `/var/www/`で、`application_name`が`facebook`だとしたら、Rocketeerは`/var/www/facebook/`を作成して、このプロジェクトに関することはすべてそのフォルダの中で起きることになります。
すべてはパッケージされ、その中に含まれます。沢山のアプリケーションをサーバ上で動かすことができ、しかも(その運用は)すべてスムーズです。

<!--original
Then you give it your `application_name`. This is to let Rocketeer handle having multiple applications on the same server. The application name will be used to create a subfolder in the `root_folder` where everything related to _this_ application will happen.
If per example your `root_folder` is `/var/www/` and your `application_name` is `facebook`, then Rocketeer will create `/var/www/facebook/` and everything it does in the span of this project will happen in that folder.
Everything is packaged and contained, that way you can have as many applications on your server and it will still be all smooth.
-->

### ストラテジー

<!--original
### The strategy
-->

前述のように、Rocketeerのフォルダ設計はCapistranoにインスパイアされています。同じくらい嫌いな側面もあるので、(...?)

<!--original
As I said, Rocketeer's folder architecture is inspired by Capistrano's. Because as much as there are aspects of the latter I dislike, you cannot go against something that's been though out and refined for years either.
-->

上で触れたフォルダ(`/var/www/facebook/`)の中で、Rocketeerは3つのフォルダを作成します。

<!--original
In the folder mentioned above (`/var/www/facebook/`), Rocketeer will create three folders.
-->

**current** にはあなたのアプリケーションの _最新_ バージョンが常に入ります。ほかのフォルダで何が起きようとも、**これがあなたのオンラインに公開したいフォルダです。** もし、最新版以外のリリースを公開する必要があるとしても(たとえば、最新版がバグっているとか)、ここ以外のフォルダを公開することはできません。その場合、Rocketeerのツールを使ってロールバックする必要があります。

<!--original
**current** is where the _latest_ version of your application will always be. No matter what happens in the other folders, **that's the folder you want to serve online**. You never ever serve any other folder than this one, if you need to serve a release that is not the latest one (because the latest one is bugged per example) then you need to use Rocketeer's tools to rollback.
In our case, the Apache directive will look like this :
-->

```apache
```__TEMP-WRAPPER-TAG__
<Directory /var/www/facebook/current/public/>
  Options Indexes FollowSymLinks MultiViews
  Order allow,deny
  Allow from all
  AllowOverride All
</Directory>
```__TEMP-WRAPPER-TAG__
```

二つ目のフォルダー **releases** はあなたのアプリケーションの履歴が保存される場所です。毎回、あなたが`deploy`をヒットするたびに、タイプスタンプ付きのフォルダが _releases_ に作られます(例えば、日時が2013-07-21 01:01:01だったら、20130721010101)。設定ファイルで、履歴をどれたけ深く(古く)まで取っておくか、設定することができます。デフォルトでは、最新の4リリースを保存するようになっています。新しいリリースが作られて公開の準備ができると、Rocketeerはそこを参照するように`current`フォルダの[symlink](http://en.wikipedia.org/wiki/Symbolic_link)を更新します。このやりかたはとても柔軟で、ロールバックの際、Rocketeerは`current`フォルダがどれなのかだけ変更すれば済みます。

<!--original
The second folder, **releases**, is where the history of your application is stored. Every time you hit `deploy`, a timestamped folder will be created in _releases_ (20130721010101 per example for 2013-07-21 01:01:01). You can configure in your config file how deep the history goes : by default it will keep the four latest releases.
Once a new release is created and is ready to be served, Rocketeer will update the [symlink](http://en.wikipedia.org/wiki/Symbolic_link) of the `current` folder to make it point to it. This system is particularly flexible as it allows Rocketeer to simply update what folder `current` points to in case of rollback.
-->

最後、3つ目のフォルダ **shared** は、それぞれのリリース間で共有されるファイルが保存される場所です。私たちのFacebookアプリケーションを例に取るなら、アバターをアップロードできるユーザがいて、`public/users/avatars`に保存されます。再度デプロイをして、Rocketeerが新しくリリースを作るまではまったく問題ありませんが、そのままだとアップロードされたはずの写真が含まれない、元の状態のままです。
この問題を解決するために、設定ファイルの`shared`配列に`public/user/avatars`のようなルートからの相対パスを書くことができます。Rocketeerがそれを見ると、自動的にそのフォルダ`avatars`を`shared`に移動して、デプロイのたびに新しいリリースはその共有フォルダを引き継ぎます。デフォルトで、Rocketeerはアプリケーションのログを引き継いで、発生した例外の履歴を保持しますが、あなたのアプリケーション次第で好きなだけたくさんのフォルダを加えることができます。もし、SQLiteデータベース(データはファイルに保存されます)を使っていたら、それも共有したいはずです。

<!--original
And finally the third folder, **shared**, is where files that are shared between each releases are stored. Take per example our Facebook application, it has users that can upload their avatars on it, and they are stored in `public/users/avatars`. This is all fine until you decide to deploy again and Rocketeer creates a new release pristine folder from scratch where your uploaded images won't be.
To solve this problem, in the config file you have a `shared` array where you can put paths relative to the root folder of your application, like `public/users/avatars`. Once Rocketeer see that, it will automatically move the folder `avatars` to `shared` and from there, every time you deploy, the new release will inherit all the shared folders.
By default Rocketeer always shares the logs of the application so that an history of the Exceptions that occurred is kept, but you can add as many folders as you like depending of your application. If you have an SQLite database that is stored in a file, you might want to share it too.
-->

まとめると、あなたのリモートサーバはこんな感じになるはずです。

<!--original
To sum it up, here is what your remote server will look like :
-->

```
| var
|-- www
  |-- facebook
    |-- current => /var/www/facebook/releases/20130721000000
    |-- releases
    |  |-- 20130721000000
    |  |  |-- app
    |  |     |-- storage
    |  |       |-- logs => /var/www/facebook/shared/app/storage/logs
    |  | 20130602000000
    |-- shared
      |-- app
        |-- storage
          |-- logs
```
