## チュートリアルのURL

https://laravel.com/docs/9.x/routing

## メモ

### ルーティングの基本

- routesディレクトリにあるルートファイルで定義される
  - 例: web.php
    - Webインターフェースのためのルートを定義する
    - セッション状態やCSRF保護などの機能を提供するWebミドルウェアグループが割り当てられる
  - 例: api.php
    - LaravelをAPIとして使う場合のルートを定義する
    - Apiミドルウェアグループが割り当てられる
    - RouteServiceProviderによってルートグループ内にネストされる
    - プレフィクス(デフォルトは/api)やオプションを変更する場合は、RouteServiceProviderクラスを変更する

- 以下のHTTPメソッドに応答するルートを登録できる
  - get, post, put, patch, delete, options
  - matchを使うことで、複数のHTTPメソッドに応答するルートを登録できる
    - 例: ```Route::match(['get','post'], '/', function(){});```
  - anyを使うことで、すべてのHTTPメソッドに応答するルートを登録できる
    - 例: ```Route::any('/', function(){});```
  - redirectを使うことで、別のURIへリダイレクトさせることができる
    - 例: /here へアクセスしてきたら /there へリダイレクトさせる(HTTPステータスコードは302)
      - ``` Route::redirect('/here', '/there'); ```
    - 例: /here へアクセスしてきたら /there へリダイレクトさせる(HTTPステータスコードは301)
      - ``` Route::redirect('/here', '/there', 301); ``` 
      - ``` Route::permanentRedirect('/here', '/there'); ``` 
  - 同じURIメソッドを共通する複数のルートを定義する場合は、any,match,redirectのメソッドを使ったルートはそれ以外のメソッドを使ったルートの後に定義する必要がある

- ルートが必要とする依存関係を宣言したうえで、サービスコンテナによって解決させることができる
  - 例: Requestクラスをルートコールバックに注入する
```php
use Illuminate\Http\Request;
 
Route::get('/users', function (Request $request) {
    // ...
});
```

- Webルートファイルで定義されている以下のHTTPメソッドを指すHTMLファームにはCSRFトークンを含めなければいけない
  - post,get,put,patch,delete
  - CSRFトークンを含めないとリクエスト拒否される
```php
<form method="POST" action="/profile">
    // CSRFトークン
    @csrf
    ...
</form>
```

- Route::viewを使って、ビューだけ返すことができる
```php
// 第一引数: URI(必須)
// 第二引数: ビュー名(必須)
// 第三引数: ビューに渡すデータの配列(オプション) 
Route::view('/welcome', 'welcome', ['name' => 'Taylor']);
```

- artisanコマンドを使用して、アプリケーションに定義されたすべてのルートの概要を取得できる
```bash
# アプリケーションに定義されたすべてのルートの概要を表示する
php artisan route:list

# アプリケーションに定義されたすべてのルートの概要と各ルートに割り当てられたルートミドルウェアを表示する
php artisan route:list -v

# 指定したURI(例: apiから始まるURI)のみ表示
php artisan route:list --path=api

# サードパーティのパッケージで定義されているルートを除いて表示する
php artisan route:list --except-vendor

# サードパーティのパッケージで定義されたルートのみを表示する
php artisan route:list --only-vendor
```


### パラメータ

- ルートパラメータを使って、ルートのURIから特定のデータを取得するようにできる
```php
use Illuminate\Http\Request;

// ルートパラメータは、{}で囲んでおく(ルートパラメータにはアルファベットとアンダースコアが使える)
// ルートパラメータの順序に応じて、コールバックの引数に渡される
// /posts/1/comments/10 だと postIdには1、commentIdには10が渡される
Route::get('/posts/{post}/comments/{comment}', function ($postId, $commentId) {
    //
});

// サービスコンテナをコールバックで使いたい場合は、サービスコンテナのあとにルートパラメータをリストする必要がある
// Route::get('/user/{id}', function ($id, Request $request) のように定義するとダメ
Route::get('/user/{id}', function (Request $request, $id) {
    return 'User '.$id;
});

// URIに存在しない可能性があるルートパラメータには?を付けて、デフォルト値を指定する必要がある
Route::get('/user/{name?}', function ($name = 'John') {
    return $name;
});
```

