# 取り込み仕様を記述する

ファイルを作成

```
cd apache-druid-30.0.0/
sudo vi quickstart/tutorial/ingestion-tutorial-data.json
```

```json
{"ts":"2018-01-01T01:01:35Z","srcIP":"1.1.1.1", "dstIP":"2.2.2.2", "srcPort":2000, "dstPort":3000, "protocol": 6, "packets":10, "bytes":1000, "cost": 1.4}
{"ts":"2018-01-01T01:01:51Z","srcIP":"1.1.1.1", "dstIP":"2.2.2.2", "srcPort":2000, "dstPort":3000, "protocol": 6, "packets":20, "bytes":2000, "cost": 3.1}
{"ts":"2018-01-01T01:01:59Z","srcIP":"1.1.1.1", "dstIP":"2.2.2.2", "srcPort":2000, "dstPort":3000, "protocol": 6, "packets":30, "bytes":3000, "cost": 0.4}
{"ts":"2018-01-01T01:02:14Z","srcIP":"1.1.1.1", "dstIP":"2.2.2.2", "srcPort":5000, "dstPort":7000, "protocol": 6, "packets":40, "bytes":4000, "cost": 7.9}
{"ts":"2018-01-01T01:02:29Z","srcIP":"1.1.1.1", "dstIP":"2.2.2.2", "srcPort":5000, "dstPort":7000, "protocol": 6, "packets":50, "bytes":5000, "cost": 10.2}
{"ts":"2018-01-01T01:03:29Z","srcIP":"1.1.1.1", "dstIP":"2.2.2.2", "srcPort":5000, "dstPort":7000, "protocol": 6, "packets":60, "bytes":6000, "cost": 4.3}
{"ts":"2018-01-01T02:33:14Z","srcIP":"7.7.7.7", "dstIP":"8.8.8.8", "srcPort":4000, "dstPort":5000, "protocol": 17, "packets":100, "bytes":10000, "cost": 22.4}
{"ts":"2018-01-01T02:33:45Z","srcIP":"7.7.7.7", "dstIP":"8.8.8.8", "srcPort":4000, "dstPort":5000, "protocol": 17, "packets":200, "bytes":20000, "cost": 34.5}
{"ts":"2018-01-01T02:35:45Z","srcIP":"7.7.7.7", "dstIP":"8.8.8.8", "srcPort":4000, "dstPort":5000, "protocol": 17, "packets":300, "bytes":30000, "cost": 46.3}
```

```
sudo ./bin/start-druid
[sudo] masami のパスワード:
[Mon Aug  5 21:19:37 2024] Starting Apache Druid.
[Mon Aug  5 21:19:37 2024] Open http://localhost:8888/ or http://masami-L:8888/ in your browser to access the web console.
[Mon Aug  5 21:19:37 2024] Or, if you have enabled TLS, use https on port 9088.
[Mon Aug  5 21:19:37 2024] Starting services with log directory [/usr/src/apache-druid-30.0.0/log].
[Mon Aug  5 21:19:37 2024] Running command[zk]: bin/run-zk conf
[Mon Aug  5 21:19:37 2024] Running command[broker]: bin/run-druid broker /usr/src/apache-druid-30.0.0/conf/druid/auto '-Xms874m -Xmx874m -XX:MaxDirectMemorySize=582m'
[Mon Aug  5 21:19:37 2024] Running command[router]: bin/run-druid router /usr/src/apache-druid-30.0.0/conf/druid/auto '-Xms256m -Xmx256m -XX:MaxDirectMemorySize=128m'
[Mon Aug  5 21:19:37 2024] Running command[middleManager]: bin/run-druid middleManager /usr/src/apache-druid-30.0.0/conf/druid/auto '-Xms64m -Xmx64m' '-Ddruid.worker.capacity=2 -Ddruid.indexer.runner.javaOptsArray=["-server","-Duser.timezone=UTC","-Dfile.encoding=UTF-8","-XX:+ExitOnOutOfMemoryError","-Djava.util.logging.manager=org.apache.logging.log4j.jul.LogManager","-Xms256m","-Xmx256m","-XX:MaxDirectMemorySize=256m"]'
[Mon Aug  5 21:19:37 2024] Running command[coordinator-overlord]: bin/run-druid coordinator-overlord /usr/src/apache-druid-30.0.0/conf/druid/auto '-Xms950m -Xmx950m'
[Mon Aug  5 21:19:37 2024] Running command[historical]: bin/run-druid historical /usr/src/apache-druid-30.0.0/conf/druid/auto '-Xms1013m -Xmx1013m -XX:MaxDirectMemorySize=1520m'
```

