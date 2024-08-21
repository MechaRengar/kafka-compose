# kafka-docker-compose
# file này dùng để khỏi tạo server Kafka được quản lý với ZooKeeper
# Tích hợp quản lý Schema & Avro Serialization trong hệ thống

===============================================================================
version: '3'
services:
  zookeeper:
    image: confluentinc/cp-zookeeper:latest
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
      ZOOKEEPER_TICK_TIME: 2000
      ZOOKEEPER_AUTOPURGE_PURGE_INTERVAL: "0"
    ports:
      - 22181:2181
  
  kafka:
    image: confluentinc/cp-kafka:latest
    depends_on:
      - zookeeper
    ports:
      - 9092:9092  # External access
      - 29092:29092  # Internal Docker network access
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_LISTENERS: "INTERNAL://0.0.0.0:29092,EXTERNAL://0.0.0.0:9092"
      KAFKA_ADVERTISED_LISTENERS: "INTERNAL://kafka:29092,EXTERNAL://localhost:9092"
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: "INTERNAL:PLAINTEXT,EXTERNAL:PLAINTEXT"
      KAFKA_INTER_BROKER_LISTENER_NAME: INTERNAL
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
  
  schema-registry:
    image: confluentinc/cp-schema-registry:latest
    depends_on:
      - kafka
    ports:
      - 8386:8386
    environment:
      SCHEMA_REGISTRY_HOST_NAME: schema-registry
      SCHEMA_REGISTRY_KAFKASTORE_BOOTSTRAP_SERVERS: kafka:29092
      SCHEMA_REGISTRY_LISTENERS: http://0.0.0.0:8386

  kafka-ui:
    container_name: kafka-ui
    image: provectuslabs/kafka-ui:latest
    depends_on:
      - kafka
      - schema-registry
    ports:
      - 8080:8080
    environment:
      KAFKA_CLUSTERS_0_NAME: "docker-kafka-server"
      KAFKA_CLUSTERS_0_BOOTSTRAPSERVERS: "kafka:29092"  # Internal network access
      KAFKA_CLUSTERS_0_SCHEMAREGISTRY: "http://schema-registry:8386"

networks:
  default:
    driver: bridge
  ===========================================================================================
