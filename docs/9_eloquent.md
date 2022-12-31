## チュートリアルのURL

https://laravel.com/docs/9.x/eloquent

## メモ

### Eloquentモデルの生成方法
- make:model を使って、Eloquentモデルを生成できる。
  - make:model を使って生成されたモデルは、app/Models ディレクトリ配下に配置される。
  - オプションを指定することでモデルの他にファクトリやコントローラなどが生成できる。

```bash
# Flightという名前のモデルを生成する
php artisan make:model Flight --migration
php artisan make:model Flight -m

# Flightという名前のモデルとそのファクトリを生成
php artisan make:model Flight --factory
php artisan make:model Flight -f
 
# Flightという名前のモデルとそのシーダーを生成
php artisan make:model Flight --seed
php artisan make:model Flight -s
 
# Flightという名前のモデルとそのコントローラを生成
php artisan make:model Flight --controller
php artisan make:model Flight -c
 
# Flightという名前のモデルとそのコントローラ・リソースクラス・フォームリクエストクラスを生成
php artisan make:model Flight --controller --resource --requests
php artisan make:model Flight -crR
 
# Fligtという名前のモデルとそのポリシーを生成
php artisan make:model Flight --policy
 
# Flightという名前のモデルとそのマイグレーションファイル・ファクトリ・シーダー・コントローラを生成する
php artisan make:model Flight -mfsc

# Flightという名前のモデルとそのマイグレーションファイル・ファクトリ・シーダー・ポリシー・コントローラ・フォームリクエストを生成する
php artisan make:model Flight --all
 
# Memeberという名前の中間テーブル用のモデルを生成
php artisan make:model Member --pivot
```

- model:show を使って、特定のEloquentモデルの概要を確認できる。
```bash
# Flightという名前のモデルの利用可能な属性とリレーションシップを含む概要を表示する
php artisan model:show Flight
```

### 規約
#### テーブル名
- テーブル名を明示していない場合、モデル名の複数形かつスネークケースに変換したテーブル名を使用する。
- tableプロパティを使って、モデルに対応するテーブル名を定義できる。

```php
namespace App\Models;
 
use Illuminate\Database\Eloquent\Model;
 
class Flight extends Model
{
    // Flightモデルに対応するテーブル名を「my_flights」と定義する
    protected $table = 'my_flights';
}
```

#### 主キー
- デフォルトの動作
  - idカラムを主キーとして使用する
    - primaryKey プロパティで主キーとして使用するカラムを変更可能
  - 主キーは増加する整数値であるものとして、整数値にキャストする
    - incrementing, keyType プロパティで主キーの制約を変更可能
  - 複合主キーではないものとする

```php
namespace App\Models;
 
use Illuminate\Database\Eloquent\Model;
 
class Flight extends Model
{
    // Flightsモデルの主キーを「flight_id」と定義する
    protected $primaryKey = 'flight_id';

    // 主キーが増加する整数では無いと定義する
    public $incrementing = false;

    // 主キーの型がstring型だと定義する
    protected $keyType = 'string';
}
```

#### UUID & ULIDキー
- モデルの主キーとしてUUIDまたはULIDを使用できる

```php
use Illuminate\Database\Eloquent\Concerns\HasUuids;
use Illuminate\Database\Eloquent\Concerns\HasUlids;
use Illuminate\Database\Eloquent\Model;
use Ramsey\Uuid\Uuid;
 
class Article extends Model
{
    // 順序付けられたUUID(タイムスタンプが先頭に来る)
    // use HasUuids;

    // ULID
    // use HasUlids;

    // UUIDv4形式に沿った文字列を主キーの値とする
    public function newUniqueId()
    {
        return (string) Uuid::uuid4();
    }
    
    // UUIDまたはULIDを受け取るカラムを定義する
    public function uniqueIds()
    {
        return ['id', 'discount_code'];
    }
}
```

