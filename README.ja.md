Windows モニタリングテンプレート
===============================

Windows モニタリング
-------------------

Windows サーバのリソース監視をします。

* OS標準のパフォーマンス情報採取コマンドを用いて、CPU、メモリ、ディスク、ネットワークリソースのグラフ化をします。
* Zabbix Windows 標準テンプレートを使用して、イベント監視をします(オプション)。
* Windows 2003/2008/20012 Server 32bit/64bit をサポートします。事前に各プラットフォームでエージェントコンパイルが必要です。

**[注意]**

* Windows モニタリングテンプレートは基本モジュールにバンドルされています。テンプレートを個別にセットアップする必要はありません。

ファイル構成
------------

テンプレートのファイル構成は以下の通りです。

|            ディレクトリ           |        ファイル名        |                  用途                 |
|-----------------------------------|--------------------------|---------------------------------------|
| lib/agent/Windows/conf/           | iniファイル              | エージェント採取設定ファイル          |
| lib/agent/Windows/script/         | 採取スクリプト           | エージェント採取スクリプト            |
| lib/Getperf/Command/Site/Windows/ | pmファイル               | データ集計スクリプト                  |
| lib/graph/Windows/                | jsonファイル             | Cactiグラフテンプレート登録ルール     |
| lib/zabbix/Windows.json           | jsonファイル             | Zabbix監視テンプレート登録ルール      |
| lib/cacti/template/0.8.8g/        | xmlファイル              | Cactiテンプレートエクスポートファイル |
| script/                           | create_graph_template.sh | グラフテンプレート登録スクリプト      |

メトリック
-----------

パフォーマンス統計グラフなどの監視項目定義は以下の通りです。

|          Key           |                                        Description                                         |
|------------------------|--------------------------------------------------------------------------------------------|
| **パフォーマンス統計** | **サーバリソースグラフ**                                                                   |
| ProcessorTotal         | **CPU**<br> CPU Util%                                                                      |
| Memory                 | **メモリ** <br> Memory Usage                                                               |
| PhysicalDisk           | **ディスクI/O統計** <br> ディスクビジー率 / ディスクIOPS / ディスク転送量 / ディスクIO遅延 |
| Network                | **ネットワークI/O統計**<br> ネットワークパケット数 / ネットワーク転送量 / 通信エラー       |
| LogicalDisk            | **ディスク容量**<br> ディスク使用率                                                        |
| **イベント**           | **Zabbix 監視**                                                                            |
| Windows                | Zabbix 標準テンプレートのWindowsテンプレートによる監視                                     |

Install
=======

テンプレートのビルド
--------------------

Git Hub からプロジェクトをクローンします。

```
(git clone してプロジェクト複製)
```

プロジェクトディレクトリに移動して、--template オプション付きでサイトの初期化をします。

```
cd t_Windows
initsite --template .
```

Cacti グラフテンプレート作成スクリプトを順に実行します。

```
./script/create_graph_template__Windows.sh
```

Cacti グラフテンプレートをファイルにエクスポートします。

```
cacti-cli --export Windows
```

集計スクリプト、グラフ登録ルール、Cactiグラフテンプレートエクスポートファイル一式をアーカイブします。

```
mkdir -p $GETPERF_HOME/var/template/archive/
sumup --export=Windows --archive=$GETPERF_HOME/var/template/archive/config-Windows.tar.gz
```

テンプレートのインポート
------------------------

監視サイトに前述で作成したアーカイブファイルを解凍します。

```
cd {モニタリングサイトホーム}
tar xvf $GETPERF_HOME/var/template/archive/config-Windows.tar.gz
```

Cacti グラフテンプレートをインポートします。

```
cacti-cli --import Windows
```

インポートした集計スクリプトを反映するため、集計デーモンを再起動します。

```
sumup restart
```

エージェントセットアップ
========================

データ採取スクリプトの配布
--------------------

Windowsデータ採取ライブラリ一式を、監視対象のWindowsサーバに配布します。監視サイトのlib の下にある以下のディレクトリ下のファイル一式を監視対象の エージェントホームディレクトリ(C:\ptune)にコピーします。

```
ls lib/agent/Windows/
conf  script
```

エージェントの起動
------------------

設定を反映させるため、エージェントを再起動します。

```
c:\ptune\bin\getperfctl stop
c:\ptune\bin\getperfctl start
```

Cacti グラフ登録
================

上記エージェントセットアップ後、データ集計が実行されると、サイトホームディレクトリの node の下にノード定義ファイルが出力されます。
出力されたノード定義ディレクトリを指定して cacti-cli を実行します。

```
cd {サイトホーム}
cacti-cli node/Windows/{Windowsホスト名}/
```

Zabbix 監視登録
===============

.hosts への監視対象サーバIPの登録
----------------------------

DNSなどが設定されておらず、監視対象サーバホスト名からIPアドレスの参照ができない場合は、.hosts ファイルに IP アドレスとホスト名の設定をします。

```
cd {サイトホーム}
vi .hosts
```

"IPアドレス ホスト名" の形式で IP　アドレスを登録してください。

Zabbix の監視設定
--------------------

zabbix-cli コマンドの、 --info オプションで設定内容の確認をしてから、--add オプションで登録します。

```
# 設定内容の確認
zabbix-cli --info node/Windows/{Windowsホスト名}/
# 問題がなければ登録
zabbix-cli --add node/Windows/{Windowsホスト名}/
```

AUTHOR
-----------

Minoru Furusawa <minoru.furusawa@toshiba.co.jp>

COPYRIGHT
-----------

Copyright 2014-2016, Minoru Furusawa, Toshiba corporation.

LICENSE
-----------

This program is released under [GNU General Public License, version 2](http://www.gnu.org/licenses/gpl-2.0.html).
