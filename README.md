# プロジェクト名
BookShelf 書籍レビューアプリ
## 概要
- 模擬案件
- 会員登録機能  
  会員登録
- ログイン機能  
  ログイン
## ER図
## 環境構築手順
### 1. Laravelプロジェクトの作成 (Laravel 10.x)
- docker run --rm -u "$(id -u):$(id -g)" -v "$(pwd):/var/www/html" -w /var/www/html -e COMPOSER_CACHE_DIR=/tmp/composer_cache laravelsail/php82-composer:latest composer create-project laravel/laravel:^10.0 bookshelf-app
### 2. Laravel Sailのインストール
- プロジェクトディレクトリに移動  
  cd bookshelf-app
- Laravel Sailをインストール  
  docker run --rm -u "$(id -u):$(id -g)" -v "$(pwd):/var/www/html" -w /var/www/html -e COMPOSER_CACHE_DIR=/tmp/composer_cache laravelsail/php82-composer:latest composer require laravel/sail --dev
- Sailの設定ファイルをパブリッシュ（MySQLを選択）  
  docker run --rm -u "$(id -u):$(id -g)" -v "$(pwd):/var/www/html" -w /var/www/html -e COMPOSER_CACHE_DIR=/tmp/composer_cache laravelsail/php82-composer:latest php artisan sail:install --with=mysql
- ※M1/M2/M3 Mac（Apple Silicon）をお使いの方：  
  sail up -d 実行時に no matching manifest for linux/arm64/v8 エラーが発生した場合、compose.yaml の mysql サービスに platform: 'linux/amd64' を追加してください。
### 3. .env ファイルの設定
- .env ファイルを開き、データベース接続情報が以下と一致していることを確認します。  
  DB_CONNECTION=mysql
  DB_HOST=mysql
  DB_PORT=3306
  DB_DATABASE=laravel
  DB_USERNAME=sail
  DB_PASSWORD=password
  
  重要: DB_HOST は localhost や 127.0.0.1 ではなく、Dockerコンテナ名である mysql を指定します。
### 4. フロントエンドのセットアップ (Vite & Tailwind CSS)
- 本プロジェクトでは、フロントエンドのスタイリングにTailwind CSSを使用します。  
  以下の手順でセットアップを行ってください。
- 1. NPM依存パッケージのインストール  
  sail npm install  
  ※Sailコンテナが起動していることを確認。起動していない場合は ./vendor/bin/sail up -d を実行
- 2. Alpine.jsのインストール  
  sail npm install alpinejs
- 3. Tailwind CSSと @tailwindcss/forms プラグインのインストール  
  sail npm install -D tailwindcss@^3.4.0 @tailwindcss/forms postcss autoprefixer  
  ※ @tailwindcss/forms はフォーム要素のスタイルをリセットするLaravel標準プラグインです。
- 4. 設定ファイルの生成  
  sail npx tailwindcss init -p
- 5. Tailwind CSSのテンプレートパス設定とforms プラグインの有効化  
  tailwind.config.js を以下の内容で上書きしてください：

  import defaultTheme from 'tailwindcss/defaultTheme';  
  import forms from '@tailwindcss/forms';  
  
  /** @type {import('tailwindcss').Config} */  
  export default {  
      content: [  
          './vendor/laravel/framework/src/Illuminate/Pagination/resources/views/*.blade.php',  
          './storage/framework/views/*.php',  
          './resources/views/**/*.blade.php',  
      ],  
      theme: {  
          extend: {  
              fontFamily: {  
                  sans: ['Figtree', ...defaultTheme.fontFamily.sans],  
              },  
          },  
      },  
      plugins: [forms],  
  };
- 6. 本プロジェクトのresourcesファイルを coachtech-prepared-file/Preparedblade-mockcase-BookShelf リポジトリの Basicブランチ のresourcesファイルと入れ替え  
  
- 7. Vite開発サーバーの起動  
  sail npm run dev  
### 5. phpMyAdminの追加
- compose.yaml を開き、mysql サービスの後に以下の設定を追加してください。  
  
  phpmyadmin:  
      image: 'phpmyadmin:latest'  
      ports:  
          - '${FORWARD_PHPMYADMIN_PORT:-8080}:80'  
      environment:  
          PMA_HOST: mysql  
          PMA_USER: '${DB_USERNAME}'  
          PMA_PASSWORD: '${DB_PASSWORD}'  
      networks:  
          - sail  
      depends_on:  
          - mysql
### 6. Sailの起動とエイリアス設定
-  Sailをバックグラウンドで起動  
  ./vendor/bin/sail up -d
-  エイリアスを設定して 'sail' だけでコマンドを実行できるようにする  
  echo "alias sail='[ -f sail ] && bash sail || bash vendor/bin/sail'" >> ~/.zshrc
- シェルを再起動するか、新しいターミナルを開いてエイリアスを有効にする  
  exec $SHELL
### 7. アプリケーションキーの生成
- ルートで以下のコマンドを実行する  
  sail artisan key:generate
### 8. データベースのマイグレーションと初期データ投入
- 以下のコマンドでテーブルを作成し、初期データを投入します。  
  sail artisan migrate --seed  
  
  ※既存のデータベースをリセットしたい場合は以下を実行してください。  
  sail artisan migrate:fresh --seed  
  
   ※ 日本語化（バリデーション・認証メッセージ）について（基本） :  
  config/app.php の locale を ja にし、lang/ja/ にメッセージファイルを手動配置して行います。  
  laravel-lang/lang などの laravel-lang/* 系パッケージ（composer require laravel-lang/...）は導入しないでください。  
  同系パッケージは 2026年5月のサプライチェーン攻撃でマルウェア配布に悪用された経緯があります。