#### タイムスタンプ
- デフォルトの動作
  - モデルに対応するテーブルにcreated_atとupdate_atという名前のカラムが存在するものとする
    - CREATED_AT, UPDATED_AT 定数を使って、作成日と更新日を格納するカラムの名前を変更可能
  - モデルの作成・更新時にcreated_atとupdate_atを自動的に更新する
    - timestamps プロパティを使って、自動更新を行わないよう変更可能
    - withoutTimestamps メソッドを使って、update_atを更新せずにモデルを更新できる

```php
namespace App\Models;
 
use Illuminate\Database\Eloquent\Model;
 
class Flight extends Model
{
    // created_atやupdate_atの自動更新を行わないよう定義
    public $timestamps = false;

    // タイムスタンプのフォーマットを定義
    protected $dateFormat = 'U';

    // 作成日と更新日を格納するカラムの名前を定義
    const CREATED_AT = 'creation_date';
    const UPDATED_AT = 'updated_date';
}

Model::withoutTimestamps(fn () => $post->increment(['read']));
```

#### データベース接続
- デフォルトの動作
  - アプリケーション用に構成されたデフォルトのデータベース接続を使用する
    - connectionプロパティを使って、データベースの別の接続を指定する

```php
namespace App\Models;
 
use Illuminate\Database\Eloquent\Model;
 
class Flight extends Model
{
    // Flightモデルはsqfliteを使用するように定義
    protected $connection = 'sqlite';
}
```

#### デフォルト値

```php 
namespace App\Models;
 
use Illuminate\Database\Eloquent\Model;
 
class Flight extends Model
{
    // Flightモデルのdelayedプロパティはfalseをデフォルト値として定義する
    protected $attributes = [
        'delayed' => false,
    ];
}
```

#### 厳密さの設定
- Eloquentモデル全体の設定を AppServiceProvider の boot メソッド内で定義できる

```php
use Illuminate\Database\Eloquent\Model;
 
/**
 * Bootstrap any application services.
 *
 * @return void
 */
public function boot()
{
    // 運用環境ではない場合に、遅延読み込みを行わないよう定義
    Model::preventLazyLoading(! $this->app->isProduction());

    // 運用環境ではない場合に、メソッドを呼び出して入力できない属性を入力しようとした際に例外をスローするよう定義
    Model::preventSilentlyDiscardingAttributes(! $this->app->isProduction());

    // 運用環境ではない場合に、存在しないモデル属性にアクセス使用とした際に例外をスローするよう定義
    Model::preventAccessingMissingAttributes(! $this->app->isProduction());

    // 上記3種の定義をすべて有効にする
    Model::shouldBeStrict(! $this->app->isProduction());
}
```

### 基本的な操作

#### 取得
```php
// Flightモデルに関連付けられたテーブルからレコードをすべて取得する
$flight = Flight::all();

// アクティブ状態のFlightモデルでnameで昇順ソートした際の上位10件を取得する
$flights = Flight::where('active', 1)->orderBy('name')->take(10)->get();

// Flightモデルをリフレッシュする
$flight = Flight::where('number', 'FR 900')->first();
$flight->number = 'FR 456';
$flight = $flight->refresh(); // $flight->refresh(); でも同じ効果が得られる
$flight->number;

// rejectメソッドを使って、コレクションから条件に応じて削除できる
$flights = Flight::where('destination', 'Paris')->get();
$flights = $flights->reject(function ($flight) {
    return $flight->cancelled;
});

// Flightモデルに関連付けられたテーブルからレコードを200件毎取得して処理を行う
Flight::chunk(200, function ($flights) {
    foreach ($flights as $flight) {
        //
    }
});
// 遅延コレクションを利用する場合はこう書く
// foreach (Flight::lazy() as $flight) {
    
// }
// カーソルを利用する場合はこう書く
// foreach (Flight::cursor() as $flight) {
//     //
// }

// departedがtrueのFlightモデルを200件ずつ取得して、departedをfalseにする
Flight::where('departed', true)
    ->chunkById(200, function ($flights) {
        $flights->each->update(['departed' => false]);
    }, $column = 'id');
// 遅延コレクションを利用する場合はこう書く
// Flight::where('departed', true)
//     ->lazyById(200, $column = 'id')
//     ->each->update(['departed' => false]);

// activeが1のFlightモデルのレコード数を取得する
$count = Flight::where('active', 1)->count();
// activeが1のFlightモデルの中で最大のpriceカラムの値を取得する
$max = Flight::where('active', 1)->max('price');
```

