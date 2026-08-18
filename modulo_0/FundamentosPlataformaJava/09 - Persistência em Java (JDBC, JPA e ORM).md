---
aliases:
  - Persistência em Java (JDBC, JPA e ORM)
tags:
  - java
  - fundamentos
  - modulo_0
  - persistencia
---

# Persistência em Java: JDBC, JPA e ORM

Três camadas de abstração empilhadas: **JDBC** fala SQL cru com o banco; **ORM** é o conceito de mapear objetos para tabelas; **JPA** é a especificação Java que padroniza como fazer ORM. Cada uma resolve um problema diferente, e é raro (mas possível) usar uma sem a outra.

## JDBC (Java Database Connectivity)

API de baixo nível para conectar e executar SQL diretamente contra um banco relacional: `Connection`, `Statement`/`PreparedStatement`, `ResultSet`. Todo driver de banco (PostgreSQL, MySQL, Oracle, H2...) implementa essa interface — é a camada que faz Java conseguir "falar" com qualquer banco relacional sem código específico por fornecedor na aplicação.

**Curiosidades:**
- Existem historicamente 4 "tipos" de driver JDBC; hoje praticamente todo driver relevante é **Tipo 4** (implementação pura Java que fala o protocolo de rede nativo do banco, sem camada intermediária em C/ODBC).
- Toda ferramenta ORM do ecossistema Java — Hibernate incluso — usa JDBC por baixo dos panos. Não existe "pular" o JDBC de fato; ele só fica escondido atrás de abstrações.
- `PreparedStatement` não é só sobre performance (reuso de plano de execução) — é a defesa padrão contra **SQL injection**, já que parametriza os valores em vez de concatenar strings na query.

## ORM (Object-Relational Mapping)

Não é uma sigla exclusiva de Java, é um **padrão de arquitetura**: a técnica de mapear objetos (com atributos, referências e herança) para tabelas relacionais (linhas, colunas, chaves estrangeiras) e vice-versa, resolvendo o chamado *impedance mismatch* (o modelo orientado a objetos e o modelo relacional representam dados de formas fundamentalmente diferentes — por exemplo, herança e grafos de referência não têm equivalente direto em tabelas).

**Curiosidade:** ORM existe em praticamente toda linguagem (SQLAlchemy em Python, ActiveRecord em Ruby/Rails, Entity Framework em .NET) — JPA é apenas a formalização **padronizada** desse conceito para o mundo Java, o que permite trocar de implementação (Hibernate por EclipseLink, por exemplo) sem reescrever o código de acesso a dados, já que ambos implementam a mesma interface JPA.

## JPA (Jakarta Persistence API)

A especificação Java que padroniza ORM: anotações (`@Entity`, `@Id`, `@OneToMany`...), a linguagem de consulta **JPQL** (orientada a objetos, não a tabelas), e o `EntityManager` como ponto central de gerenciamento do ciclo de vida das entidades. JPA é só a **interface** — precisa de uma implementação concreta para funcionar, sendo **Hibernate** a mais usada de longe (também EclipseLink e OpenJPA).

**Curiosidades:**
- A ordem histórica é o inverso do que muita gente assume: o **Hibernate existiu primeiro** (nos anos 2000, como reação à complexidade do EJB 2.x Entity Beans) e foi tão bem-sucedido que virou a base conceitual sobre a qual a JPA foi padronizada. Ou seja, JPA padronizou ideias do Hibernate, não o contrário.
- Foi renomeada de **Java Persistence API** para **Jakarta Persistence** na migração de namespace `javax` → `jakarta` (ver [[Edições da Plataforma Java (Java SE, EE, Jakarta EE e ME)]]), mas a sigla "JPA" continua sendo usada informalmente no dia a dia mesmo depois da troca de nome oficial.
- Spring Data JPA **não é** uma implementação alternativa de JPA — é uma camada de conveniência (repositórios com métodos derivados por nome, ex.: `findByEmail`) construída em cima de uma implementação JPA existente (normalmente Hibernate).

## Conteúdos técnicos

JPA é conceito/teoria neste momento do curso (não é necessário escrever entidades ainda) — mas JDBC vale um exemplo mínimo por ser a base de tudo:

```java
try (Connection conn = DriverManager.getConnection(url, user, senha);
     PreparedStatement stmt = conn.prepareStatement(
             "SELECT nome FROM cliente WHERE id = ?")) {
    stmt.setLong(1, clienteId);
    try (ResultSet rs = stmt.executeQuery()) {
        if (rs.next()) {
            System.out.println(rs.getString("nome"));
        }
    }
} // try-with-resources fecha Connection, Statement e ResultSet automaticamente
```

## Referências oficiais

- [Jakarta Persistence Specification](https://jakarta.ee/specifications/persistence/)
- [JDBC — Oracle Java Tutorials](https://docs.oracle.com/javase/tutorial/jdbc/)
- [Hibernate ORM Documentation](https://hibernate.org/orm/documentation/)
