## pyenv, Poetry を使ったインストール
```
mkdir django-tutorial
cd django-tutorial
pyenv local 3.7.0

poetry init
poetry add django
poetry install
poetry shell

python -m django --version
# >> 3.0.8
```

## プロジェクト作成 & 開発用サーバの動作確認
```
# プロジェクトの作成 : プロジェクトとは、データベースの設定や Django 固有のオプション、アプリケーション固有の設定などといった、個々の Django インスタンスの設定を集めたもの
django-admin startproject mysite

# $tree mysite
# mysite # プロジェクトのコンテナ
# ├── manage.py # プロジェクトに対する様々な操作を行うためのコマンドラインユーティリティ
# └── mysite # このプロジェクトの実際の Python パッケージ この名前が Python パッケージの名前であり、 import の際に 使用する名前 (例えば import mysite.urls)
#    ├── __init__.py # このディレクトリが Python パッケージであることを Python に知らせるための空のファイル
#    ├── asgi.py # プロジェクトを提供する ASGI 互換 Web サーバーのエントリポイント
#    ├── settings.py # プロジェクトの設定ファイル
#    ├── urls.py # プロジェクトの URL 宣言、いうなれば Django サイトにおける「目次」に相当
#    └── wsgi.py # プロジェクトをサーブするためのWSGI互換Webサーバーとのエントリーポイント

# ポートを指定して開発サーバを起動 (デフォルトは 8000)
python manage.py runserver 8000
```

## Polls アプリケーションを作る
アプリは Python パス上のどこにでも置くことができる. 今回は manage.py と同じディレクトリ内に polls を作成する。
```
python manage.py startapp polls
# polls/
#     __init__.py
#     admin.py
#     apps.py
#     migrations/
#         __init__.py
#     models.py
#     tests.py
#     views.py
```

## Database の設定

アプリケーションの内、よく使われる機能はデフォルトで mysite/setting.py の INSTALLED_APPS に書かれている。これらは最低1つデータベースのテーブルを使うので、テーブルを作る必要がある。

- django.contrib.admin - 管理（admin）サイト。まもなく使います
- django.contrib.auth - 認証システム
- django.contrib.contenttypes - コンテンツタイプフレームワーク
- django.contrib.sessions - セッションフレームワーク
- django.contrib.messages - メッセージフレームワーク
- django.contrib.staticfiles - 静的ファイルの管理フレームワーク

```
# INSTALLED_APPS の設定を参照するとともに、 mysite/settings.py ファイルのデータベース設定に従って必要なデータベースのテーブルを作成する
python manage.py migrate
```

## モデルの作成

polls/models.py を編集してモデルを定義する。データベースのレイアウトとそれに付随するメタデータがモデルの本質である。
コード中のクラスが models.Model を継承する。
- 個々のクラス変数 `CharField, IntegerField` などは Field クラスのインスタンスとして表現されている。


```polls/models.py
from django.db import models


class Question(models.Model):
    question_text = models.CharField(max_length=200)
    pub_date = models.DateTimeField('date published')


class Choice(models.Model):
    question = models.ForeignKey(Question, on_delete=models.CASCADE)
    choice_text = models.CharField(max_length=200)
    votes = models.IntegerField(default=0)
```

- 以上のコードを追加したあと、`mysite/mysite/urls.py` の `INSTALLED_APPS` に次のパスを追加することで、polls アプリケーションが追加されたことを Django に教える。

```mysite/settings.py¶
INSTALLED_APPS = [
    'polls.apps.PollsConfig',
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
]
```

- 次に以下のコマンドを実行することで、Django にモデルに変更があったことを伝え、それをマイグレーションの形で保存させることができる。
```
python manage.py makemigrations polls
```

- マイグレーションがどんな SQL を実行するか見てみる。
```
python manage.py sqlmigrate polls 0001

# 次のような結果が返される (PostgreSQL を使用している場合)

BEGIN;
--
-- Create model Question
--
CREATE TABLE "polls_question" (
    "id" serial NOT NULL PRIMARY KEY,
    "question_text" varchar(200) NOT NULL,
    "pub_date" timestamp with time zone NOT NULL
);
--
-- Create model Choice
--
CREATE TABLE "polls_choice" (
    "id" serial NOT NULL PRIMARY KEY,
    "choice_text" varchar(200) NOT NULL,
    "votes" integer NOT NULL,
    "question_id" integer NOT NULL
);
ALTER TABLE "polls_choice"
  ADD CONSTRAINT "polls_choice_question_id_c5b4b260_fk_polls_question_id"
    FOREIGN KEY ("question_id")
    REFERENCES "polls_question" ("id")
    DEFERRABLE INITIALLY DEFERRED;
CREATE INDEX "polls_choice_question_id_c5b4b260" ON "polls_choice" ("question_id");

COMMIT;
```