### Defining the schema

```
vi quickstart/tutorial/ingestion-tutorial-index.json
```

dataSource のパラメータを指定

```json
"dataSchema" : {
  "dataSource" : "ingestion-tutorial",
}
```

Time Column の指定

```json
"dataSchema" : {
  "dataSource" : "ingestion-tutorial",
  "timestampSpec" : {
    "format" : "iso",
    "column" : "ts"
  }
}
```

ロールアップが有効になっている場合は、入力列を「ディメンション」と「メトリック」の 2 つのカテゴリに分ける必要があります。「ディメンション」はロールアップのグループ化列であり、「メトリック」は集計される列です。

```json
"dataSchema" : {
  "dataSource" : "ingestion-tutorial",
  "timestampSpec" : {
    "format" : "iso",
    "column" : "ts"
  },
  "granularitySpec" : {
    "rollup" : true
  }
}
```

dimension と metric の設定

```
"dataSchema" : {
  "dataSource" : "ingestion-tutorial",
  "timestampSpec" : {
    "format" : "iso",
    "column" : "ts"
  },
  "dimensionsSpec" : {
    "dimensions": [
      "srcIP",
      { "name" : "srcPort", "type" : "long" },
      { "name" : "dstIP", "type" : "string" },
      { "name" : "dstPort", "type" : "long" },
      { "name" : "protocol", "type" : "string" }
    ]
  },
  "granularitySpec" : {
    "rollup" : true
  }
}
```

メトリックは dataSchema 内に metricsSpec を指定します

```json
"dataSchema" : {
  "dataSource" : "ingestion-tutorial",
  "timestampSpec" : {
    "format" : "iso",
    "column" : "ts"
  },
  "dimensionsSpec" : {
    "dimensions": [
      "srcIP",
      { "name" : "srcPort", "type" : "long" },
      { "name" : "dstIP", "type" : "string" },
      { "name" : "dstPort", "type" : "long" },
      { "name" : "protocol", "type" : "string" }
    ]
  },
  "metricsSpec" : [
    { "type" : "count", "name" : "count" },
    { "type" : "longSum", "name" : "packets", "fieldName" : "packets" },
    { "type" : "longSum", "name" : "bytes", "fieldName" : "bytes" },
    { "type" : "doubleSum", "name" : "cost", "fieldName" : "cost" }
  ],
  "granularitySpec" : {
    "rollup" : true
  }
}
```

セグメントの粒度は segmentGranularity によって設定されます

```json
"dataSchema" : {
  "dataSource" : "ingestion-tutorial",
  "timestampSpec" : {
    "format" : "iso",
    "column" : "ts"
  },
  "dimensionsSpec" : {
    "dimensions": [
      "srcIP",
      { "name" : "srcPort", "type" : "long" },
      { "name" : "dstIP", "type" : "string" },
      { "name" : "dstPort", "type" : "long" },
      { "name" : "protocol", "type" : "string" }
    ]
  },
  "metricsSpec" : [
    { "type" : "count", "name" : "count" },
    { "type" : "longSum", "name" : "packets", "fieldName" : "packets" },
    { "type" : "longSum", "name" : "bytes", "fieldName" : "bytes" },
    { "type" : "doubleSum", "name" : "cost", "fieldName" : "cost" }
  ],
  "granularitySpec" : {
    "type" : "uniform",
    "segmentGranularity" : "HOUR",
    "rollup" : true
  }
}
```

クエリの粒度は、queryGranularity のプロパティによって設定されます

```json
"dataSchema" : {
  "dataSource" : "ingestion-tutorial",
  "timestampSpec" : {
    "format" : "iso",
    "column" : "ts"
  },
  "dimensionsSpec" : {
    "dimensions": [
      "srcIP",
      { "name" : "srcPort", "type" : "long" },
      { "name" : "dstIP", "type" : "string" },
      { "name" : "dstPort", "type" : "long" },
      { "name" : "protocol", "type" : "string" }
    ]
  },
  "metricsSpec" : [
    { "type" : "count", "name" : "count" },
    { "type" : "longSum", "name" : "packets", "fieldName" : "packets" },
    { "type" : "longSum", "name" : "bytes", "fieldName" : "bytes" },
    { "type" : "doubleSum", "name" : "cost", "fieldName" : "cost" }
  ],
  "granularitySpec" : {
    "type" : "uniform",
    "segmentGranularity" : "HOUR",
    "queryGranularity" : "MINUTE",
    "rollup" : true
  }
}
```

