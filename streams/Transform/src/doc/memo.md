# memo

## schema registry error

スキーマ登録でエラー。
この前に一度、`host_id`をtype:intとしたスキーマで投げていたことが原因?

```bash
Exception in thread "main" org.apache.kafka.common.errors.SerializationException: Error registering Avro schema: {"type":"record","name":"AirBnbNYCHost","namespace":"com.takzok.kafka.avro","fields":[{"name":"host_id","type":"string"},{"name":"host_name","type":"string"}]}
Caused by: io.confluent.kafka.schemaregistry.client.rest.exceptions.RestClientException: Schema being registered is incompatible with an earlier schema; error code: 409
        at io.confluent.kafka.schemaregistry.client.rest.RestService.sendHttpRequest(RestService.java:230)
        at io.confluent.kafka.schemaregistry.client.rest.RestService.httpRequest(RestService.java:256)
        at io.confluent.kafka.schemaregistry.client.rest.RestService.registerSchema(RestService.java:356)
        at io.confluent.kafka.schemaregistry.client.rest.RestService.registerSchema(RestService.java:348)
        at io.confluent.kafka.schemaregistry.client.rest.RestService.registerSchema(RestService.java:334)
        at io.confluent.kafka.schemaregistry.client.CachedSchemaRegistryClient.registerAndGetId(CachedSchemaRegistryClient.java:168)
        at io.confluent.kafka.schemaregistry.client.CachedSchemaRegistryClient.register(CachedSchemaRegistryClient.java:222)
        at io.confluent.kafka.schemaregistry.client.CachedSchemaRegistryClient.register(CachedSchemaRegistryClient.java:198)
        at io.confluent.kafka.serializers.AbstractKafkaAvroSerializer.serializeImpl(AbstractKafkaAvroSerializer.java:70)
        at io.confluent.kafka.serializers.KafkaAvroSerializer.serialize(KafkaAvroSerializer.java:53)
        at org.apache.kafka.common.serialization.Serializer.serialize(Serializer.java:62)
        at org.apache.kafka.clients.producer.KafkaProducer.doSend(KafkaProducer.java:894)
        at org.apache.kafka.clients.producer.KafkaProducer.send(KafkaProducer.java:856)
        at org.apache.kafka.clients.producer.KafkaProducer.send(KafkaProducer.java:743)
        at com.takzok.kafka.streams.TransformDriver.produceInputs(TransformDriver.java:63)
        at com.takzok.kafka.streams.TransformDriver.main(TransformDriver.java:28)
```

### 登録されたスキーマ情報の削除

[schema registryのAPI](https://docs.confluent.io/current/schema-registry/develop/api.html)

#### subjectsの一覧

```
GET http://localhost:8081/subjects
Accept: application/vnd.schemaregistry.v1+json, application/vnd.schemaregistry+json, application/json
```

```json
[
  "NYCHosts-value",
  "kafka-music-charts-song-play-count-repartition-value",
  "kafka-music-charts-top-five-songs-by-genre-repartition-value",
  "PageViews-value",
  "kafka-music-charts-all-songs-changelog-value",
  "kafka-music-charts-song-play-count-repartition-key",
  "pageview-region-lambda-example-UserProfiles-STATE-STORE-0000000002-changelog-value",
  "song-feed-value",
  "pageview-region-lambda-example-KSTREAM-AGGREGATE-STATE-STORE-0000000011-repartition-value",
  "play-events-value",
  "kafka-music-charts-song-play-count-changelog-value",
  "kafka-music-charts-top-five-songs-repartition-value",
  "kafka-music-charts-KSTREAM-MAP-0000000004-repartition-value",
  "pageview-region-lambda-example-KSTREAM-MAP-0000000001-repartition-value",
  "UserProfiles-value"
]
```

#### subjectのバージョン確認

特定のsubjectが保持しているバージョンを確認する

```
GET http://localhost:8081/subjects/NYCHosts-value/versions
Accept: application/vnd.schemaregistry.v1+json, application/vnd.schemaregistry+json, application/json
```

```json
[1]
```

#### subjectの削除

```
DELETE http://localhost:8081/subjects/NYCHosts-value
Accept: application/vnd.schemaregistry.v1+json, application/vnd.schemaregistry+json, application/json
```

```json
[1]
```

### シリアライズでのエラー

srtingをdouble型にcastできない。

```log
Exception in thread "main" org.apache.kafka.common.errors.SerializationException: Error serializing Avro message
Caused by: java.lang.ClassCastException: java.lang.String cannot be cast to java.lang.Double
        at org.apache.avro.generic.GenericDatumWriter.writeWithoutConversion(GenericDatumWriter.java:133)
        at org.apache.avro.generic.GenericDatumWriter.write(GenericDatumWriter.java:75)
        at org.apache.avro.generic.GenericDatumWriter.writeField(GenericDatumWriter.java:166)
        at org.apache.avro.generic.GenericDatumWriter.writeRecord(GenericDatumWriter.java:156)
        at org.apache.avro.generic.GenericDatumWriter.writeWithoutConversion(GenericDatumWriter.java:118)
        at org.apache.avro.generic.GenericDatumWriter.write(GenericDatumWriter.java:75)
        at org.apache.avro.generic.GenericDatumWriter.write(GenericDatumWriter.java:62)
        at io.confluent.kafka.serializers.AbstractKafkaAvroSerializer.serializeImpl(AbstractKafkaAvroSerializer.java:92)
        at io.confluent.kafka.serializers.KafkaAvroSerializer.serialize(KafkaAvroSerializer.java:53)
        at org.apache.kafka.common.serialization.Serializer.serialize(Serializer.java:62)
        at org.apache.kafka.clients.producer.KafkaProducer.doSend(KafkaProducer.java:894)
        at org.apache.kafka.clients.producer.KafkaProducer.send(KafkaProducer.java:856)
        at org.apache.kafka.clients.producer.KafkaProducer.send(KafkaProducer.java:743)
        at com.takzok.kafka.streams.TransformDriver.produceInputs(TransformDriver.java:91)
        at com.takzok.kafka.streams.TransformDriver.main(TransformDriver.java:28)
```

csvから読み取っているときにstringで入れてしまってるのが悪い?

```java
 nycRoomBuilder.set("price", record.get("price"));
```

こうする。他にもintがあるので同様にcastする。

```java
nycRoomBuilder.set("price", Double.parseDouble(record.get("price")));
```

### null valueへの対応

値が無いためExceptionを吐く。

```log
Exception in thread "main" java.lang.NumberFormatException: empty String
        at sun.misc.FloatingDecimal.readJavaFormatString(FloatingDecimal.java:1842)
        at sun.misc.FloatingDecimal.parseDouble(FloatingDecimal.java:110)
        at java.lang.Double.parseDouble(Double.java:538)
        at com.takzok.kafka.streams.TransformDriver.produceInputs(TransformDriver.java:87)
        at com.takzok.kafka.streams.TransformDriver.main(TransformDriver.java:28)
```

avroスキーマのtypeにnullを追加する。

```json
{"name": "last_review", "type": ["string", "null"]},
```

