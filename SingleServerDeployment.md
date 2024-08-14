# Quickstart (local)

動機：Druid の利点はクエリの同時実行によるパフォーマンスが他の OLAP より優れているとされていること。toC で分析基盤を提供する場合には有利そう。

https://druid.apache.org/docs/latest/tutorials/

このクイックスタートでは、次のことを行います。

- Druid をインストールする
- ドルイドサービスを開始する
- SQL を使用してデータを取り込み、クエリする

Java のインストールがまだの場合はインストールします

```
echo $JAVA_HOME

sudo apt install openjdk-17-jdk

JAVA_PATH=$(readlink -f $(which java))
JAVA_HOME=${JAVA_PATH%/bin/java}
echo "export JAVA_HOME=$JAVA_HOME" >> ~/.bashrc
echo "export PATH=\$PATH:\$JAVA_HOME/bin" >> ~/.bashrc
source ~/.bashrc
```

### Install Druid

```
curl -O https://dlcdn.apache.org/druid/30.0.0/apache-druid-30.0.0-bin.tar.gz
cd apache-druid-30.0.0
```

### Start up Druid services

```
masami@masami-L:/usr/src/apache-druid-30.0.0$ sudo ./bin/start-micro-quickstart
[Wed Jul 31 22:31:29 2024] Starting Apache Druid.
[Wed Jul 31 22:31:29 2024] Open http://localhost:8888/ or http://masami-L:8888/ in your browser to access the web console.
[Wed Jul 31 22:31:29 2024] Or, if you have enabled TLS, use https on port 9088.
[Wed Jul 31 22:31:29 2024] Starting services with log directory [/usr/src/apache-druid-30.0.0/log].
[Wed Jul 31 22:31:29 2024] Running command[zk]: bin/run-zk conf
[Wed Jul 31 22:31:29 2024] Running command[coordinator-overlord]: bin/run-druid coordinator-overlord conf/druid/single-server/micro-quickstart
[Wed Jul 31 22:31:29 2024] Running command[broker]: bin/run-druid broker conf/druid/single-server/micro-quickstart
[Wed Jul 31 22:31:29 2024] Running command[middleManager]: bin/run-druid middleManager conf/druid/single-server/micro-quickstart
[Wed Jul 31 22:31:29 2024] Running command[historical]: bin/run-druid historical conf/druid/single-server/micro-quickstart
[Wed Jul 31 22:31:29 2024] Running command[router]: bin/run-druid router conf/druid/single-server/micro-quickstart
```

URL にアクセスすると次の管理ページにアクセスできます

![1](https://github.com/user-attachments/assets/342da6de-2a37-4d89-ad97-02e2b50163df)

### Load data

画面上から Query の外部データへの接続を選択し、ベースディレクトリのファイル名を指定し、データに接続します。

![2](https://github.com/user-attachments/assets/07713f66-834e-481f-987d-ff4dc4f01788)

取り込み完了後、テーブルにサンプル データを挿入します

```sql
REPLACE INTO "wikipedia" OVERWRITE ALL
WITH "ext" AS (
  SELECT *
  FROM TABLE(
    EXTERN(
      '{"type":"local","baseDir":"quickstart/tutorial/","filter":"wikiticker-2015-09-12-sampled.json.gz"}',
      '{"type":"json"}'
    )
  ) EXTEND ("time" VARCHAR, "channel" VARCHAR, "cityName" VARCHAR, "comment" VARCHAR, "countryIsoCode" VARCHAR, "countryName" VARCHAR, "isAnonymous" VARCHAR, "isMinor" VARCHAR, "isNew" VARCHAR, "isRobot" VARCHAR, "isUnpatrolled" VARCHAR, "metroCode" BIGINT, "namespace" VARCHAR, "page" VARCHAR, "regionIsoCode" VARCHAR, "regionName" VARCHAR, "user" VARCHAR, "delta" BIGINT, "added" BIGINT, "deleted" BIGINT)
)
SELECT
  TIME_PARSE("time") AS "__time",
  "channel",
  "cityName",
  "comment",
  "countryIsoCode",
  "countryName",
  "isAnonymous",
  "isMinor",
  "isNew",
  "isRobot",
  "isUnpatrolled",
  "metroCode",
  "namespace",
  "page",
  "regionIsoCode",
  "regionName",
  "user",
  "delta",
  "added",
  "deleted"
FROM "ext"
PARTITIONED BY DAY
```

### クエリ

取り込み後はクエリが実行できます

クエリビューで次のクエリを実行して、上位のチャネルのリストを生成します

```Sql
SELECT
  channel,
  COUNT(*)
FROM "wikipedia"
GROUP BY channel
ORDER BY COUNT(*) DESC
```

![3](https://github.com/user-attachments/assets/b7265e47-bdca-4f23-a344-e65e90c9ce43)