- whereを使って、ルートパラメータを正規表現で定義することができる
  - 正規表現に引っかからなければ、404が返る

```php
Route::get('/user/{id}/{name}', function ($id, $name) {
    //
})->whereNumber('id')->whereAlpha('name');

// ルートパラメータとして、スラッシュ(/)を含めてよいことを明示的に指定する(デフォルトでは、スラッシュは含めない)
// ルートパラメータにスラッシュを含められるのは、URIに含まれるルートパラーメータのうち最後のものだけ
// 例: /search/{id}/{search} だと idにはスラッシュを含められない
Route::get('/search/{search}', function ($search) {
    return $search;
})->where('search', '.*');
```

- RouteServiceProviderクラスのbootメソッドを使って、特定の正規表現によってルートパラメータを常に制約をかけることができる
```php
// App\Providers\RouteServiceProvider
public function boot()
{
    // パラメータ名がidのルートパラメータは0から9までの数字を1回以上繰り返したものに制約する
    Route::pattern('id', '[0-9]+');
}
```

```php
// idは0から9までの数字を1回以上繰り返したものだと自動的に定義される
Route::get('/user/{id}', function ($id) {
    // 
});
```

### 名前付きルート

- 名前付きルートを使用することで、特定のルートのURLまたはリダイレクトを生成できる
  - ルート名は一意である必要がある
```php
Route::get('/user/profile', function () {
    //
})->name('profile');

// 特定のコントローラのアクションに対しても名前付きルートを付けられる
Route::get(
    '/user/profile',
    [UserProfileController::class, 'show']
)->name('profile');

// profileという名前が付いたルートへのURLを生成する
$url = route('profile');

// profileという名前が付いたルートへのリダイレクトを生成する
return redirect()->route('profile');
return to_route('profile');
```

```php
Route::get('/user/{id}/profile', function ($id) {
    //
})->name('profile');
 
// profileという名前が付いたルートへ引数を渡す
$url = route('profile', ['id' => 1]);

// profileという名前が付いたルートへ引数を渡す
// photosはルートパラメータとして定義されていないため、クエリストリングとして追加される
// /user/1/profile?photos=yes
$url = route('profile', ['id' => 1, 'photos' => 'yes']);
```

- Routeインスタンスのnamedメソッドを使用することで、現在のルートが指定されたルートにルーティングされたかどうかを判断できる

```php
public function handle($request, Closure $next)
{
    if ($request->route()->named('profile')) {
        //
    }
 
    return $next($request);
}
```

### ルートグループ

- ミドルウェア
  - middlewareメソッドを使って、グループ内のルートにミドルウェアを割り当てることができる
  - 配列に記載された順番でミドルウェアが実行される
```php
// 「/」と「/user/profile」にはfirstミドルウェアとsecondミドルウェアが割り当てられる
Route::middleware(['first', 'second'])->group(function () {
    Route::get('/', function () {
        //
    });
 
    Route::get('/user/profile', function () {
        //
    });
});
```

- コントローラ
  - controllerメソッドを使って、グループ内のルートに共通のコントローラを定義できる
  - 各ルートは、そのルートで使うコントローラメソッドだけを書けばよくなる
```php
use App\Http\Controllers\OrderController;
 
// 「/order/{id}」と「/orders」はOrderControllerを使う
Route::controller(OrderController::class)->group(function () {
    // OrderControllerのshowメソッドを使う
    Route::get('/orders/{id}', 'show');
    // OrderControllerもstoreメソッドを使う
    Route::post('/orders', 'store');
});
```

