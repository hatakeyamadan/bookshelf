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