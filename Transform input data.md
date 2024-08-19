# 入力データを変換する

https://druid.apache.org/docs/latest/tutorials/tutorial-transform-spec

データは最初から single machine クイックスタートでインストールされています

```
cat quickstart/tutorial/transform-data.json
{"timestamp":"2018-01-01T07:01:35Z","animal":"octopus",  "location":1, "number":100}
{"timestamp":"2018-01-01T05:01:35Z","animal":"mongoose", "location":2,"number":200}
{"timestamp":"2018-01-01T06:01:35Z","animal":"snake", "location":3, "number":300}
{"timestamp":"2018-01-01T01:01:35Z","animal":"lion", "location":4, "number":300}
```

変換定義は`transformSpec`で記述します

```json
{
  "type": "index_parallel",
  "spec": {
    "dataSchema": {
      "dataSource": "transform-tutorial",
      "timestampSpec": {
        "column": "timestamp",
        "format": "iso"
      },
      "dimensionsSpec": {
        "dimensions": ["animal", { "name": "location", "type": "long" }]
      },
      "metricsSpec": [
        { "type": "count", "name": "count" },
        { "type": "longSum", "name": "number", "fieldName": "number" },
        {
          "type": "longSum",
          "name": "triple-number",
          "fieldName": "triple-number"
        }
      ],
      "granularitySpec": {
        "type": "uniform",
        "segmentGranularity": "week",
        "queryGranularity": "minute",
        "intervals": ["2018-01-01/2018-01-03"],
        "rollup": true
      },
      "transformSpec": {
        "transforms": [
          {
            "type": "expression",
            "name": "animal",
            "expression": "concat('super-', animal)"
          },
          {
            "type": "expression",
            "name": "triple-number",
            "expression": "number * 3"
          }
        ],
        "filter": {
          "type": "or",
          "fields": [
            {
              "type": "selector",
              "dimension": "animal",
              "value": "super-mongoose"
            },
            {
              "type": "selector",
              "dimension": "triple-number",
              "value": "300"
            },
            { "type": "selector", "dimension": "location", "value": "3" }
          ]
        }
      }
    },
    "ioConfig": {
      "type": "index_parallel",
      "inputSource": {
        "type": "local",
        "baseDir": "quickstart/tutorial",
        "filter": "transform-data.json"
      },
      "inputFormat": {
        "type": "json"
      },
      "appendToExisting": false
    },
    "tuningConfig": {
      "type": "index_parallel",
      "partitionsSpec": {
        "type": "dynamic"
      },
      "maxRowsInMemory": 25000
    }
  }
}
```

```
sudo bin/post-index-task --file quickstart/tutorial/transform-index.json -
-url http://localhost:8081
[sudo] masami のパスワード:
Beginning indexing data for transform-tutorial
Task started: index_parallel_transform-tutorial_akhjpdbl_2024-08-05T13:30:32.070Z
Task log:     http://localhost:8081/druid/indexer/v1/task/index_parallel_transform-tutorial_akhjpdbl_2024-08-05T13:30:32.070Z/log
Task status:  http://localhost:8081/druid/indexer/v1/task/index_parallel_transform-tutorial_akhjpdbl_2024-08-05T13:30:32.070Z/status
Task index_parallel_transform-tutorial_akhjpdbl_2024-08-05T13:30:32.070Z still running...
Task index_parallel_transform-tutorial_akhjpdbl_2024-08-05T13:30:32.070Z still running...
Task finished with status: SUCCESS
Completed indexing data for transform-tutorial. Now loading indexed data onto the cluster...
transform-tutorial is 0.0% finished loading...
transform-tutorial loading complete! You may now query your data
```

クエリを実行すると変換データが確認できます

```sql
masami@masami-L /u/s/apache-druid-30.0.0> sudo bin/dsql
Welcome to dsql, the command-line client for Druid SQL.
Connected to [http://localhost:8082/].

Type "\h" for help.
dsql> select * from "transform-tutorial";
┌──────────────────────────┬────────────────┬──────────┬───────┬────────┬───────────────┐
│ __time                   │ animal         │ location │ count │ number │ triple-number │
├──────────────────────────┼────────────────┼──────────┼───────┼────────┼───────────────┤
│ 2018-01-01T05:01:00.000Z │ super-mongoose │        2 │     1 │    200 │           600 │
│ 2018-01-01T06:01:00.000Z │ super-snake    │        3 │     1 │    300 │           900 │
│ 2018-01-01T07:01:00.000Z │ super-octopus  │        1 │     1 │    100 │           300 │
└──────────────────────────┴────────────────┴──────────┴───────┴────────┴───────────────┘
Retrieved 3 rows in 0.03s.
```