- メモ(cursor と lazy の使い分け方)
  - cursor
    - cursorで読み込みたいEloquentモデルにリレーションシップが含まれない場合
      - 単一のEloquentモデルしかメモリに保持しないため
    - 多数のレコードを扱う必要が無い場合
      - メモリ不足になる恐れがあるため
  - lazy
    - cursorのユースケースから外れた場合


- サブクエリ
```php
// すべての目的地とその目的地に最近到着したフライト名を取得する
$tmp = Destination::addSelect(['last_flight' => Flight::select('name')
    ->whereColumn('destination_id', 'destinations.id')
    ->orderByDesc('arrived_at')
    ->limit(1)
])->get();

// 最後のフライトが目的地に到着した時刻に基づいてすべての目的地を並べ替える
$tmp = Destination::orderByDesc(
    Flight::select('arrived_at')
        ->whereColumn('destination_id', 'destinations.id')
        ->orderByDesc('arrived_at')
        ->limit(1)
)->get();
```
- 取得に失敗した場合の処理

```php
// Flightモデルの主キーが1のレコードを取得する
$flight = Flight::find(1);
 
// activeが1のFlightモデルで最初のレコードを取得する
$flight = Flight::where('active', 1)->first();
// $flight = Flight::firstWhere('active', 1);

// Flightモデルの主キーが1のレコードを取得する
// もし取得できなければ、例外を投げる
$flight = Flight::findOr(1, function () {
    // この例外が捕捉されなければHTTPステータスコードとして404を返す
    throw Illuminate\Database\Eloquent\ModelNotFoundException;
});
// $flight = Flight::findOrFail(1); 

// legsが3よりも大きいFlightモデルで最初のレコードを取得する
// もし取得できなければ、例外を投げる
$flight = Flight::where('legs', '>', 3)->firstOr(function () {
    // この例外が捕捉されなければHTTPステータスコードとして404を返す
    throw Illuminate\Database\Eloquent\ModelNotFoundException;
});
// $flight = Flight::where('legs', '>', 3)->firstOrFail();

// nameが「London to Paris」のFlightモデルで最初のレコードを取得する
// もし取得できなければ、以下のプロパティを持つFlightモデルを返す(データベースには永続化済み)
// - name は「London to Paris」
// - delayed は「1」
// - arrival_time は「11:30」
$flight = Flight::firstOrCreate(
    ['name' => 'London to Paris'],
    ['delayed' => 1, 'arrival_time' => '11:30']
);

// nameが「Tokyo to Sydney」のFlightモデルで最初のレコードを取得する
// もし取得できなければ、以下のプロパティを持つFlightモデルを返す(データベースに永続化するには $flight->save() を実行する)
// - name は「Tokyo to Sydney」
// - delayed は「1」
// - arrival_time は「11:30」
$flight = Flight::firstOrNew(
    ['name' => 'Tokyo to Sydney'],
    ['delayed' => 1, 'arrival_time' => '11:30']
);
```

#### 作成・更新
- 作成

```php
// nameが「Tokyo to London」のFlightモデルを作成し、永続化する
$flight = new Flight
$flight->name = 'Tokyo to London';
$flight->save();

// nameが「Tokyo to London」のFlightモデルを作成し、永続化する(createメソッドを使った場合)
$flight = Flight::create([
    'name' => 'Tokyo to London',
]);
```

- 更新
```php
// 主キーが1のFlightモデルのnameを「Paris to London」に更新し、永続化する
$flight = Flight::find(1);
$flight->name = 'Paris to London';
$flight->save();

// activeが1かつdestinationが「San Diego」のFlightモデルのdelayedを1に更新し、永続化する
Flight::where('active', 1)
      ->where('destination', 'San Diego')
      ->update(['delayed' => 1]);
```

