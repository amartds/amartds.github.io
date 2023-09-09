---
title: Vetor não ordenado
date: 2023-09-09 10:10:10 +/-TTTT
categories: [Estrutura de Dados]
tags: [TypeScript, Python]
---

# Vetor não ordenado

Vetor não ordenados são estruturas de dados para armazenamos dados em sequência, se imaginarmos uma variável em um programa ela é composta por um dado anexado em um endereço de memória, quando se trata de um vetor estamos falando de conjuntos de dados.

# Qual a utilidade de um vetor não ordenado?

Uma vetor não ordenado tem a utilidade de guardar agrupamentos de dados que como o nome já diz não necessitam estritamente de uma ordem, vetores sempre terão um limite e esse é diferencial para uma lista, em um vetor não ordenado pode se guardar um grupo de objetos ou primitivos temporários. vetore não ordenados é uma boa escolha na hora de lidar com poucos dados fixos.

# implementação vetor não ordenado em Python

```py

import numpy as np

'''
Vetor não ordenado de inteiros, não permite inserção de valores duplicados
Operações:
    inserir
    imprimir
    excluir
    pesquisar
'''


class VetorNaoOrdenado:
    def __init__(self, capacidade_vetor):
        self.capacidade_vetor = capacidade_vetor
        self.ultima_posicao = -1
        self.valores = np.empty(self.capacidade_vetor, dtype=int)

    def insere_no_vetor(self, valor):
        self.ultima_posicao += 1
        self.valores[self.ultima_posicao] = valor

    def insere(self, valor):
        if self.ultima_posicao == self.capacidade_vetor - 1:
            print('A capacidade foi atingida')
        else:
            if self.ultima_posicao == -1:
                self.insere_no_vetor(valor)
            else:
                for i in range(self.ultima_posicao + 1):
                    if self.valores[i] == valor:
                        print(f'Não é permitido inserir {valor} já existe no vetor')
                        return
                self.insere_no_vetor(valor)

    def pesquisa_linear(self, valor):
        for i in range(self.ultima_posicao + 1):
            if self.valores[i] == valor:
                return i
        return -1

    def excluir(self, valor):
        if self.ultima_posicao == -1:
            return -1
        indice_pesquisado = self.pesquisa_linear(valor)
        for i in range(indice_pesquisado, self.ultima_posicao):
            self.valores[i] = self.valores[i + 1]
        self.ultima_posicao -= 1

    def imprime_valores(self):
        if self.ultima_posicao == -1:
            print('O vetor está vazio')
        else:
            for i in range(self.ultima_posicao + 1):
                print(i, '-->', '[', self.valores[i], ']')


if __name__ == '__main__':
    vetor = VetorNaoOrdenado(5)
    vetor.insere(1)
    vetor.insere(8)
    vetor.insere(7)
    vetor.insere(6)
    vetor.insere(5)
    vetor.excluir(7)
    vetor.imprime_valores()

    print('Posicao', vetor.pesquisa_linear(19))


```

# implementação vetor não ordenado em Typescript

```ts
export class VetorNaoOrdenado {
  private ultimaPosicao: number;
  private vetor: number[];

  constructor(private capacidade: number) {
    this.ultimaPosicao = -1;
    this.vetor = [];
  }

  valorNaoEncontrado(value: number): boolean {
    return this.procurar(value) === -1;
  }

  inserir(value: number) {
    if (this.ultimaPosicao == this.capacidade - 1) {
      return -1;
    }
    if (this.valorNaoEncontrado(value)) {
      this.ultimaPosicao += 1;
      this.vetor[this.ultimaPosicao] = value;
    } else {
      return -1;
    }
  }

  remover(value: number) {
    const foundIndex = this.procurar(value);
    if (!this.vetor[foundIndex]) {
      return -1;
    }

    for (let i = 0; i < this.capacidade - 1; i++) {
      if (this.vetor[i] == value) {
        for (let j = i; j < this.capacidade - 1; j++) {
          this.vetor[j] = this.vetor[j + 1];
        }
      }
    }
    this.ultimaPosicao -= 1;
  }

  procurar(value: number): number {
    let result = -1;
    for (let i = 0; i < this.capacidade - 1; i++) {
      if (this.vetor[i] === value) {
        result = i;
      }
    }
    return result;
  }

  imprimir() {
    for (let i = 0; i < this.capacidade - 1; i++) {
      if (!this.vetor[i]) continue;
      console.log(`[Posição: ${i}] -> Valor: ${this.vetor[i]}`);
    }
  }
}

const lista = new VetorNaoOrdenado(5);

lista.inserir(1);
lista.inserir(3);
lista.inserir(2);
lista.inserir(10);

lista.remover(1);

lista.imprimir();
```

é isso :)
