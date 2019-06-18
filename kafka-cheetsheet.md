## Kafka Cheetsheet

##### create local cluster

```bash
function kafka-up() {
    cat >docker-compose.yml<<END
    version: '3.5'
    
    networks:
      default:
        name: kafka_network
        driver: bridge
    services:
      zookeeper:
        image: confluentinc/cp-zookeeper:latest
        ports:
          - 2181:2181
        environment:
          ZOOKEEPER_CLIENT_PORT: 2181
          ZOOKEEPER_TICK_TIME: 2000
    
      kafka:
        image: confluentinc/cp-kafka:latest
        depends_on:
          - zookeeper
        ports:
          - 29092:29092
        environment:
          KAFKA_BROKER_ID: 1
          KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
          KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka:9092,PLAINTEXT_HOST://localhost:29092
          KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT
          KAFKA_INTER_BROKER_LISTENER_NAME: PLAINTEXT
          KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
    END
    
    docker-compose up -d 
    rm -f docker-compose.yml
    echo "associated containers should be run in the network kafka_network"
}
```

##### destroy local cluster

```bash
function kafka-down() {
    cat >docker-compose.yml<<END
    version: '3.5'
    
    networks:
      default:
        name: kafka_network
        driver: bridge
    services:
      zookeeper:
        image: confluentinc/cp-zookeeper:latest
        ports:
          - 2181:2181
        environment:
          ZOOKEEPER_CLIENT_PORT: 2181
          ZOOKEEPER_TICK_TIME: 2000
    
      kafka:
        image: confluentinc/cp-kafka:latest
        depends_on:
          - zookeeper
        ports:
          - 29092:29092
        environment:
          KAFKA_BROKER_ID: 1
          KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
          KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka:9092,PLAINTEXT_HOST://localhost:29092
          KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT
          KAFKA_INTER_BROKER_LISTENER_NAME: PLAINTEXT
          KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
    END
    
    docker-compose down
    rm -f docker-compose.yml
    echo "local cluster destroyed"
}
```
