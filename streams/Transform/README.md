# Transform Kafka Streams Application

For kafka streams study.

kaggleの[New York City Airbnb Open Data](https://www.kaggle.com/dgomonov/new-york-city-airbnb-open-data/downloads/new-york-city-airbnb-open-data.zip/3)をデータソースに、[Transform a stream](https://docs.confluent.io/current/streams/developer-guide/dsl-api.html#transform-a-stream)に記載のtransformを試す。

元のkaggleのデータは単一のcsvだが、`host_id`, `host_name`の表だ切り出した[ab-nyc-2019-host.csv](/streams/Transform/src/main/resources/ab-nyc/ab-nyc-2019-host.csv)と、元のデータから`host_name`カラムを削除した[ab-nyc-2019-room.csv](/streams/Transform/src/main/resources/ab-nyc/ab-nyc-2019-room.csv)を用意する。
