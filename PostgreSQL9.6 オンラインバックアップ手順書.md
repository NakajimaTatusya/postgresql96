# PostgreSQL9.6 オンラインバックアップ手順書

* 本ドキュメントの作業を行う前提条件
  * サーバーOSは、Windows Server 2016,2019を想定しています。
  * [推奨]本番 PostgreSQL サーバーと同等のスペックのサーバーをご用意ください。
  * [必須]検証 PostgreSQL サーバーへ、本番 PostgreSQL サーバーと同じバージョンの PostgreSQL をインストールしておいてください。
  * [必須]本番 PostgreSQL サーバーと検証 PostgreSQL サーバーは同じネットワークに所属し、接続可能である事をご確認ください。

* 本ドキュメントで使用している用語の説明

| 用語 | 説明 |
| :--- | :--- |
| 本番 PostgreSQL サーバー | 現在運用されている PostgresSQL9.6 データベースサーバー |
| 検証 PostgreSQL サーバー | 新規作成する PostgreSQL9.6 データベースサーバー。バックアップ復元先。 |
| pg_basebackup | PostgreSQL ビルトインコマンド。データベースファイルのバックアップを行う。 |
| pg_dump | PostgreSQL ビルトインコマンド。論理バックアップを行う。 |

## 本番 PostgreSQL サーバー の設定

* [必須]管理者ユーザーのユーザー名とパスワードを用意してください。

### [任意]レプリケーションユーザー作成

```sql
-- [任意] リモートバックアップを行わない場合は不要です。
-- パフォーマンス計測のために、REPLICATION データベースへ接続するためのユーザーを作成
-- 特殊なユーザー、普段は使いません。パスワードも設定しません。
CREATE USER perftest WITH REPLICATION
```

### postgresql.conf, pg_hba.conf の確認

* 下記の設定(postgresql.conf,pg_hba.conf)が行われていることが必要です。
* 設定がない場合は、conf ファイルを編集し、PostgreSQL Service を !!! 再起動 !!! してください。

* postgresql.conf

```ini
# [必須]WALの出力レベルを設定、規定値は minimal
wal_level = replica

# [必須]pg_basebackup を実行するために 1 以上にする。データベースをレプリケーションする場合は、standbyサーバー数に合わせて変更する
max_wal_senders = 1

# [任意] 設定されていなくても大丈夫です。
archive_mode = 'on'
archive_command = 'copy "%p" "C:\\tmp\\archivelog\\%f"'
```

* pg_hba.conf
* このファイルには、REPLICATIONデータベースという特殊なデータベースに接続するためのユーザを定義しています。

```ini
# TYPE  DATABASE        USER            ADDRESS                 METHOD

# IPv4 local connections:
host    all             all             127.0.0.1/32            md5
# IPv6 local connections:
host    all             all             ::1/128                 md5

# Allow replication connections from localhost, by a user with the replication privilege.
# 本番 PostgreSQL サーバーで実施する場合
host    replication     postgres        127.0.0.1/32            md5
host    replication     postgres        ::1/128                 md5
# [任意]検証 PostgreSQL サーバーからリモートで実施する場合
# trust とすることでパスワード無しで接続
host    replication     perftest        <検証 PostgreSQL サーバーのIP>/32          trust
```

## 本番 PostgreSQL サーバーバックアップ実行

* バックアップはオンラインバックアップです。本番サーバーを停止する必要はありません。
* バックアップ処理は低負荷ですが、安全のため業務が行われていない時間帯を設定することを推奨します。
* 念のため論理バックアップ（pg_dump）を取得してから行ってください。
* pg_basebackup の実行は「本番 PostgreSQL サーバー」で行う方法と、「検証 PostgreSQL サーバー」からリモートで行う方法があります。

### 「本番 PostgreSQL サーバー」で pg_basebackup を実施し、「検証 PostgreSQL サーバー」へレストアする

* バックアップ用のディレクトリを作成する

```cmd
REM 本番 PostgreSQL サーバーで実施する

REM YYYYMMDD 実行日
cmd> mkdir C:\tmp\PostgreSQL96\backup\YYYYMMDD
```

* pg_basebackup を実行する

```cmd
REM 本番 PostgreSQL サーバーで実施する

REM PostgresSQL コマンドのパスが通っていない場合は、下記へカレントディレクトリを変更します。
cmd> cd C:\Program Files\PostgreSQL\9.6\bin

REM オンラインバックアップ実行
cmd> pg_basebackup -R -h localhost -p 5432 -U postgres -D "C:\tmp\PostgreSQL96\backup\YYYYMMDD" --xlog --checkpoint=fast --progress
```

* 検証 PostgreSQL サーバーの PostgreSQL サービスを停止する

```cmd
REM 検証 PostgreSQL サーバーで実施する

REM サービス名は [コントロール パネル] > [システムとセキュリティ] > [管理ツール] > [サービス] から確認してください。
REM サービス名は御社の環境を確認してください。
net stop postgresql-x64-9.6
```

* 検証 PostgreSQL サーバーへバックアップファイルを全てコピーする
  * 「C:\tmp\PostgreSQL96\backup\YYYYMMDD」 を、「検証 PostgreSQL サーバー」の「C:\tmp\PostgreSQL96\backup\」へコピーする

