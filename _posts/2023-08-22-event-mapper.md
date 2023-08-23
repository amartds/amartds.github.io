---
title: Event Mapper
date: 2023-08-22 10:10:10 +/-TTTT
categories: [pattern, padrões]
tags: [pattern, padrões, Typescript]
---

# Event Mapper

Imagine que você está em um cenário onde tem varias funções que precisam ser executadas conforme um caminho seja passado por parâmetro, por exemplo: dentro de um domínio de de clientes precisamos executar funções a partir de uma chamada desse path, e nesse caso temos sabedoria para não resolver isso com switch case ou da forma abaixo.

```ts
class Cliente {
  public constructor(private _id: number, private _name: string) {}

  get clientes(): number {
    return this._id;
  }

  // outros comportamentos do cliente
}

class ClienteService {
  public execute(path: string) {
    if (path === "sincronizarImagens") {
      // lógica para sincronizar as imagens do cliente
    }

    if (path === "sincronizarReport") {
      // lógica para sincronizar os relatórios do cliente
    }

    if (path === "sincronizarLogs") {
      // lógica para sincronizar os logs do cliente
    }
  }
}
```

essa não é uma boa solução para esse problema

## Iniciando Solução

Podemos pensar em excluir essa lógica de ifs e uma forma de se fazer isso é com o pattern event-mapper.

A primeira parte é definir um tipo onde teremos uma chave e o tipo de parâmetros da função de `callback`.
Perceba que nesse ponto podemos entrar com qualquer `string` no tipo da chave, e nos parâmetros estamos esperando o _ID_ e não vamos retornar nada.

```ts
type TypeEventMapper = {
  [key: string]: (id: number) => void;
};
```

Agora precisamos de uma função para resolver nossa callback conforme o parâmetro que passarmos, mas o detalhe é que não vamos passar parâmetro nenhum, te mostro já.

```ts

class ClientService {
    private resolverEventMapperByName(): TypeEventMapper {
        return {
            sincronizarImagens:(id: number) => this.searchImages(id),
            sincronizarReport(id: number) => this.sincronizarReport(id),
            sincronizarLogs(id: number) => this.sincronizarLogs(id),
        }
    }

    sincronizarImagens(id: number): void {
        // lógica para sincronizar as imagens do cliente
    }

    sincronizarReport(id: number): void {
        // lógica para sincronizar os relatórios do cliente
    }

    sincronizarLogs(id: number): void {
        // lógica para sincronizar os logs do cliente
    }
}
```

Conforme o exemplo acima, resolvemos nossa execução de forma transparente, sem a necessidade de verificações estranhas.

## Como usar?

```ts
class ClientService {

    execute(path: string) {

        const id = 123;

        const handler = this.resolverEventMapperByName()[path]

        handler(id);

    }

    private resolverEventMapperByName(): TypeEventMapper {
        return {
            sincronizarImagens:(id: number) => this.searchImages(id),
            sincronizarReport(id: number) => this.sincronizarReport(id),
            sincronizarLogs(id: number) => this.sincronizarLogs(id),
        }
    }

    sincronizarImagens(id: number): void {
        // lógica para sincronizar as imagens do cliente
    }

    sincronizarReport(id: number): void {
        // lógica para sincronizar os relatórios do cliente
    }

    sincronizarLogs(id: number): void {
        // lógica para sincronizar os logs do cliente
    }
}

```

Dessa forma conseguimos resolver nossa chamada com `Event-Mapper` de forma transparente, há outras formas de resolver isso também, mas essa é uma forma que deixa as coisas bem legíveis.

é isso :)
