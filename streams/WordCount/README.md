# Word Count Kafka Streams Application
For kafka streams study.
This application works as following.
- Consume records from  `TextLines` topic.
- Split each text line by whitespace into words.
- Count the occurences of each word.
- Write each word(message key) and its occurences(message value) to `WordWithCounts` topic.

## Usage
  1. Run `gradle dockerBuildImage`. Check the task is finished successfully.
  1. Run `docker images | grep takzok/word-count`. Check the container image is built. 
  1. Copy `docker-compose.override.yml` to `kafka-streams` directory and run `docker-compose up -d` command at `kafka-streams` directory.

`word-count` container may start quickly and it causes consuming error. You will recover it by restarting word-count container.
