## チュートリアルのURL

https://laravel.com/docs/9.x/frontend

## メモ

## Laravelでフロントエンドを構築する場合

### Laravel Blade

- LaravelでHTMLをレンダリングする際に使用するテンプレート言語
  - Smartyみたいなやつ
- recoures/views に配置したBladeファイルを使用してHTMLをレンダリングすることができる
- この方法でアプリケーションを構築する場合、通常、フォーム送信やその他のページ インタラクションは、サーバーからまったく新しい HTML ドキュメントを受け取り、ページ全体がブラウザーによって再レンダリングされる
  - MPA(Multi Page Action)とも呼ばれている

### Laravel Livewire

- UIの個別の部分をレンダリングして、フロントエンドから呼び出して操作できるメソッドとデータを公開するコンポーネントをBladeファイル上で呼び出せるようにするフレームワーク
- Alpine.jsを使用して、ダイアログウィンドウをレンダリングする場合など、必要な場所にのみJavaScriptをフロントエンドに散布することもある

### StarterKits

- for Laravel Blade
  - https://github.com/laravel/breeze
- for Laravel Livewire 
  - https://github.com/laravel/jetstream
    - Livewire + Blade モードで使用

## Javascriptフレームワークでフロントエンドを構築する場合

### Inertia

- LaravelアプリケーションとVue or Reactフロントエンドとのギャップを埋める機能
- Inertiaを使用することで、ルーティング、データ ハイドレーション、および認証のために Laravel ルートとコントローラーを活用できる
- SSR(Server Side Rendering)にも対応している

### StarterKits

- https://github.com/laravel/jetstream

## アセットのバンドル

- アプリケーションの CSS を本番環境対応のアセットにバンドルする必要がある可能性がある
- アプリケーションのフロントエンドを Vue または React で構築することを選択した場合は、コンポーネントをブラウザ対応の JavaScript アセットにバンドルする必要もある
- LaravelはViteを使ってアセットをバンドルするようになっており、vite.config.js にバンドル時の設定を記載する
