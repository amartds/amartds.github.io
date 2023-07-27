---
title: Como criar rapidamente uma conexão com um banco h2 no Spring-boot
date: 2022-07-23 10:10:10 +/-TTTT
categories: [Java]
tags: [spring-boot, java]
---

## Requisitos

Se você não sabe como configurar um projeto no spring-boot ou não entende a injeção de dependência do spring e como configurar beans esse tutorial ainda não é para você :) vamos em frente.

## Iniciando

Utilizar banco de dados em memória é uma mão na roda, o motivo é que não precisamos instanciar containers ou mesmo baixar ou ter instalado um banco como mysql ou postgresql, vamos fazer isso de forma bem rápida :)

Após ter seu projeto spring-boot configurado é preciso adicionar algumas depedências no arquivos pom.xml

```xml

<dependency>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>

<dependency>
<groupId>com.h2database</groupId>
<artifactId>h2</artifactId>
<scope>runtime</scope>
</dependency>

```

Acima adicionamos o stater do jpa bem como o artefato para o h2.

Agora precisamos adicionar as configurações no arquivo de properties do nosso projeto

```

spring.datasource.url=jdbc:h2:file:./data/minha-base-de-dados
spring.datasource.driverClassName=org.h2.Driver
spring.datasource.username=sa
spring.datasource.password=password

server.port=3333

spring.h2.console.enabled=true
spring.h2.console.path=/h2-console

spring.jpa.database-platform=org.hibernate.dialect.H2Dialect
spring.jpa.hibernate.ddl-auto=update

```

## Adicionando service com runtime para testes

Vamos adicionar um service no spring para quando a aplicação iniciar já popularmos alguma coisa, mas pra isso vamos adicionar uma entidade de banco e um repository padrão do spring data jpa

```java

@Entity()
@Table(name = "produtos")
public class Produto {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(nullable = false)
    private Long id;

    @Column(length = 50, nullable = false)
    private String name;

    public Produto(String name) {
        this.name = name;
    }

    public Produto() {
    }

    public Long getId() {
        return id;
    }

    public String getName() {
        return name;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public void setName(String name) {
        this.name = name;
    }
}

```

```java

public interface ProdutoRepository extends JpaRepository<Produto, Long> {
}

```

Agora podemos adicionar o service para população do banco

```java

@Service
public class ProdutoRunner implements ApplicationRunner {

    private ProdutoRepository repository;
    public ProdutoRunner(ProdutoRepository repository) {

        this.repository = repository;
    }

    @Override
    public void run(ApplicationArguments args) throws Exception {

        List<Produto> produtos = new ArrayList<Produto>();

        Produto produto = new Produto("Geladeira");
        Produto produto2 = new Produto("Televisão");
        Produto produto3 = new Produto("Maquina de lavar");
        Produto produto4 = new Produto("Microondas");
        Produto produto5 = new Produto("Fogão");

        produtos.add(produto);
        produtos.add(produto2);
        produtos.add(produto3);
        produtos.add(produto4);
        produtos.add(produto5);

        this.repository.saveAll(produtos);
    }
}

```

sempre que a aplicação iniciar esse service acima irá adicionar no banco h2 esses dados.

### Startando e testando

Agora basta levantar sua api fazer a conexão em seu browser

![Screenshot from 2023-07-26 23-07-13](https://github.com/amartds/amartds/assets/5201283/13eba73d-ead9-455e-af9a-b4c3db1f1af7)

podemos conferir como ficou

![Screenshot from 2023-07-26 23-10-22](https://github.com/amartds/amartds/assets/5201283/3c3cc91d-ecab-494f-a1ce-79df38148e1e)

valewww