- 更新されたかどうかの判断
```php
$user = User::create([
    'first_name' => 'Taylor',
    'last_name' => 'Otwell',
    'title' => 'Developer',
]);
$user->title = 'Painter';

print($user->name); // Painter
// 引数に渡されたプロパティの変更前の値を取得する
print($user->getOriginal('name')); // Taylor

// 引数に渡されたプロパティがモデルが取得されてから変更されたかどうかを判断する
$user->isDirty(['first_name', 'title']); // true
// 引数に渡されたプロパティがモデルが取得されてから変更されていないかどうかを判断する
$user->isClean(['first_name', 'title']); // false
$user->save();
// 引数に渡されたプロパティが現在のリクエストサイクル内でモデルが最後に保存されたときに変更されたかどうかを判断する
$user->wasChanged(['first_name', 'title']); // true
```

- 作成・更新と永続化をセットで行うメソッド(create, fill, update)を使う場合は、fillableとguardedを定義する必要がある

```php
class Flight extends Model
{
    // create, fill, updateメソッドの実行時に、nameには値が代入できる
    // $flight = Flight::create(['id'=>'100', 'name' => 'London to Paris']); を実行するとidは黙って破棄される
    protected $fillable = [
        'name',
    ];

    // create, fill, updateメソッドの実行時に、idには値が代入できない
    // $flight = Flight::create(['id'=>'100', 'name' => 'London to Paris']); を実行するとidは黙って破棄される
    protected $guarded = [
        id,
    ];
}

```

-  作成・更新と永続化をセットで行うメソッドを呼び出して入力できないプロパティを入力しようとした際に例外を吐く設定を、AppServiceProvider の boot メソッド内で定義できる
```php
use Illuminate\Database\Eloquent\Model;
 
/**
 * Bootstrap any application services.
 *
 * @return void
 */
public function boot()
{
    // ローカル環境では作成・更新と永続化をセットで行うメソッドを呼び出して入力できないプロパティを入力しようとした際に例外を吐くよう設定する
    Model::preventSilentlyDiscardingAttributes($this->app->isLocal());
}
```

- updateOrCreate, upsert を使って、既存のモデル更新か新しくモデル作成を行うことができる
```php
// departureが「Oakland」かつdestinationが「San Diego」のFlightモデルを、priceを「99」discountedを「1」に変更して、永続化する
// そのようなFlightモデルが存在しなければ、以下のようなデータを持つFlightモデルを作成して、永続化する
// - departureが「Oakland」
// - destinationが「San Diego」
// - priceが「99」
// - discountedが「1」
$flight = Flight::updateOrCreate(
    ['departure' => 'Oakland', 'destination' => 'San Diego'],
    ['price' => 99, 'discounted' => 1]
);

// 単一のクエリで複数のアップサートを行う場合は、updateOrCreateの代わりにupsertを使う
// 2番目の引数: テーブル内のレコードを一意に識別する列をリストする(このリスト内に主キーまたは一意なカラムが必要)
// 3番目の引数: レコードが存在した場合に更新するカラム名
// departureが「Oakland」かつdestinationが「San Diego」のFlightモデルの priceを「99」に更新して、永続化する。
// departureが「Chicago」かつdestinationが「New York」のFlightモデルのpriceを「150」に更新して、永続化する。
Flight::upsert([
    ['departure' => 'Oakland', 'destination' => 'San Diego', 'price' => 99],
    ['departure' => 'Chicago', 'destination' => 'New York', 'price' => 150]
], ['departure', 'destination'], ['price']);
```

#### 削除
- 物理削除

```php
// deleteメソッドを使って、特定のFlightモデルを削除する
Flight::where('active', 0)->delete();

// Flightモデルに関連づいた他のモデルのレコードも削除する
// $flight->truncate();

// 削除したいFlightモデルの主キーがわかる場合は、destroyメソッドを使って削除する
Flight::destroy([1, 2, 3]);
```

- 論理削除にしたい場合は、モデルの変更とdelete_atプロパティの追加を行う必要がある
```php
namespace App\Models;
 
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;
 
class Flight extends Model
{
    use SoftDeletes;
}
```

