---
aliases:
  - APIs Empresariais Java (CDI, EJB, JMS, JTA e JNDI)
tags:
  - java
  - fundamentos
  - modulo_0
  - jakarta-ee
---

# APIs Empresariais Java: CDI, EJB, JMS, JTA e JNDI

Cinco especificações Jakarta EE que resolvem problemas clássicos de aplicações corporativas: injeção de dependência (CDI), componentes de negócio gerenciados pelo servidor (EJB), mensageria assíncrona (JMS), transações que cruzam múltiplos recursos (JTA) e localização de recursos registrados no ambiente (JNDI). São conceitos essenciais para entender arquitetura enterprise Java — inclusive porque explicam de onde vieram ideias que hoje parecem "do Spring", mas na verdade são padrão da plataforma.

## CDI (Contexts and Dependency Injection)

A especificação padrão do Jakarta EE para **injeção de dependência** e gerenciamento de ciclo de vida de objetos (`@Inject`, `@Produces`, escopos como `@RequestScoped`, `@ApplicationScoped`).

**Curiosidade:** a ordem cronológica de novo é o inverso do que parece — o **Spring Framework** (2003) popularizou DI em Java **anos antes** do CDI (formalizado em 2009, JSR 299). CDI nasceu como a resposta "oficial" da plataforma a um padrão que o Spring já havia validado no mercado. Hoje as duas abordagens coexistem: em projeto Spring usa-se `@Autowired`/injeção do Spring; em servidor Jakarta EE "puro" (WildFly, Payara), usa-se CDI nativamente.

## EJB (Enterprise JavaBeans)

Componentes de negócio gerenciados pelo servidor de aplicação, com serviços de infraestrutura (transação, segurança, concorrência, pool de instâncias) fornecidos automaticamente pelo container. Dois tipos principais: **Session Beans** (lógica de negócio invocável, `@Stateless`/`@Stateful`) e **Message-Driven Beans** (consomem mensagens JMS de forma assíncrona).

**Curiosidade:** o **EJB 2.x** (início dos anos 2000) ficou famoso pela complexidade excessiva — exigia múltiplas interfaces obrigatórias e XML de configuração extenso para até a tarefa mais simples, o que gerou uma onda de rejeição ao "Java corporativo pesado" e impulsionou a popularidade do Spring como alternativa mais leve. O **EJB 3.0** (2006) foi uma reescrita quase completa da API, adotando anotações no lugar de XML e simplificando drasticamente — mas o dano de reputação já estava feito, e hoje EJB é bem menos comum em projetos novos do que CDI + Spring.

## JMS (Java Message Service)

API padrão Java para **mensageria assíncrona**: publicar/consumir mensagens em filas (ponto-a-ponto) ou tópicos (publish/subscribe), desacoplando produtor e consumidor no tempo. Implementada por brokers como ActiveMQ e IBM MQ.

**Curiosidade:** JMS é uma API **específica de Java** — diferente de protocolos de mensageria agnósticos de linguagem como **AMQP** (usado pelo RabbitMQ) ou o próprio **Kafka** (que nem segue JMS). Isso importa na prática: uma aplicação Java pode falar com um broker Kafka, mas não via JMS — usa o client nativo do Kafka, porque Kafka não implementa a especificação JMS.

## JTA (Java Transaction API)

Padroniza **transações distribuídas**: quando uma única operação de negócio precisa ser atômica através de múltiplos recursos (por exemplo, gravar em um banco de dados **e** enviar uma mensagem JMS, com garantia de que ambos aconteçam ou nenhum aconteça). Usa o protocolo de **two-phase commit (2PC)** coordenado por um Transaction Manager.

**Curiosidade:** é diferente de uma transação JDBC comum (`conn.setAutoCommit(false)`), que só cobre **um único** banco de dados. JTA entra em cena especificamente quando a atomicidade precisa cruzar mais de um recurso transacional — cenário bem mais raro no dia a dia do que transação local, mas crítico em sistemas financeiros/integrações críticas.

## JNDI (Java Naming and Directory Interface)

API para **localizar recursos por nome** registrados no ambiente do servidor de aplicação — tipicamente `DataSource`s de banco, filas JMS, ou outros EJBs — sem a aplicação precisar conhecer detalhes de configuração (URL de conexão, credenciais) diretamente no código.

**Curiosidade:** o padrão clássico de lookup `java:comp/env/jdbc/MinhaDataSource` ainda aparece em aplicações legadas rodando em servidores de aplicação tradicionais. Em stacks modernas baseadas em Spring Boot com servidor embutido, esse papel é normalmente substituído por configuração via `application.properties`/variáveis de ambiente — mas o conceito (indireção entre "nome lógico do recurso" e "configuração real") continua o mesmo, só migrou de mecanismo.

## Conteúdos técnicos

Estas cinco APIs são, neste momento, conceito de arquitetura — não é necessário escrever código para elas agora. Vale só reconhecer a assinatura visual de cada uma quando aparecer em código legado ou em servidor Jakarta EE puro:

```java
@Stateless                          // EJB — Session Bean
public class PedidoService {

    @Inject                         // CDI — injeção de dependência
    private ClienteRepository clienteRepo;

    @Resource(lookup = "jms/PedidosQueue")  // JNDI — lookup de recurso
    private Queue filaPedidos;

    @TransactionAttribute(TransactionAttributeType.REQUIRED)  // JTA
    public void processar(Pedido pedido) { /* ... */ }
}
```

## Referências oficiais

- [Jakarta Contexts and Dependency Injection Specification](https://jakarta.ee/specifications/cdi/)
- [Jakarta Enterprise Beans Specification](https://jakarta.ee/specifications/enterprise-beans/)
- [Jakarta Messaging Specification](https://jakarta.ee/specifications/messaging/)
- [Jakarta Transactions Specification](https://jakarta.ee/specifications/transactions/)
- [Java Naming and Directory Interface — Oracle](https://docs.oracle.com/javase/jndi/)