interval を定義します(バッチのみ)

```json
"dataSchema" : {
  "dataSource" : "ingestion-tutorial",
  "timestampSpec" : {
    "format" : "iso",
    "column" : "ts"
  },
  "dimensionsSpec" : {
    "dimensions": [
      "srcIP",
      { "name" : "srcPort", "type" : "long" },
      { "name" : "dstIP", "type" : "string" },
      { "name" : "dstPort", "type" : "long" },
      { "name" : "protocol", "type" : "string" }
    ]
  },
  "metricsSpec" : [
    { "type" : "count", "name" : "count" },
    { "type" : "longSum", "name" : "packets", "fieldName" : "packets" },
    { "type" : "longSum", "name" : "bytes", "fieldName" : "bytes" },
    { "type" : "doubleSum", "name" : "cost", "fieldName" : "cost" }
  ],
  "granularitySpec" : {
    "type" : "uniform",
    "segmentGranularity" : "HOUR",
    "queryGranularity" : "MINUTE",
    "intervals" : ["2018-01-01/2018-01-02"],
    "rollup" : true
  }
}
```

タスクを定義します

```json
{
  "type": "index_parallel",
  "spec": {
    "dataSchema": {
      "dataSource": "ingestion-tutorial",
      "timestampSpec": {
        "format": "iso",
        "column": "ts"
      },
      "dimensionsSpec": {
        "dimensions": [
          "srcIP",
          { "name": "srcPort", "type": "long" },
          { "name": "dstIP", "type": "string" },
          { "name": "dstPort", "type": "long" },
          { "name": "protocol", "type": "string" }
        ]
      },
      "metricsSpec": [
        { "type": "count", "name": "count" },
        { "type": "longSum", "name": "packets", "fieldName": "packets" },
        { "type": "longSum", "name": "bytes", "fieldName": "bytes" },
        { "type": "doubleSum", "name": "cost", "fieldName": "cost" }
      ],
      "granularitySpec": {
        "type": "uniform",
        "segmentGranularity": "HOUR",
        "queryGranularity": "MINUTE",
        "intervals": ["2018-01-01/2018-01-02"],
        "rollup": true
      }
    }
  }
}
```

オブジェクトで指定される入力ソースを定義しましょう

```json
{
  "type": "index_parallel",
  "spec": {
    "dataSchema": {
      "dataSource": "ingestion-tutorial",
      "timestampSpec": {
        "format": "iso",
        "column": "ts"
      },
      "dimensionsSpec": {
        "dimensions": [
          "srcIP",
          { "name": "srcPort", "type": "long" },
          { "name": "dstIP", "type": "string" },
          { "name": "dstPort", "type": "long" },
          { "name": "protocol", "type": "string" }
        ]
      },
      "metricsSpec": [
        { "type": "count", "name": "count" },
        { "type": "longSum", "name": "packets", "fieldName": "packets" },
        { "type": "longSum", "name": "bytes", "fieldName": "bytes" },
        { "type": "doubleSum", "name": "cost", "fieldName": "cost" }
      ],
      "granularitySpec": {
        "type": "uniform",
        "segmentGranularity": "HOUR",
        "queryGranularity": "MINUTE",
        "intervals": ["2018-01-01/2018-01-02"],
        "rollup": true
      }
    },
    "ioConfig": {
      "type": "index_parallel",
      "inputSource": {
        "type": "local",
        "baseDir": "quickstart/",
        "filter": "ingestion-tutorial-data.json"
      },
      "inputFormat": {
        "type": "json"
      }
    }
  }
}
```

ユーザーがさまざまな取り込みパラメータを調整できる tuningConfig セクションがあります