- サブドメイン
  - domainメソッドを使って、サブドメインを指定することができる
  - ルートドメインを登録する前にサブドメインを登録する必要がある
    - ルートドメインとサブドメインで同じURIパスを持っていた場合に、ルートドメインが先に登録されているとサブドメインのURIパスまで到達しないため
```php
// example.comのサブドメインとして、「{account}.example.com」を定義する
Route::domain('{account}.example.com')->group(function () {
    // サブドメインのルートパラメータを使うことができる
    Route::get('user/{id}', function ($account, $id) {
        //
    });
});
```

- プレフィクス
  - prefixメソッドを使って、グループ内のルートのURIに特定のプレフィクスを付けることができる
```php
// 「/admin/users」というURIにマッチするようになる
Route::prefix('admin')->group(function () {
    Route::get('/users', function () {
      //
    });
});
```

- 名前付きルートのプレフィクス
  - nameメソッドを使って、グループ内のルート名に特定のプレフィクスを付けることができる
```php
Route::name('admin.')->group(function () {
    // 「admin.users」という名前付きルートになる
    Route::get('/users', function () {
        // 
    })->name('users');
});
```

### ルートモデルバインディング

- 暗黙的なバインディング
  - ルートやコントローラのアクションで定義されたEloquentモデルのうち、タイプヒントの変数名がルートセグメント名と一致するものを自動的に解決する
```php
use App\Models\User;

// ルートセグメント名のuserと、タイプヒント付きの変数名$userが一致しているため、
// userに入っている値をIDとして持つUserインスタンスを自動解決する
// 例: /users/1 なら User.id が1の User クラスのインスタンスがクロージャに渡される
// 一致するクラスのインスタンスが見つからない場合は、404を返す
Route::get('/users/{user}', function (User $user) {
    return $user->email;
});
```
```php
use App\Http\Controllers\UserController;
use App\Models\User;


Route::get('/users/{user}', [UserController::class, 'show']);

// ルートセグメント名のuserと、UserControllerのshowメソッドの引数名$userが一致しているため、
// ルートのときと同様にUserインスタンスが自動解決される
public function show(User $user)
{
    return view('user.profile', ['user' => $user]);
}
```

  - 論理削除したモデルの扱い
    - 暗黙的なバインディングでは論理削除されたモデルは取得されない
    - withTrashedを使うことで、論理削除されたモデルも取得できる
```php
use App\Models\User;

// 例: User.id が 100 のUserインスタンスは論理削除されている場合に、/users/100 にアクセスすると、
// 通常は404が返るが、withTrashedをルートの定義にチェーンすることでUserインスタンスが取得できるようになる
Route::get('/users/{user}', function (User $user) {
    return $user->email;
})->withTrashed();
```
  - キーのカスタマイズ
  
```php
use App\Models\Post;
use App\Models\User;

// Postインスタンスを解決する際に、キーとしてidではなくslugカラムを使用するように設定する
Route::get('/posts/{post:slug}', function (Post $post) {
    return $post;
});

// 特定のユーザのスラッグ(記事を識別するIDのようなもの)を持つPostインスタンスを解決するようにする
Route::get('/users/{user}/posts/{post:slug}', function (User $user, Post $post) {
    return $post;
});

// Postインスタンスを解決する際のキーを設定していない場合に、Laravelでよしなに解決するよう設定する方法
Route::get('/users/{user}/posts/{post}', function (User $user, Post $post) {
    return $post;
})->scopeBindings();

// ルートグループ全体に対して、スコープバインディング(Laravelでよしなに解決する機能)を使用するよう設定する方法
Route::scopeBindings()->group(function () {
    Route::get('/users/{user}/posts/{post}', function (User $user, Post $post) {
        return $post;
    });
});

// スコープバインディングを使用しないよう明示的に設定する方法
Route::get('/users/{user}/posts/{post:slug}', function (User $user, Post $post) {
    return $post;
})->withoutScopedBindings();
```