* 検証 PostgreSQL サーバーの「C:\tmp\PostgreSQL96\backup\YYYYMMDD」配下の設定ファイルを編集する
* postgresql.conf ファイルを編集する

```ini
# postgresql.conf
hot_standby = 'off'
```

* recovery.conf ファイルを編集する

```ini
# recovery.conf
standby_mode = 'off'
primary_conninfo = 'host=localhost port=5432 user=postgres password=<password>'
recovery_target_timeline='latest'
restore_command = 'copy "C:\\tmp\\archivelog\\%f" "%p"'
```

* ダミーアーカイブフォルダ作成

```cmd
REM 検証 PostgreSQL サーバーで実施する

REM レストア用のダミーディレクトリを作成する(アーカイブモードが'no'の場合 restore_command の指定をダミーフォルダにする)
REM restore_command パラメータに関係します。

cmd> mkdir C:\tmp\archivelog

REM ※archive_mode = 'on' で運用されている場合は、archive_command で退避しているWALアーカイブを
REM 　上記のフォルダへコピーしておく。
```

* 検証 PostgreSQL サーバーの「C:\tmp\PostgreSQL96\backup\YYYYMMDD\」は以下のファイル／フォルダを全てコピーする。

```cmd
cmd> cd C:\Program Files\PostgreSQL\9.6
cmd> xcopy C:\tmp\PostgreSQL96\backup\YYYYMMDD data /E /Y /I /D
```

* サービスを起動する

```cmd
net start postgresql-x64-9.6
```

* 動作を確認する
  * [スタートメニュー] > [PostgreSQL9.6] > [SQL Shell (psql)] などを使用して、SELECT文を実行してみる

### 2回目以降のレストアについて

* 性能検証テスト実施後に、検証 PostgreSQL サーバーを初期状態にもどすには、以下の手順を実施します。

* 検証 PostgreSQL サーバーの PostgreSQL サービスを停止する

```cmd
REM 検証 PostgreSQL サーバーで実施する

REM サービス名は [コントロール パネル] > [システムとセキュリティ] > [管理ツール] > [サービス] から確認してください。
REM サービス名は御社の環境を確認してください。
net stop postgresql-x64-9.6
```

* 検証 PostgreSQL サーバーの PostgreSQL9.6 インストール既定のデータベースフォルダ以下ファイル／フォルダを全て削除

```cmd
REM PostgreSQL9.6 インストール既定のデータベースフォルダ以下ファイル／フォルダを全て削除
cmd> del /q "C:\Program Files\PostgreSQL\9.6\data\*"
cmd> for /d %p in ("C:\Program Files\PostgreSQL\9.6\data\*.*") do rmdir "%p" /s /q
```

* 検証 PostgreSQL サーバーの「C:\tmp\PostgreSQL96\backup\YYYYMMDD\」は以下のファイル／フォルダを全てコピーする。

```cmd
cmd> cd C:\Program Files\PostgreSQL\9.6
cmd> xcopy C:\tmp\PostgreSQL96\backup\YYYYMMDD data /E /Y /I /D
```

* サービスを起動する

```cmd
net start postgresql-x64-9.6
```

* 動作を確認する
  * [スタートメニュー] > [PostgreSQL9.6] > [SQL Shell (psql)] などを使用して、SELECT文を実行してみる

### [任意]検証 PostgreSQL サーバーからリモートで実施する

* PostgresSQL サービスを停止する

```cmd
REM サービス名は [コントロール パネル] > [システムとセキュリティ] > [管理ツール] > [サービス] から確認してください。
REM サービス名は御社の環境を確認してください。
net stop postgresql-x64-9.6
```

* バックアップ実行
  * 新規作成したデータベースになりますので、「C:\Program Files\PostgreSQL\9.6\data」フォルダ以下のファイル/フォルダすべて削除してください。

```cmd
REM PostgreSQL9.6 インストール既定のデータベースフォルダ以下ファイル／フォルダを全て削除
cmd> del /q "C:\Program Files\PostgreSQL\9.6\data\*"
cmd> for /d %p in ("C:\Program Files\PostgreSQL\9.6\data\*.*") do rmdir "%p" /s /q
REM PostgreSQL9.6 インストール既定のデータベースフォルダにバックアップ
cmd> pg_basebackup -R -h <本番 PostgreSQL サーバーのIP> -p 5432 -U perftest -D "C:\Program Files\PostgreSQL\9.6\data" --xlog --checkpoint=fast --progress
REM レストア用のダミーディレクトリを作成する(アーカイブモードが'off'の場合 restore_command の指定をダミーフォルダにする)
cmd> mkdir C:\tmp\archivelog
```

* 設定ファイルの編集

  * postgresql.conf ファイルを編集する
  * recovery.conf ファイルを編集する

```ini
# postgresql.conf
hot_standby = 'off'

# recovery.conf
standby_mode = 'off'
primary_conninfo = 'host=localhost port=5432 user=postgres password=<password>'
recovery_target_timeline='latest'
restore_command = 'copy "C:\\tmp\\archivelog\\%f" "%p"'
```

* サービスを起動する

```cmd
net start postgresql-x64-9.6
```

* 動作を確認する
  * [スタートメニュー] > [PostgreSQL9.6] > [SQL Shell (psql)] などを使用して、SELECT文を実行してみる
