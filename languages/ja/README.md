# C++ Boilterplate

C++ Boilerplate は C++ のソースコードを効率的に管理するためのテンプレートです。

## 機能

C++ Boilerplate は次のような機能を実現します。

### 規約化されたディレクトリ構成

ディレクトリ構成は Gradle の C++ プロジェクトにおけるディレクトリ構成から
着想を得ており、分かりやすいソースコード管理を実現します。

### 規約化されたビルドのライフサイクル

Maven のようなライフサイクルを定義しており、
開発時のビルド・テストなどのステップを分かりやすく管理します。

### 特定のビルドシステムに依存しないビルド構成

C++ Boilerplate は Makefile 経由で CMake などのビルドシステムを実行します。
トリガを Makefile にすることで開発者は Make を使いつつ、
バックエンドで使用するビルドシステムは
CMake や QMake などのビルドシステムを自由に選択できます。

この構成は IDE フレンドリです。
CMake を使うプロジェクトであっても Eclipse 用の CMake プラグインを
インストールする必要はありません。
CMake を呼び出す Makefile さえあれば Eclipse からは Makefile プロジェクトとして
取り込むことが可能です。

### Docker コンテナでビルドするための構成

ビルド環境に Docker コンテナを使用することで
特定の環境に依存せずにビルドが実行可能です。
依存ライブラリは Dockerfile で管理するため、
煩わしいライブラリのインストール手順を踏む必要はありません。

### Jenkins パイプラインを実現するための構成

Jenkins 上でのビルドは Jenkinsfile で管理されます。
Jenkinsfile は単純に Make のライフサイクルを実行するだけなので
ローカル環境と Jenkins 環境のビルド差分が発生することはありません。

## 使い方

### 必要なもの

* Linux
* Docker
* Docker Compose

### ディレクトリ構成

基本的なディレクトリ構成は次のようになっています。

```
.
├─doc
├─dockerfiles
│  └─develop
│      └─Dockerfile
├─libs
│  ├─include
│  └─lib
├─src
│  ├─example
│  │  ├─cpp
│  │  │  └─example
│  │  └─headers
│  │      └─example
│  ├─exampleTest
│  │  ├─cpp
│  │  │  ├─example
│  │  │  └─main.cc
│  │  └─headers
│  │      └─example
│  └─main
│      └─cpp
│          └─main.cc
├─.clang-format
├─CMakeLists.txt
├─docker-compose.yml
├─Jenkinsfile
├─Makefile
├─README.md
└─version.h.in
```

#### dockerfiles

ビルドで使用するコンテナイメージを作成するための Dockerfile を管理します。
プロジェクトのビルドに必要なツールやライブラリは
すべて Dockerfile 内でインストールするように記述します。
ディレクトリ名は docker-compose のサービス名に一致するように定義します。
すなわち `dockerfiles/develop` というディレクトリがあれば
`docker-compose.yml` 内で

```yaml
services:
  develop:
    build:
      context: dockerfiles/develop
```

のように参照します。

#### libs

プロジェクトに依存する内製ライブラリを管理します。
オープンソースで公開できないライブラリなど
管理の仕方が特殊なものをここで管理します。
オープンソースのライブラリはここでは管理しません。
代わりに Dockerfile で管理して下さい。

#### src

プロジェクトのソースコードを管理します。
プロジェクト名が `example` であれば `example` ディレクトリ配下が
そのプロジェクトが提供する成果物を作成するためのソースコードです。
`exampleTest` は単体テストのソースコードを管理します。

もしスタンドアローンなアプリケーションを作成する場合は
`main` ディレクトリに `main()` 関数を定義して下さい（`example` ではありません）。
つまり `example` のソースコードは必ずライブラリとしてコンパイルします。
そうすることで `exampleTest` をビルドする際に `example` の関数シンボルが
`exampleTest` にリンクできるようになります。

#### .clang-format

コーディングスタイルを整形するための clang-format 用の設定ファイルです。

#### CMakeLists.txt

ソースコードのビルド設定を記述します。
必ずしも CMake を使う必要はありません。
重要なのは CMake の実行を Makefile から行うことです。
そうすることでビルドの実行者は常に Make の使い方だけを知っていれば良くなります。
また CMake から別のビルドシステムへの置き換えが行いやすくなります。

#### docker-compose.yml

Docker コンテナを生成するための YAML ファイルです。
docker-compose の実行時に使用します。

#### Jenkinsfile

Jenkins パイプラインを定義するためのファイルです。

#### Makefile

ビルドのトリガとして使用する Makefile です。
C++ Boilerplate ではあらゆるタスクが Makefile を起点として実行する必要があります。
Makefile では Maven のようなライフサイクルが下記の通り用意されています。
ただし新たなライフサイクルを自由に追加することもできます。

| ライフサイクル | 説明                     |
| -------------- | ------------------------ |
| up             | コンテナを作成           |
| down           | コンテナを削除           |
| shell          | コンテナに入る           |
| init           | 成果物ディレクトリを作成 |
| remove         | 成果物ディレクトリを削除 |
| cmake          | CMake を実行             |
| build          | コンパイルを実行         |
| test           | 単体テストを実行         |
| clean          | 成果物を削除             |

ビルドをしたいときは次のように実行します。

```
$ make up
$ make shell
# make build CONFIG=[Debug|Release]
# exit
$ make down
```

成果物は `out/Debug, out/Release` に保存されます。

ビルドをするためにわざわざ up/down するのは一見面倒に思えます。
`make build` を実行したら自動的にコンテナを作成して
コンテナ内でビルドをすればいいじゃないかと思う方もいるでしょう。
そうしない理由はコンテナ上ではなくホスト上でビルドしたいという要望にも
応えられるようにするためです。
そのような要望はデバッグ時には十分あり得ることです。

#### version.h.in

プロジェクトのバージョンを管理するためのヘッダテンプレートです。
`make build` をすると `out/Debug, out/Release` 配下に `version.h` が作成されます。