- 解決する際に常にid以外のカラムを主キーとして使用する場合は、EloquentモデルのgetRouteKeyNameメソッドで定義する
```php
class Post extends Model{
  /**
   * Get the route key for the model.
   *
   * @return string
   */
  public function getRouteKeyName()
  {
      return 'slug';
  }
}
```
  - missingメソッドを使って、バインディングされたモデルが見つからない場合の動作を定義できる
```php
use App\Http\Controllers\LocationsController;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Redirect;
 
// Locationインスタンスの解決に失敗した場合に、locations.index へリダイレクトする
Route::get('/locations/{location:slug}', [LocationsController::class, 'show'])
        ->name('locations.view')
        ->missing(function (Request $request) {
            return Redirect::route('locations.index');
        });
```

- 暗黙的なバインディング(Enum)

```php
use App\Enums\Category;
use Illuminate\Support\Facades\Route;

// enum Category: string
// {
//     case Fruits = 'fruits';
//     case People = 'people';
// }

// Categoryとして上のような定義があった場合に、 GET /categories/fruits か GET /categories/people 以外はHTTPステータスコードとして404を返す
Route::get('/categories/{category}', function (Category $category) {
    return $category->value;
});
```

- 明示的なバインディング
  - RouteServiceProviderクラスのbootメソッドでモデルの解決方法を明示的を定義できる。

```php
use App\Models\User;
use Illuminate\Support\Facades\Route;
 
/**
 * Define your route model bindings, pattern filters, etc.
 *
 * @return void
 */
public function boot()
{
    // ルートパラメータとしての{user}は、Userインスタンスとして解決するよう定義
    Route::model('user', User::class);
 
    // ルートパラメータとしての{user}は、User.name == value に最初にヒットするUserインスタンスとして解決するよう定義
    Route::bind('user', function ($value) {
        return User::where('name', $value)->firstOrFail();
    });
}
```

  - モデルの解決方法はEloquentモデルの resolveRouteBinding と resolveChildRouteBinding でも定義できる
```php
class User extends Model{
  // Userインスタンスの解決方法を定義
  /**
   * Retrieve the model for a bound value.
   *
   * @param  mixed  $value
   * @param  string|null  $field
   * @return \Illuminate\Database\Eloquent\Model|null
   */
  public function resolveRouteBinding($value, $field = null)
  {
      return $this->where('name', $value)->firstOrFail();
  }

  // 暗黙的なバインディングにおけるscopeBindingを利用している場合に、親モデルの子バインディングを解決する方法を定義
  /**
   * Retrieve the child model for a bound value.
   *
   * @param  string  $childType
   * @param  mixed  $value
   * @param  string|null  $field
   * @return \Illuminate\Database\Eloquent\Model|null
   */
  public function resolveChildRouteBinding($childType, $value, $field)
  {
      return parent::resolveChildRouteBinding($childType, $value, $field);
  }
}
```

### フォールバック

- fallbackメソッドを使って、受信したリクエストに一致するルートがない場合に実行されるルートを定義できる
  - fallbackメソッドを使ったルート定義は、一番最後に登録される必要がある
```php
Route::fallback(function (){
  //
});
```

### レート制限

- レート制限の定義
  - 特定のルートまたはルートグループに対してレート制限を行うことができる。
  - App/Providers/RouteServiceProviderクラスのconfigureRateLimitingメソッド内にレート制限について定義できる
  - レート制限を超えるとHTTPステータスコードとして429を返す