```php
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;
 
Schema::table('flights', function (Blueprint $table) {
    $table->softDeletes();
});
 
Schema::table('flights', function (Blueprint $table) {
    $table->dropSoftDeletes();
});
```

```php
// Flightインスタンスが論理削除したかどうかを判断する
$flight->trashed();

// 論理削除されたFlightインスタンスを復元する
$flight->restore();

// Flightインスタンスを完全に削除
$flight->forceDelete();

// 論理削除済みのものも含めてaccount_idが「1」のFlightインスタンスを取得する
$flights = Flight::withTrashed()
                ->where('account_id', 1)
                ->get();
// 論理削除されたもののみでairline_idが「1」のFlightインスタンスを取得する
$flights = Flight::onlyTrashed()
                ->where('airline_id', 1)
                ->get();
```

### 応用的な操作

#### 定期的な削除

- 定期的に削除したいモデルに変更を加える
```php
namespace App\Models;
 
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Prunable;
// use Illuminate\Database\Eloquent\MassPrunable;
 
class Flight extends Model
{
    use Prunable;
    // pruningを使わない場合は、こちらに使うと定期的な削除を効率的に実行できる
    // use MassPrunable;
 
    
    // 定期的な削除対象のインスタンスを定義する
    public function prunable()
    {
        return static::where('created_at', '<=', now()->subMonth());
    }

    // prunableを実行する前に実行される処理を定義する
    // データベースから完全に削除される前に保存されたファイルやモデルに関連付けられた追加のリソースを削除するのに役立つ
    protected function pruning()
    {
        //
    }
}
```

- App\Console\Kernel の scheduleメソッドで、削除の周期を定義する
```php
protected function schedule(Schedule $schedule)
{
    // 1週間毎にmodel:prune artisanコマンドを実行する
    $schedule->command('model:prune')->daily();

    // app/Models ディレクトリ以外にModelファイルを置いている場合、modelオプションでモデルを指定する
    $schedule->command('model:prune', [
        '--model' => [Flight::class],
    ])->daily();

    // 特定のモデルを定期的な削除の対象から外す場合は、exceptオプションでモデルを指定する
    $schedule->command('model:prune', [
        '--except' => [Flight::class],
    ])->daily();
}
```

- 定期的な削除を実行した際に削除されるレコード数を確認する場合は、 `php artisan model:prune --pretend` を実行する

#### 複製
- relicateメソッドを使って、既存のモデルインスタンスの未保存のコピーを作成できる
```php
$shipping = Address::create([
    'type' => 'shipping',
    'line_1' => '123 Example Street',
    'city' => 'Victorville',
    'state' => 'CA',
    'postcode' => '90001',
]);
 
// shippingをコピーして、typeを「billing」に変更する
$billingA = $shipping->replicate()->fill([
    'type' => 'billing'
]);

// shippinhのtype以外をコピーする
$billingB = $shipping->replicate([
    'type',
]);

$billingA->save();
```

#### 制約の適用
- Illuminate\Database\Eloquent\Scopeインターフェースを実装するクラスを定義して、任意のディレクトリに配置する

```php
namespace App\Models\Scopes;
 
use Illuminate\Database\Eloquent\Builder;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Scope;
 
class AncientScope implements Scope
{
    public function apply(Builder $builder, Model $model)
    {
        $builder->where('created_at', '<', now()->subYears(2000));
    }
}
```

- 制約を適用したいモデルのbootedメソッドに制約を追加する
```php
namespace App\Models;
 
use App\Models\Scopes\AncientScope;
use Illuminate\Database\Eloquent\Model;
 
class User extends Model
{
    protected static function booted()
    {
        static::addGlobalScope(new AncientScope);
        // 制約のためにわざわざクラスを作らない場合は、クロージャ内で定義する
        // static::addGlobalScope('ancient', function (Builder $builder) {
        //     $builder->where('created_at', '<', now()->subYears(2000));
        // });
    }
    
    
    public function scopePopular($query)
    {
        return $query->where('votes', '>', 100);
    }

    public function scopeOfType($query, $type)
    {
        return $query->where('type', $type);
    }
}
```