```json
{
  "type": "index_parallel",
  "spec": {
    "dataSchema": {
      "dataSource": "ingestion-tutorial",
      "timestampSpec": {
        "format": "iso",
        "column": "ts"
      },
      "dimensionsSpec": {
        "dimensions": [
          "srcIP",
          { "name": "srcPort", "type": "long" },
          { "name": "dstIP", "type": "string" },
          { "name": "dstPort", "type": "long" },
          { "name": "protocol", "type": "string" }
        ]
      },
      "metricsSpec": [
        { "type": "count", "name": "count" },
        { "type": "longSum", "name": "packets", "fieldName": "packets" },
        { "type": "longSum", "name": "bytes", "fieldName": "bytes" },
        { "type": "doubleSum", "name": "cost", "fieldName": "cost" }
      ],
      "granularitySpec": {
        "type": "uniform",
        "segmentGranularity": "HOUR",
        "queryGranularity": "MINUTE",
        "intervals": ["2018-01-01/2018-01-02"],
        "rollup": true
      }
    },
    "ioConfig": {
      "type": "index_parallel",
      "inputSource": {
        "type": "local",
        "baseDir": "quickstart/",
        "filter": "ingestion-tutorial-data.json"
      },
      "inputFormat": {
        "type": "json"
      }
    },
    "tuningConfig": {
      "type": "index_parallel",
      "partitionsSpec": {
        "type": "dynamic",
        "maxRowsPerSegment": 5000000
      }
    }
  }
}
```

```
sudo bin/post-index-task --file quickstart/tutorial/ingestion-tutorial-ind
ex.json --url http://localhost:8081
Beginning indexing data for ingestion-tutorial
Task started: index_parallel_ingestion-tutorial_mpmhegoc_2024-08-05T13:09:29.399Z
Task log:     http://localhost:8081/druid/indexer/v1/task/index_parallel_ingestion-tutorial_mpmhegoc_2024-08-05T13:09:29.399Z/log
Task status:  http://localhost:8081/druid/indexer/v1/task/index_parallel_ingestion-tutorial_mpmhegoc_2024-08-05T13:09:29.399Z/status
Task index_parallel_ingestion-tutorial_mpmhegoc_2024-08-05T13:09:29.399Z still running...
Task index_parallel_ingestion-tutorial_mpmhegoc_2024-08-05T13:09:29.399Z still running...
Task finished with status: SUCCESS
Completed indexing data for ingestion-tutorial. Now loading indexed data onto the cluster...
ingestion-tutorial is 0.0% finished loading...
ingestion-tutorial is 0.0% finished loading...
ingestion-tutorial loading complete! You may now query your data
```

SQL の実行

```Sql
masami@masami-L /u/s/apache-druid-30.0.0> sudo bin/dsql
Welcome to dsql, the command-line client for Druid SQL.
Connected to [http://localhost:8082/].

Type "\h" for help.
dsql> select * from "ingestion-tutorial";
┌──────────────────────────┬─────────┬─────────┬─────────┬─────────┬──────────┬───────┬──────┬───────┬─────────┐
│ __time                   │ srcIP   │ srcPort │ dstIP   │ dstPort │ protocol │ bytes │ cost │ count │ packets │
├──────────────────────────┼─────────┼─────────┼─────────┼─────────┼──────────┼───────┼──────┼───────┼─────────┤
│ 2018-01-01T01:01:00.000Z │ 1.1.1.1 │    2000 │ 2.2.2.2 │    3000 │ 6        │  6000 │  4.9 │     3 │      60 │
│ 2018-01-01T01:02:00.000Z │ 1.1.1.1 │    5000 │ 2.2.2.2 │    7000 │ 6        │  9000 │ 18.1 │     2 │      90 │
│ 2018-01-01T01:03:00.000Z │ 1.1.1.1 │    5000 │ 2.2.2.2 │    7000 │ 6        │  6000 │  4.3 │     1 │      60 │
│ 2018-01-01T02:33:00.000Z │ 7.7.7.7 │    4000 │ 8.8.8.8 │    5000 │ 17       │ 30000 │ 56.9 │     2 │     300 │
│ 2018-01-01T02:35:00.000Z │ 7.7.7.7 │    4000 │ 8.8.8.8 │    5000 │ 17       │ 30000 │ 46.3 │     1 │     300 │
└──────────────────────────┴─────────┴─────────┴─────────┴─────────┴──────────┴───────┴──────┴───────┴─────────┘
Retrieved 5 rows in 0.16s.
```
