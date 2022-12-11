## チュートリアルのURL

https://laravel.com/docs/9.x/facades

## メモ

### ファサードとは？
- サービスコンテナで使用可能なクラスへの静的インスタンスを提供する機能

```php
use Illuminate\Support\Facades\Cache;
use Illuminate\Support\Facades\Route;

// ファサードを用いてRouteクラスとCacheクラスにアクセスする
Route::get('/cache', function () {
    return Cache::get('key');
});
```

### ファサードのヘルパ関数
- Laravelの一般的な機能はファサードで提供されるが、より容易に扱えるようにするグルーバルヘルパ関数がある
  - 例: view, response, url, config

```php
use Illuminate\Support\Facades\Response;
 
// ファサードを用いてRouteクラスとResponseクラスにアクセスする
Route::get('/users', function () {
    return Response::json([
        // ...
    ]);
});
 
// ヘルパ関数のresponse()を用いてResponseクラスにアクセスする
// ヘルパ関数を使用しているため、「use Illuminate\Support\Facades\Response;」は不要
Route::get('/users', function () {
    return response()->json([
        // ...
    ]);
});

```

### ファサード vs 依存性注入(DI)

- 依存性注入は、注入されたクラスの実装を交換可能であるため、モックやスタブを挿入できテストを作成する際に便利
- 通常、静的なクラスメソッドはモック・スタブ化できないが、ファサードならそれらのテストを行える
    - 動的メソッドを使用して、サービスコンテナから解決されたオブジェクトへのメソッド呼び出しをプロキシするため

```php
use Illuminate\Support\Facades\Cache;

Route::get('/cache', function () {
    return Cache::get('key');
});
```

```php
use Illuminate\Support\Facades\Cache;
 
/**
 * A basic functional test example.
 *
 * @return void
 */
public function testBasicExample()
{
    // Cacheのgetメソッドを引数にkeyが渡されたらvalueを返すようモックする
    Cache::shouldReceive('get')
         ->with('key')
         ->andReturn('value');
 
    // Route::get('cache')にアクセスする
    $response = $this->get('/cache');

    // valueが返ってくることを確認する
    $response->assertSee('value');
}
```

### ファサード vs ヘルパ関数
- ファサードとヘルパ関数の間には実質的な違いは無い
- 例: 以下のファサード呼び出しとヘルパ関数を呼び出しは同等

```php
// ファサード
return Illuminate\Support\Facades\View::make('profile');
// ヘルパ関数
return view('profile');
```


```php
Route::get('/cache', function () {
    // cacheヘルパはCacheファサードの裏で動作しているクラスのgetメソッドを呼びだす
    // そのためテストを書く場合は、Cacheファサードのテストを書くときと同じように書ける
    return cache('key');
    // ファサードを使って書くならこう
    // return Cache::get('key');
});
```

### ファサードの仕組み

- Laravelのファサードとカスタムファサードは、Illuminate\Support\Facades\Facade を基底クラスとして拡張する
- Facadeクラスはファサードへの関数呼び出しをサービスコンテンナで依存解決したオブジェクトに送るために、__callStatic()マジックメソッドを使用している

#### 流れ(Cacheの場合)

1. Cacheクラスのgetメソッドを呼び出す 
```php
return Cache::get('key');
```

2. Cacheクラスでサービスコンテナに登録している名前を取得する
```php
// Illuminate\Support\Facades\Cache
class Cache extends Facade
{
    // サービスコンテナに登録している名前を取得する
    protected static function getFacadeAccessor() { return 'cache'; }
}
```

3. 取得した名前に紐付けられているインスタンスの依存解決を行い、getメソッドをそのオブジェクトに対して実行する

### リアルタイムファサード

- アプリケーション内の任意のクラスをファサードのように扱うことができる機能


#### リアルタイムファサードを使用しない場合
- Podcastクラスのpublishメソッドのテストを行う際に、Publisherをメソッドインジェクションすることで、Publisherのモックが書きやすくテストが書きやすくなる
- しかし、publishメソッドを呼び出すたびに、Publisherインスタンスを渡す必要があって面倒

```php
<?php
 
namespace App\Models;
 
use App\Contracts\Publisher;
use Illuminate\Database\Eloquent\Model;
 
class Podcast extends Model
{
    /**
     * Publish the podcast.
     *
     * @param  Publisher  $publisher
     * @return void
     */
    public function publish(Publisher $publisher)
    {
        $this->update(['publishing' => now()]);
 
        $publisher->publish($this);
    }
}
```

#### リアルタイムファサードを使用する場合
- リアルタイムファサードでPublisherをファサードのように扱えるにしたことで、publishメソッドを呼びだす際にPublisherインスタンスを渡す必要が無くなって便利

```php
<?php
 
namespace App\Models;

// ファサードのように扱いたいクラスの名前空間の先頭にFacadesと付ける(今回はPublisherクラスをファサードのように扱う)
// use App\Contracts\Publisher; 
use Facades\App\Contracts\Publisher;
use Illuminate\Database\Eloquent\Model;
 
class Podcast extends Model
{
    /**
     * Publish the podcast.
     *
     * @return void
     */
    public function publish()
    {
        $this->update(['publishing' => now()]);
 
        Publisher::publish($this);
    }
}
```

```php
<?php
 
namespace Tests\Feature;
 
use App\Models\Podcast;
// ファサードのように扱ったクラスの名前空間の先頭にFacadesと付ける(今回はPublisherクラスをファサードのように扱う)
// use App\Contracts\Publisher; 
use Facades\App\Contracts\Publisher;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Tests\TestCase;
 
class PodcastTest extends TestCase
{
    use RefreshDatabase;
 
    /**
     * A test example.
     *
     * @return void
     */
    public function test_podcast_can_be_published()
    {
        $podcast = Podcast::factory()->create();
 
        // 組み込みファサードテストヘルパを使って、メソッド呼び出しのモックが作成できる
        Publisher::shouldReceive('publish')->once()->with($podcast);
 
        $podcast->publish();
    }
}
```