```php
// [グローバルスコープ] この状態でメソッドを呼び出すと制約が適用されたクエリが実行される
User::all(); // = select * from `users` where `created_at` < 0021-12-31 00:00:00

// [ローカルスコープ] この状態でメソッドを呼び出すと制約が適用されたクエリが実行される
User::popular()->get(); // = select * from `users` where `votes` > 100 AND `created_at` < 0021-12-31 00:00:00

// orWhereを使うことで制約を複数個使うことができる
User::popular()->orWhere->ofType('admin')->get();
```

  - 制約の削除
```php
// 削除したい制約クラスを引数として渡すことで、制約を外せる
User::withoutGlobalScope(AncientScope::class)->get();

// 削除したい制約クラスの名前を引数として渡すことで、制約を外せる(クロージャで制約を定義した場合)
User::withoutGlobalScope('ancient')->get();

// 制約クラスをすべて削除する
User::withoutGlobalScopes()->get();

// 削除したい制約クラスを引数として複数個渡すことで、制約を外せる
User::withoutGlobalScopes([
    FirstScope::class, SecondScope::class
])->get();
```

#### 比較
```php
// post と anotherPost が、主キー・テーブル・データベース接続先を持っているか確認する
$post->is($anotherPost); 
$post->isNot($anotherPost);
```

### ライフサイクルに応じた処理
#### クロージャを使った場合
```php
namespace App\Models;
 
use Illuminate\Database\Eloquent\Model;
use function Illuminate\Events\queueable;

 
class User extends Model
{
    /**
     * The "booted" method of the model.
     *
     * @return void
     */
    protected static function booted()
    {
        // createdが実行された時に、実行する処理を定義する
        static::created(function ($user) {
            //
        });

        // deletedが実行された時に、実行する処理を定義する(キューに突っ込んで非同期で処理する)
        static::deleted(queueable(function ($user) {
            //
        }));

    }
}

```

#### オブザーバクラスを使った場合

- make:observer を使って、オブザーバクラスを生成できる
  - 生成されたファイルは App/Observers ディレクトリ配下に置かれる
```bash
php artisan make:observer UserObserver --model=User
```
```php
namespace App\Observers;
 
use App\Models\User;
 
class UserObserver
{

    // トランザクションが確定したあとでイベントハンドラを実行させる設定
    public $afterCommit = true;

    /**
     * Handle the User "created" event.
     *
     * @param  \App\Models\User  $user
     * @return void
     */
    public function created(User $user)
    {
        //
    }
 
    /**
     * Handle the User "updated" event.
     *
     * @param  \App\Models\User  $user
     * @return void
     */
    public function updated(User $user)
    {
        //
    }
 
    /**
     * Handle the User "deleted" event.
     *
     * @param  \App\Models\User  $user
     * @return void
     */
    public function deleted(User $user)
    {
        //
    }
 
    /**
     * Handle the User "restored" event.
     *
     * @param  \App\Models\User  $user
     * @return void
     */
    public function restored(User $user)
    {
        //
    }
 
    /**
     * Handle the User "forceDeleted" event.
     *
     * @param  \App\Models\User  $user
     * @return void
     */
    public function forceDeleted(User $user)
    {
        //
    }
}
```
  
- App\Providers\EventServiceProvider の boot メソッドまたは $observers のどちらかで、オブザーバを登録する
```php
use App\Models\User;
use App\Observers\UserObserver;
 
protected $observers = [
    User::class => [UserObserver::class],
];

/**
 * Register any events for your application.
 *
 * @return void
 */
public function boot()
{
    User::observe(UserObserver::class);
}
```

#### イベントのミュート
```php
// withoutEvents を使って、イベントを送出せずにモデルを操作できる
$user = User::withoutEvents(function () {
    User::findOrFail(1)->delete();
 
    return User::find(2);
});

$user = User::findOrFail(1); 
$user->name = 'Victoria Faith';

// イベントを送出せずにモデルを保存・削除・復元できる
$user->saveQuietly();
$user->deleteQuietly();
$user->restoreQuietly();
```