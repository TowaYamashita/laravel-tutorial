## チュートリアルのURL

https://laravel.com/docs/9.x/container

## メモ
<!-- チュートリアルの内容を記入してください -->

### サービスコンテナとは何？

- DI(Dependency Injection)を実現するデザインパターンの一種
- Laravelでは、DIに加えてインスタンス生成の方法のカスタマイズも提供している

### サービスコンテナの利点

- サービスコンテナを利用することで、以下の効果が期待できる
  - 疎結合にできる
  - テスト時にモック実装に差し替えやすくできる

### サービスコンテナの設定

- 大抵の場合は、コンテナにバインドまたは依存関係の解決方法を手動で書く必要は無いが、手動で対応する必要があるパターンもある
  - 例: 具体的なクラスではなくインターフェースに依存するクラスをサービスコンテナで扱いたい場合
    - 依存関係の解決方法をサービスコンテナに伝える必要がある
  - 例: Laravelパッケージを作成する場合
    - 作成したパッケージのサービスをサービスコンテナにバインドする必要がある

### バインディング

- 各サービスプロバイダのregister()内でサービスコンテナへバインディングを行う


```php
use App\Services\Transistor;
use App\Services\PodcastParser;
 
// シンプルなバインディング
// bind()内にバインディングしたいクラスのインスタンスを返すクロージャを与える
$this->app->bind(Transistor::class, function ($app) {
    return new Transistor($app->make(PodcastParser::class));
});

// 一度だけ解決する必要があるクラスまたはインターフェースをバインディングする
// この方法でバインディングした場合、サービスコンテナから呼び出す際に同じインスタンスが返される
$this->app->singleton(Transistor::class, function ($app) {
    return new Transistor($app->make(PodcastParser::class));
});

// 特定のLaravelリクエストまたはジョブサイクル内で1回だけ解決する必要があるクラスまたはインターフェースをバインディングする
// singletonでバインディングする場合とは異なり、Laravelアプリケーションが新しいライフサイクルを開始すると、登録されたインスタンスが初期化される。
// 新しいライフサイクルが開始される例: 
// ・Laravel Octaneワーカーが新しいリクエストを処理するとき
// ・Laravel Queueワーカーが新しくジョブを処理するとき
$this->app->scoped(Transistor::class, function ($app) {
    return new Transistor($app->make(PodcastParser::class));
});

// 既存のインスタンスをバインディングする
$service = new Transistor(new PodcastParser);
$this->app->instance(Transistor::class, $service);
```

#### インターフェースと実装

- 特定のインターフェースに対して実装をバインディングする
- 例: EventPusher(インターフェース)にRedisEventPusher(実装)をバインディングする
  - このようにバインディングすると、EventPusherの実装が必要な際に、RedisEventPusherを使うことができる

```php
use App\Contracts\EventPusher;
use App\Services\RedisEventPusher;
 
$this->app->bind(EventPusher::class, RedisEventPusher::class);
```

- 特定のインターフェースに対して複数の実装を条件によって使い分けるようバインディングする
- 例: Filesystem(インターフェース)を必要とするPhotoController,VideoControler,UploadControllerに対して異なる実装をおこなったインスタンスをバインディングする

```php
use App\Http\Controllers\PhotoController;
use App\Http\Controllers\UploadController;
use App\Http\Controllers\VideoController;
use Illuminate\Contracts\Filesystem\Filesystem;
use Illuminate\Support\Facades\Storage;

// Filesystem(インターフェース)を実装したStorage::disk('local')を渡す
$this->app->when(PhotoController::class)
          ->needs(Filesystem::class)
          ->give(function () {
              return Storage::disk('local');
          });
 
// Filesystem(インターフェース)を実装したStorage::disk('s3')を渡す
$this->app->when([VideoController::class, UploadController::class])
          ->needs(Filesystem::class)
          ->give(function () {
              return Storage::disk('s3');
          });
```

#### プリミティブ値へのバインディング

```php
use App\Http\Controllers\UserController;
 
$this->app->when(UserController::class)
          ->needs('$variableName')
          ->give($value);

// コンフィグから値を取得してバインディングする場合は、giveConfigを使う
$this->app->when(ReportAggregator::class)
    ->needs('$timezone')
    ->giveConfig('app.timezone');
```