```php
use Illuminate\Cache\RateLimiting\Limit;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\RateLimiter;
 
/**
 * Configure the rate limiters for the application.
 *
 * @return void
 */
protected function configureRateLimiting()
{
    // 1分あたり1000リクエストを受け付けるレート制限を「sample1」という名前で定義
    RateLimiter::for('sample1', function (Request $request) {
        return Limit::perMinute(1000);
    });

    // 1分あたり1000リクエストを受け付けそれを超えた場合にカスタムされたレスポンスを返すレート制限を「sample2」という名前で定義
    RateLimiter::for('sample2', function (Request $request) {
        return Limit::perMinute(1000)->response(function (Request $request, array $headers) {
          return response('Custom response...', 429, $headers);
        });
    });

    // 1分あたり100リクエスト(VIPカスタマーの場合は制限なし)を受け付けるレート制限を「sample3」という名前で定義
    RateLimiter::for('sample3', function (Request $request) {
        return $request->user()->vipCustomer()
                    ? Limit::none()
                    : Limit::perMinute(100);
    });


    // 以下のレート制限を「sample4」という名前で定義
    // - 認証されたユーザIDが存在すれば、そのユーザIDごとに1分あたり100リクエストを受け付ける
    // - そうでなければ、IPアドレスごとに1分あたり10リクエストを受け付ける
    RateLimiter::for('sample4', function (Request $request) {
        return $request->user()
                    ? Limit::perMinute(100)->by($request->user()->id)
                    : Limit::perMinute(10)->by($request->ip());
    });

    // 以下のレート制限を「sample5」という名前で定義
    // 1分間に500リクエストまたはリクエストにメールアドレスの入力が含まれていれば1分間に3リクエストを受け付ける
    RateLimiter::for('sample5', function (Request $request) {
        return [
            Limit::perMinute(500),
            Limit::perMinute(3)->by($request->input('email')),
        ];
    });
}
```

- レート制限へのアタッチ
  - throttleミドルウェアを使って、ルートまたはルートグループに対してレート制限を割り当てることができる
```php
// 「sample4」というレート制限名を /audio エンドポイントに対して割り当てる
Route::middleware(['throttle:sample4'])->group(function () {
    Route::post('/audio', function () {
        //
    });
});
```

  - Redisを使ってレート制限を管理する場合は、App\Http\Kernel のマッピングを以下のように変更する。
```php
// 通常は throttleミドルウェアは ThrottleRequests がマッピングされている
// 'throttle' => \Illuminate\Routing\Middleware\ThrottleRequests::class,
'throttle' => \Illuminate\Routing\Middleware\ThrottleRequestsWithRedis::class,
```

### フォームメソッドスプーフィング

- HTMLフォームはput,patch,deleteメソッドをサポートしていないため、非表示なフィールドでメソッドを指定する _methodを追加する必要がある

```html
<form action="/example" method="POST">
    <input type="hidden" name="_method" value="PUT">
    <input type="hidden" name="_token" value="{{ csrf_token() }}">
</form>
```

```blade
<form action="/example" method="POST">
    @method('PUT')
    @csrf
</form>
```

### 現在のルートへのアクセス
- Routeの current、currentRouteName、currentRouteAction メソッドを使用することで、現在のHTTPリクエストを処理しているルートに関する情報にアクセスできる

```php
use Illuminate\Support\Facades\Route;
use Illuminate\Http\Request;
 
Route::get(
    '/users',
    [UserController::class, 'show']
);

// パラメータを含まない現在のURLを取得する
// 例: https://example.com/users
$route = Route::current();

// コントローラ名とアクション名を取得する
// 例: users.show
$name = Route::currentRouteName();

// アクション名を取得する
// 例: App\Http\Controllers\UserController@show
$action = Route::currentRouteAction();
```

### CORS
- OPTIONSリクエストは、HandleCorsミドルウェア(デフォルトで含まれている)によって自動的に処理される
- CORS設定は、config/cors.phpで変更できる

### キャッシュ
- 本番環境へのデプロイ時に以下のコマンドを実行することで、ルートファイルをキャッシュできる
  - php artisan route:cache
  - キャッシュされたルートファイルはリクエストごとに読み込まれるため、ルートの登録にかかる時間がキャッシュしないときと比べて短縮されるメリットがある
- 以下のコマンドを実行することで、キャッシュされたルートファイルをクリアできる
  - php artisan route;clear