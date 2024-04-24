---
title: Docker Kafka & Debezium
date: 2023-09-09 10:10:10 +/-TTTT
categories: [Kafka, Debezium]
tags: [Kafka]
---

# Rodando Kafka com Debezium

Muitas vezes, estamos lidando com um legado e, de alguma forma, precisamos escutar ou saber das operações que estão ocorrendo em determinadas tabelas. Uma forma de fazer isso é trabalhar com o Debezium, que irá ler todos os logs que ocorrerem em seu banco de dados, claro, apenas nas tabelas listadas em sua configuração.

# Como subir o Kafka com Debezium localmente?

Para configurar toda a infraestrutura necessária, vamos criar um docker-compose.

```yml
version: '3.7'

services:
  mysql-kafka:
    image: mysql:5.7
    environment:
      - MYSQL_DATABASE=kafka-test
      - MYSQL_ROOT_PASSWORD=root
    ports:
      - 3306:3306
    volumes:
      - .data/mysql:/var/lib/mysql
    networks:
      - kafka-test-network

  connect:
    image: debezium/connect:latest
    depends_on:
      - kafka
      - mysql-kafka
    environment:
      - BOOTSTRAP_SERVERS=kafka:9092
      - GROUP_ID=1
      - CONFIG_STORAGE_TOPIC=my_connect_configs
      - OFFSET_STORAGE_TOPIC=my_connect_offsets
      - STATUS_STORAGE_TOPIC=my_connect_statuses
    ports:
      - 8083:8083
      - 5005:5005
    networks:
      - kafka-test-network

  zookeeper:
    image: confluentinc/cp-zookeeper:latest
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
      ZOOKEEPER_TICK_TIME: 2000
    networks:
      - kafka-test-network
    deploy:
      resources:
        limits:
          cpus: '0.8'
          memory: 800M
        reservations:
          memory: 100M

  kafka:
    image: confluentinc/cp-kafka:latest
    depends_on:
      - zookeeper
    ports:
      - '9092:9092'
      - '9094:9094'
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
      KAFKA_NUM_PARTITIONS: 1
      KAFKA_DEFAULT_REPLICATION_FACTOR: 1
      KAFKA_AUTO_CREATE_TOPICS_ENABLE: 'true'
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_INTER_BROKER_LISTENER_NAME: INTERNAL
      KAFKA_LISTENERS: INTERNAL://:9092,OUTSIDE://:9094
      KAFKA_ADVERTISED_LISTENERS: INTERNAL://kafka:9092,OUTSIDE://localhost:9094
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: INTERNAL:PLAINTEXT,OUTSIDE:PLAINTEXT
    networks:
      - kafka-test-network
    deploy:
      resources:
        limits:
          cpus: '0.8'
          memory: 800M
        reservations:
          memory: 100M

networks:
  kafka-test-network:
    driver: bridge
    name: kafka-test-network
```

A primeira imagem que precisamos é do próprio MySQL. Nele, vamos criar um banco para posteriormente listar os logs.

Em seguida, iniciamos a imagem do Debezium e adicionamos as dependências do Kafka e do Zookeeper.

**Obs.:** Este docker-compose é apenas um exemplo. O recomendado é usar um arquivo .env para adicionar os parâmetros e configurações de nossas imagens.

# Subindo o kafka

Agora que já preparamos nossas imagens, vamos subir o Kafka. Para isso:

```

$docker-compose up

```

A partir desse momento, se tudo der certo, teremos toda a nossa infraestrutura rodando localmente para testarmos.

Você pode executar

```
docker ps
```

Para ver o estado dos containers ativos, você pode executar:

Se tudo ocorreu bem, vamos ver uma saída parecida com esta em nosso terminal:

