# Apache Kafka からストリーミングデータを読み込む

次のチュートリアルを実施します
https://druid.apache.org/docs/latest/tutorials/tutorial-kafka

Kafka をダウンロードする

```
curl -O https://archive.apache.org/dist/kafka/3.7.1/kafka_2.12-3.7.1.tgz
sudo tar -xzf kafka_2.12-3.7.1.tgz
cd kafka_2.12-3.7.1/
```

Kafka ブローカーを起動

```
 ./bin/kafka-server-start.sh config/server.properties
```

Kafka トピックの作成

```
sudo ./bin/kafka-topics.sh --create --topic kttm --bootstrap-server localhost:9092
Created topic kttm.
```

Kafka にデータをロードする

Kafka のルートディレクトリにフォルダを作成し、そこにデータをダウンロードします

```
mkdir sample-data
cd sample-data/
curl -O https://static.imply.io/example-data/kttm-nested-v2/kttm-nested-v2-2019-08-25.json.gz
```

Kafka ルート ディレクトリでサンプルイベントを Kafka トピックに登録します

```
cd ..
zcat ./sample-data/kttm-nested-v2-2019-08-25.json.gz | ./bin/kafka-console-producer.sh --broker-list localhost:9092 --topic kttm
```

Druid にデータを登録します

DataSorce の Stream を選択
![2](https://github.com/user-attachments/assets/dc46cb21-d6f6-4652-a152-b8d3f74bc891)

ブートストラップ サーバーに「localhost:9092」を
トピックに「kttm」を入力
![3](https://github.com/user-attachments/assets/2efa677d-4d46-42b7-9565-426e88a9cb14)

右下の Next 〇〇ボタンを押して、Tune まで進める
Use Eariest offset を True に  
granualy を day に設定して、データを公開する
![4](https://github.com/user-attachments/assets/0bfc9b39-f81d-4168-a8f5-9c78f315da52)

データの公開されると、クエリが実行できるようになる
![5](https://github.com/user-attachments/assets/00f51075-5e59-441d-9ce5-745e405646bf)

### Submit a supervisor spec

データ ローダーを使用する代わりに、スーパーバイザー仕様を Druid に送信することもできます。

Web コンソールから Task を選択し、Submit Json Task を実行します
![6](https://github.com/user-attachments/assets/75aca9aa-3145-45c7-99fc-a47a8c801ff5)

Json スペックを貼り付けて Submit します
![7](https://github.com/user-attachments/assets/c9865e0a-0d15-40be-8cc4-c8a14d0325df)

### API を使用する

Druid API を使用してスーパーバイザー仕様を送信することもできます

サンプルスペックをダウンロードします

```
curl -o kttm-kafka-supervisor.json https://raw.githubusercontent.com/apache/druid/master/docs/assets/files/kttm-kafka-supervisor.json
```

ファイルスペックを送信します

```
curl -X POST -H 'Content-Type: application/json' -d @kttm-kafka-superv
isor.json http://localhost:8081/druid/indexer/v1/supervisor
{"id":"kttm-kafka-supervisor-api"}⏎
```

タスクから確認できます
![8](https://github.com/user-attachments/assets/fbb66028-5def-4706-80c2-71ac8553aa42)
