# Transform Kafka Streams Application

For kafka streams study.

kaggleの[New York City Airbnb Open Data](https://www.kaggle.com/dgomonov/new-york-city-airbnb-open-data/downloads/new-york-city-airbnb-open-data.zip/3)をデータソースに、[Transform a stream](https://docs.confluent.io/current/streams/developer-guide/dsl-api.html#transform-a-stream)に記載のtransformを試す。

元のkaggleのデータは単一のcsvだが、`host_id`, `host_name`の表だ切り出した[ab-nyc-2019-host.csv](/streams/Transform/src/main/resources/ab-nyc/ab-nyc-2019-host.csv)と、元のデータから`host_name`カラムを削除した[ab-nyc-2019-room.csv](/streams/Transform/src/main/resources/ab-nyc/ab-nyc-2019-room.csv)を用意する。

## Driver App

ClassNotFoundExceptionが返ってしまう。

```shell
(⎈ |local:default)kadono% java -cp .:build/libs/KStreamTransform.jar com.takzok.kafka.streams.TransformDriver                                                                                                                                               [~/local/dev/kafka-streams/streams/Transform][master]
Exception in thread "main" java.lang.NoClassDefFoundError: org/apache/avro/generic/GenericRecordBuilder
        at com.takzok.kafka.streams.TransformDriver.produceInputs(TransformDriver.java:43)
        at com.takzok.kafka.streams.TransformDriver.main(TransformDriver.java:28)
Caused by: java.lang.ClassNotFoundException: org.apache.avro.generic.GenericRecordBuilder
        at java.net.URLClassLoader.findClass(URLClassLoader.java:381)
        at java.lang.ClassLoader.loadClass(ClassLoader.java:424)
        at sun.misc.Launcher$AppClassLoader.loadClass(Launcher.java:349)
        at java.lang.ClassLoader.loadClass(ClassLoader.java:357)
        ... 2 more
```
