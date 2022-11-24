## チュートリアルのURL

https://laravel.com/docs/9.x/installation

## Laravelのプロジェクト作成方法

### composerを使う場合

- `composer create-project laravel/laravel [プロジェクト名]` でLaravelプロジェクトを作成できる

  - 例: example-app という名前のプロジェクトを作成する場合は、以下のようなコマンドを実行する

    - `composer create-project laravel/laravel exampel-app`

- 開発用サーバを起動するにはプロジェクトルートに移動して、以下のコマンドを実行する

  - `php artisan serve`

  - 開発サーバへは `http://localhost:8000` でアクセスできる

### コンテナを使う場合

- `curl -s "https://laravel.build/[プロジェクト名]" | bash` でLaravelプロジェクトを作成できる

  - プロジェクトの名前には英数字、ハイフン、アンダースコアしか使えない

  - 例: example-app という名前のプロジェクトを作成する場合は、以下のようなコマンドを実行する

    - `curl -s "https://laravel.build/example-app" | bash`

- 開発用サーバを起動するにはプロジェクトルートに移動して、以下のコマンドを実行する

  - `.vendor/bin/sail up`

  - 開発サーバへは `http://localhost` でアクセスできる

## Sailについての補足

- Sailとは、Laravelをコンテナで使う上で必要な構成や設定を提供してくれるCLIツールである

- Sailを使ってプロジェクトを作成する際に、サービスを指定できる

  - `curl -s "https://laravel.build/[プロジェクト名]?with=
  [サービス名]" | bash` でサービスを指定してプロジェクトを作成できる

    - 例: mariadbとredis を指定してプロジェクトを作成する場合は、以下のようなコマンドを実行する

      - `curl -s "https://laravel.build/example-app?with=mariadb,redis`

  - 利用可能なサービスは以下の9種類

    - mysql, pgsql, mariadb, redis, memcached, meilisearch, minio, selenium, mailhog

  - サービスを何も指定しない場合、以下のサービスが自動で指定される

    - mysql, redis, meilisearch, mailhog, selenium

- `curl -s "https://laravel.build/[プロジェクト名]&devcontainer" | bash` でdevcontainerを使えるようにできる

  - devcontainer: VScodeを経由してコンテナ上にアクセスして開発できるVScodeの機能

## 設定

- config/app.php
  - timezone, localeなどの設定
- .env
  - 環境変数ファイル

## DB

- Laravelはデフォルトでmysqlを使うようになっているが、DBを取り替えて移行することが可能

  - 例: sqliteへ移行する場合
    - 1. `touch database/database.sqlite` で空のデータベースを作成
    - 2. sqliteを使うように .envファイルを変更する
    - 3. `php artisan migrate` でデータベース移行する

