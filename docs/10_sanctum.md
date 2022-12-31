## チュートリアルのURL

https://laravel.com/docs/9.x/sanctum

## メモ

### Sanctumとは？
- 以下のようなトークンベースの認証システムの実装を提供するパッケージ
  - GithubAPIにおけるパーソナルアクセストークンのようなAPIトークンをユーザに発行する機能
  - APIと通信する必要があるSPAを認証する機能

### インストール
- laravelの最新バージョンにはSanctumが含まれているためインストールの必要なし

### 構成
#### デフォルトモデルのオーバーライド
- Sactumによって内部的に使われているSanctumPersonalAccessTokenを拡張できる（通常は拡張の必要なし）
- 拡張したSanctumPersonalAccessTokenは、AppServiceProvider の boot メソッド内でそれを使用するよう設定が必要である

```php
use Laravel\Sanctum\PersonalAccessToken as SanctumPersonalAccessToken;
 
class PersonalAccessToken extends SanctumPersonalAccessToken
{
    // ...
}
```

```php
use App\Models\Sanctum\PersonalAccessToken;
use Laravel\Sanctum\Sanctum;
 
/**
 * Bootstrap any application services.
 *
 * @return void
 */
public function boot()
{
    Sanctum::usePersonalAccessTokenModel(PersonalAccessToken::class);
}
```

### API トークン認証
#### APIトークンの発行
- APIトークンを使用してリクエストする場合は、AutorizationヘッダにBearerトークンとして含める必要がある
- トークン発行の準備のため Laravel\Sanctum\HasApiTokens トレイトを使う必要がある
```php
use Laravel\Sanctum\HasApiTokens;

// ユーザへトークンを発行するためにトレイトを追加
class User extends Authenticatable
{
    use HasApiTokens, HasFactory, Notifiable;
}
```

- createToken メソッドを使って、トークン(Laravel\Sanctum\NewAccessToken)を発行できる
  - 発行したトークンはSHA256でハッシュ化してからデータベースへ格納される
```php
use Illuminate\Http\Request;

Route::post('/tokens/create', function (Request $request) {
    // OAuthのスコープのように、トークンの能力を割り当てられる。
    $token = $request->user()->createToken(
        $request->token_name,
        ['server:update'],
    );
 
    // tokensを参照して、ユーザのすべてのトークンにアクセスできる
    foreach ($user->tokens as $token) {
        //
    }

    return ['token' => $token->plainTextToken];
});
```

- tokenCan メソッドを使って、Sanctumによって認証された受信リクエストのトークンに特定の機能があるかどうかを判定できる
```php
$user->tokenCan('server:update')
```

#### トークン能力
- Sanctumに含まれるabilites, ability ミドルウェアを使って、受信したリクエストが特定の機能を付与されたトークンで認証されたかどうかを確認することができる

```php
// app/Http/Kernel.php
'abilities' => \Laravel\Sanctum\Http\Middleware\CheckAbilities::class,
'ability' => \Laravel\Sanctum\Http\Middleware\CheckForAnyAbility::class,
```

```php
// check-status,place-ordersの両方の機能がトークンに含まれているか確認する
Route::get('/orders', function () {
    // 
})->middleware(['auth:sanctum', 'abilities:check-status,place-orders']);

// check-status,place-ordersのいずれかの機能がトークンに含まれているか確認する
Route::get('/orders', function () {
    // 
})->middleware(['auth:sanctum', 'ability:check-status,place-orders']);
```

#### ルートの保護
- sanctum認証ガードを使うことで、すべてのリクエストが認証されるようルートを保護できる
```php
use Illuminate\Http\Request;
 
Route::middleware('auth:sanctum')->get('/user', function (Request $request) {
    return $request->user();
});
```

#### トークンの取り消し
- delete メソッドを使うことで、データベースがトークンを削除し、トークンを取り消すことができる
```php
// ユーザのすべてのトークンを取り消す
$user->tokens()->delete();
 
// 現在のリクエストの認証に使用されたトークンを取り消す
$request->user()->currentAccessToken()->delete();
 
// ユーザの特定のトークンを取り消す
$user->tokens()->where('id', $tokenId)->delete();
```

#### トークンの有効期限
- Sanctum設定ファイルのexpirationを変更することで、発行したトークンが期限切れになるまでの分数を定義できる(デフォルトではトークンに有効期限は無い)
- sanctum:prune-expired のartisanコマンドを使うことで、有効期限切れのトークンを削除できる
  - これをApp\Console\Kernel の scheduleメソッドで、使うことで定期的に有効期限切れのトークンを削除できる
```php
protected function schedule(Schedule $schedule)
{
    $schedule->command('sanctum:prune-expired --hours=24')->daily();
}
```

### SPA 認証
- SPAとAPIを同じトップレベルドメインを共有していることと、リクエストで Accept:application/json ヘッダーを送信できる必要がある。
- ただ、異なるサブドメインに配置可能である

