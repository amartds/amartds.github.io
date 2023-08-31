---
title: Configurando o Elastic APM com NestJS
date: 2023-08-30 10:10:10 +/-TTTT
categories: [NestJS, Docker]
tags: [Padrões, TypeScript]
---

# Configurando o Elastic APM com NestJS

Neste guia, você aprenderá como configurar uma aplicação NestJS para rodar em um container Docker e como adicionar monitoramento a esta aplicação usando o APM (Application Performance Monitoring) da Elastic.

O primeiro passo é criar sua aplicação NestJS e, em seguida instalar o APM da Elastic:

```shell

nest new apm-monitor-nest-example

npm install elastic-apm-node

```

Agora vamos montar um container bem simples com nossa aplicação NestJs, adicione o arquivo Dockerfile com o conteúdo

```yml
FROM node:18

WORKDIR /usr/src/app

COPY package*.json ./
COPY tsconfig.json ./
COPY src ./src

RUN npm install

RUN npm run build

EXPOSE 3000

CMD [ "npm", "start" ]
```

**obs.: apenas um exemplo de container, o melhor seria utilizar uma imagem menor e adicionar uma estratégia de staging-build**

como pode perceber é um container simples, apenas adicionando o workdir copiando as configurações, projeto, instalando e rodando.

para testar nesse ponto basta rodar um build

```shell
docker build -t app .
```

## Adicionando o APM na Inicialização do NestJS

Agora, vamos adicionar as configurações do APM ao arquivo main.ts do seu aplicativo NestJS:

```ts
import { NestFactory } from "@nestjs/core";
import { AppModule } from "./app.module";
import * as apm from "elastic-apm-node";

async function bootstrap() {
  apm.start({
    serviceName: "apm-monitor-nest",
    serverUrl: "http://apm-server:8200",
    logLevel: "debug",
    active: true,
  });

  const app = await NestFactory.create(AppModule);
  await app.listen(3000);
}
bootstrap();
```

## Adicionando docker-compose.yml

agora vamos montar nosso docker-compose com os containers do elastic.

No primeiro serviço adicionamos a referência para nosso Dockerfile, em seguida adicionamos o elasticsearch que se trata da ferramenta onde os dados(index) são armazenados, em seguida adicionamos o kibana que se trata de um dashboard onde podemos analisar de forma gráfica varias ferrametas da elastic e por fim temos o apm que é onde vamos capturar os dados de monitoração da nossa aplicação.

Também adicionamos uma rede para conectar todos os containers.

```yml
version: "3.7"

services:
  app:
    container_name: app
    build:
      context: ./
      dockerfile: ./Dockerfile
    ports:
      - "3000:3000"
    depends_on:
      - elasticsearch
    environment:
      ELASTICSEARCH_URL: "http://elasticsearch:9200"
    networks:
      - apm-monitor-nest

  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.15.0
    container_name: elasticsearch
    environment:
      - discovery.type=single-node
    ports:
      - "9200:9200"
      - "9300:9300"
    networks:
      - apm-monitor-nest

  kibana:
    image: kibana:7.14.2
    container_name: kibana
    links:
      - elasticsearch
    ports:
      - "5601:5601"
    networks:
      - apm-monitor-nest

  apm-server:
    image: docker.elastic.co/apm/apm-server:7.15.0
    container_name: apm-server
    ports:
      - "8200:8200"
    environment:
      - output.elasticsearch.hosts=["elasticsearch:9200"]
    networks:
      - apm-monitor-nest

networks:
  apm-monitor-nest:
    driver: bridge
    name: apm-monitor-nest

volumes:
  es_data: {}
```

## Rodando

Para rodar basta executar:

```shell
docker-compose up
```

em seguida você pode fazer um get em: localhost:3000

dessa forma será adicionado a requisição e seus dados dentro do apm que fica em: `http://localhost:5601/app/apm/services/apm-monitor-nest/`

link para o exemplo:
[apm-monitor-nest-poc](https://github.com/andremartinsds/apm-monitor-nest-poc)

é isso :)