![image](https://github.com/andremartinsds/flash-cards-api/assets/5201283/19b65bd7-e632-4b8a-9894-5a553cd8a027)

## Script SQL

Agora vamos criar uma tabela no banco para posteriormente inserirmos dados e ler como tópicos de criação no Debezium.

```sql
use `kafka-test`;
drop table if exists users;
CREATE table users(
    id int not null auto_increment,
    name varchar(50) not null,
    age int default null,
    surname varchar(40) default null,
    primary key (id)
);
```

## Connector

O Debezium possui sua própria API REST e diversos plugins. O que vamos precisar para ele começar a ler as alterações dessa tabela é:

```

curl --location 'localhost:8083/connectors' \
--header 'Content-Type: application/json' \
--data '{
    "name": "database-connector",
    "config": {
        "connector.class": "io.debezium.connector.mysql.MySqlConnector",
        "tasks.max": "1",
        "database.hostname": "mysql-kafka",
        "database.port": "3306",
        "database.user": "root",
        "database.password": "root",
        "database.server.id": "184054",
        "topic.prefix": "connector",
        "database.include.list": "kafka-test.users",
        "schema.history.internal.kafka.bootstrap.servers": "kafka:9092",
        "schema.history.internal.kafka.topic": "schema-changes.inventory"
    }
}'

```

# Inserindo dados

Agora que temos nosso plugin ativo, podemos adicionar dados no banco. Esses dados vão produzir tópicos que vamos poder consumir e posteriormente gerar eventos ou mesmo trabalhar com streams.

```sql

INSERT INTO users (name, age, surname) VALUES
('Alice', 25, 'Smith'),
('Bob', 30, 'Johnson'),
('Charlie', 35, 'Williams'),
('David', 40, 'Jones'),
('Emma', 22, 'Brown'),
('Frank', 28, 'Davis'),
('Grace', 33, 'Miller'),
('Henry', 45, 'Wilson'),
('Isabella', 27, 'Moore'),
('Jack', 32, 'Taylor'),
('Kate', 29, 'Anderson'),
('Liam', 38, 'Thomas'),
('Mia', 26, 'Jackson'),
('Noah', 31, 'White'),
('Olivia', 34, 'Harris'),
('Patrick', 37, 'Clark'),
('Quinn', 41, 'Allen'),
('Riley', 36, 'Scott'),
('Sophia', 39, 'Adams'),
('Thomas', 42, 'Lee'),
('Uma', 23, 'Parker'),
('Vincent', 44, 'Robinson'),
('Willow', 43, 'Nelson'),
('Xander', 24, 'Evans'),
('Yasmine', 21, 'Murphy'),
('Zoe', 46, 'Carter'),
('Aaron', 20, 'Baker'),
('Bella', 47, 'Wright'),
('Caleb', 19, 'Garcia'),
('Diana', 48, 'King'),
('Ethan', 50, 'Hall'),
('Fiona', 49, 'Adams'),
('Gabriel', 51, 'Mitchell'),
('Hannah', 52, 'Thompson'),
('Ian', 53, 'Green'),
('Julia', 54, 'Phillips'),
('Kevin', 55, 'Cook'),
('Laura', 56, 'Rivera'),
('Michael', 57, 'Perez'),
('Nora', 58, 'Torres'),
('Oscar', 59, 'Hill'),
('Penelope', 60, 'Young'),
('Quentin', 61, 'Gonzalez'),
('Rachel', 62, 'Hughes'),
('Samuel', 63, 'Turner'),
('Tiffany', 64, 'Parker'),
('Ulysses', 65, 'Ward'),
('Victoria', 66, 'Foster'),
('Walter', 67, 'Stewart'),
('Xavier', 68, 'Wells'),
('Yolanda', 69, 'Bennett'),
('Zachary', 70, 'Gray'),
('Arthur', 71, 'Sullivan'),
('Beatrice', 72, 'Cole'),
('Christopher', 73, 'Morris'),
('Danielle', 74, 'Gutierrez'),
('Edward', 75, 'Perry'),
('Felicity', 76, 'Barnes'),
('George', 77, 'Knight'),
('Holly', 78, 'Bryant'),
('Ivan', 79, 'Reed'),
('Jessica', 80, 'Banks'),
('Kenneth', 81, 'Lopez'),
('Lily', 82, 'Harrison'),
('Matthew', 83, 'Gomez'),
('Natalie', 84, 'Fisher'),
('Oliver', 85, 'Murray'),
('Patricia', 86, 'Marshall'),
('Quincy', 87, 'Dean'),
('Rebecca', 88, 'Gilbert'),
('Simon', 89, 'Wagner'),
('Tara', 90, 'Dunn'),
('Ursula', 91, 'Mcdonald'),
('Victor', 92, 'Pierce'),
('Wendy', 93, 'Gordon'),
('Xena', 94, 'Holmes'),
('Yvonne', 95, 'Henry'),
('Zelda', 96, 'Hunt'),
('Andrew', 97, 'Rose'),
('Barbara', 98, 'Owens'),
('Carl', 99, 'Hudson'),
('Daisy', 100, 'Bishop');

```

E assim concluímos o processo de configuração do Kafka com Debezium, estabelecendo uma estrutura local para monitoramento de operações em bancos de dados legados. Com este ambiente configurado, podemos gerar eventos a partir das alterações realizadas nas tabelas do banco de dados, facilitando a integração e o processamento de dados em tempo real. Se precisar de mais alguma informação ou tiver alguma dúvida, estou à disposição para ajudar!