#### 構成
- Sanctum設定ファイルのstatefulを使用して、SPAがどのドメインからリクエストを行うか設定する必要がある
- Sanctumのミドルウェアをapiのミドルウェアグループに追加する必要がある
  - SPAからのリクエストをLaravelのセッションCookieを使用して認証
  - サードパーティまたはモバイルアプリケーションからのリクエストはAPIトークンを使用して認証
```php
// app/Http/Kernel.php
'api' => [
    \Laravel\Sanctum\Http\Middleware\EnsureFrontendRequestsAreStateful::class,
    'throttle:api',
    \Illuminate\Routing\Middleware\SubstituteBindings::class,
],
```

- SPAとAPIを異なるサブドメインに配置している場合に追加で確認すること
  - アプリケーションの CORS 構成が Access-Control-Allow-Credentials ヘッダーに True という値を返していること
    - アプリケーションの config/cors.php 構成ファイル内の supports_credentials オプションを true に設定することで実現
  - アプリケーションのグローバルな axios インスタンスで withCredentials オプションを有効にしていること
    - resources/js/bootstrap.js ファイルで実行される必要があります
    - Axios を使用していない場合は、独自の HTTP クライアントで同等の以下の設定を行う必要がある
      - `axios.defaults.withCredentials = true;`
  - アプリケーションのセッションクッキーのドメイン構成が、ルートドメインの任意のサブドメインをサポートしていること
    - config/session.php 設定ファイルの中で、ドメインの先頭に .を付けることで実現
      - `'domain' => '.domain.com',`

#### 認証
- /sanctum/csrf-cookie に対してリクエストを送信する事により、CSRF保護の初期化ができる
  - このリクエストの間、Laravelは現在のCSRFトークンを含むXSRF-TOKENクッキーを設定される
  - このトークンは、その後のリクエストでX-XSRF-TOKENヘッダーに渡す必要があるが、一部のHTTPクライアントライブラリは自動的にセットしてくれる。セットされない場合は設定されるXSRF-TOKEN Cookieの値と一致するようにX-XSRF-TOKENヘッダを手動で設定する必要がある

- CSRF保護が初期化されたら、/login に対してPOSTリクエストを送信する必要がある
  - ログイン要求が成功し認証されると、それ以降のリクエストはクライアントに発行したセッションクッキーを介して自動的に認証される

#### ルートの保護
- sanctum認証ガードを使うことで、すべてのリクエストが認証されるようルートを保護できる
```php
use Illuminate\Http\Request;
 
Route::middleware('auth:sanctum')->get('/user', function (Request $request) {
    return $request->user();
});
```

### モバイル アプリケーション認証
- Sanctumトークンを使って、モバイルアプリケーションからAPIへのリクエストを認証することができる
- サードパーティのAPIリクエストを認証するプロセスとはAPIトークンを発行する方法に若干の違いがある

#### APIトークンの発行
- 開始するには、ユーザーの電子メール/ユーザー名、パスワード、およびデバイス名を受け付けるルートを作成し、これらの認証情報を新しいSanctumトークンに交換する
- そのトークンを使用してリクエストする場合は、AutorizationヘッダにBearerトークンとして含める必要がある

```php
use App\Models\User;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Hash;
use Illuminate\Validation\ValidationException;
 
Route::post('/sanctum/token', function (Request $request) {
    $request->validate([
        'email' => 'required|email',
        'password' => 'required',
        'device_name' => 'required',
    ]);
 
    $user = User::where('email', $request->email)->first();
 
    if (! $user || ! Hash::check($request->password, $user->password)) {
        throw ValidationException::withMessages([
            'email' => ['The provided credentials are incorrect.'],
        ]);
    }
 
    return $user->createToken($request->device_name)->plainTextToken;
});
```

#### ルートの保護
- sanctum認証ガードを使うことで、すべてのリクエストが認証されるようルートを保護できる
```php
use Illuminate\Http\Request;
 
Route::middleware('auth:sanctum')->get('/user', function (Request $request) {
    return $request->user();
});
```

#### トークンの取り消し
- delete メソッドを使うことで、データベースがトークンを削除し、トークンを取り消すことができる
```php
// ユーザのすべてのトークンを取り消す
$user->tokens()->delete();
  
// ユーザの特定のトークンを取り消す
$user->tokens()->where('id', $tokenId)->delete();
```

### テスト
- Sanctum::actingAsメソッドを使って、ユーザーを認証しトークンに付与する必要がある機能を指定できる
```php
use App\Models\User;
use Laravel\Sanctum\Sanctum;
 
public function test_task_list_can_be_retrieved()
{
    Sanctum::actingAs(
        User::factory()->create(),
        ['view-tasks']
        // すべての能力をトークンに付与する場合はワイルドカードを指定する
        // ['*']
    );
 
    $response = $this->get('/api/task');
 
    $response->assertOk();
}
```