#### 型付き変数へのバインディング、タグ付け

- バインディングの特定のカテゴリをすべて解決した上でバインディングする
- 例: Report(インターフェース)を実装したクラスのインスタンスを複数個渡せるReportAggregatorの場合

```php
$this->app->bind(CpuReport::class, function () {
    //
});
 
$this->app->bind(MemoryReport::class, function () {
    //
});
 
// CpuReportとMemoryReportのバインディングに「reports」という名前のタグを付ける
$this->app->tag([CpuReport::class, MemoryReport::class], 'reports');

// 「reports」というタグが付けられたインスタンス(CpuReportとMemoryReport)を解決してから、ReportAggregatorのコンストラクタに渡す
$this->app->when(ReportAggregator::class)
    ->needs(Report::class)
    ->giveTagged('reports');
```

```php
<?php
 
use App\Models\Report;
 
class ReportAggregator
{ 
    /**
     * The reports instances.
     *
     * @var array
     */
    protected $reports;
 
    /**
     * Create a new class instance.
     *
     * @param  array  $filters
     * @return void
     */
    public function __construct(Logger $logger, Report ...$reports)
    {
        $this->reporters = $reports;
    }
}
```

#### バインディングの拡張

- バインディング後のサービスを変更することができる

```php
// クロージャには、解決されるインスタンス($service)とコンテナインスタンス($app)が渡される
$this->app->extend(Service::class, function ($service, $app) {
    return new DecoratedService($service);
});
```

### リゾルブ

- $appにアクセスできる場合

```php
use App\Services\Transistor;

// サービスコンテナからTransitorクラスのインスタンスを取得する
$transistor = $this->app->make(Transistor::class);

// サービスコンテナから必要なコンストラクタの引数を手動で渡した上で、Transitorクラスのインスタンスを取得する
$transistor = $this->app->makeWith(Transistor::class, ['id' => 1]);
```

- $appにアクセスできないまたはサービスプロバイダにアクセスできない場合

```php
use App\Services\Transistor;
use Illuminate\Support\Facades\App;
 
// Appファサードを使用して、Transitorクラスのインスタンスを取得する 
$transistor = App::make(Transistor::class);
 
// appヘルパーを使用して、Transitorクラスのインスタンスを取得する
$transistor = app(Transistor::class);
```

### メソッドの呼び出しと注入

- callメソッドを使う
  - 任意のPHP呼び出しオブジェクトを引数に取れる

```php
use App\UserReport;
use App\Repositories\UserRepository;
use Illuminate\Support\Facades\App;
 
// サービスコンテナからUserReportクラスのgenerateメソッドを呼び出す
// generateはUserRepositoryのインスタンスが必要だが、サービスコンテナがよしなに解決してくれている
$report = App::call([new UserReport, 'generate']);

// サービスコンテナからUserRepositoryのインスタンスを取得して、それを使った処理を行う
$result = App::call(function (UserRepository $repository) {
    // ...
});
```

```php
<?php
 
namespace App;
 
use App\Repositories\UserRepository;
 
class UserReport
{
    /**
     * Generate a new user report.
     *
     * @param  \App\Repositories\UserRepository  $repository
     * @return array
     */
    public function generate(UserRepository $repository)
    {
        // ...
    }
}
```

### コンテナイベント

- オブジェクトの依存関係が解決されるたびにイベントが発生する。
- そのイベントは、resolvingを使ってlitenすることができる

```php
use App\Services\Transistor;

// Transistor型のオブジェクトが解決されるたびに発火する
// クロージャの第一引数には解決されたオブジェクトのインスタンス、第2引数にはコンテナインスタンスが渡される
$this->app->resolving(Transistor::class, function ($transistor, $app) {
  // ..
});

// 何かしらのオブジェクトが解決されるたびに発火する
// クロージャの第一引数には解決されたオブジェクトのインスタンス、第2引数にはコンテナインスタンスが渡される
$this->app->resolving(function ($object, $app) {
  // ..
});
```