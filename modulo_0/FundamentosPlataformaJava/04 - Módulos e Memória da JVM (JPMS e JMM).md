---
aliases:
  - Módulos e Memória da JVM (JPMS e JMM)
tags:
  - java
  - jvm
  - fundamentos
  - modulo_0
  - concorrencia
---

# Módulos e Memória da JVM: JPMS e JMM

Duas siglas que não têm relação direta entre si — uma é sobre **organização de código** (JPMS), a outra sobre **concorrência** (JMM) — mas ambas mudam profundamente como a plataforma se comporta "por dentro", por isso agrupadas como "internals" da JVM.

## JPMS (Java Platform Module System)

Sistema de módulos introduzido no **Java 9** (Project Jigsaw, [JSR 376](https://openjdk.org/jeps/261)), uma camada acima de pacotes: um módulo declara explicitamente, em um arquivo `module-info.java`, o que **exporta** (visível para outros módulos) e o que **requer** (dependências de outros módulos). Isso impõe **encapsulamento forte** — código fora do módulo não consegue mais acessar classes internas não exportadas via reflection irrestrita, algo que antes era possível com pacotes públicos.

**Curiosidades:**
- O motivador original não era "modularizar aplicações de negócio" — era **modularizar o próprio JDK**, que até o Java 8 era um monólito gigante (`rt.jar`) com tudo dentro, mesmo que a aplicação usasse 5% dele. O JPMS permitiu quebrar o JDK em ~70 módulos e, com `jlink`, gerar runtimes customizados só com os módulos realmente usados — relevante para containers/imagens Docker menores.
- Adoção do JPMS em código de **aplicação** (não do JDK) foi bem mais lenta e controversa do que o esperado: bibliotecas antigas no classpath tradicional viram "módulos automáticos" (nome derivado do JAR, sem `module-info.java` próprio), o que funciona mas perde as garantias de encapsulamento forte — na prática, muitos projetos Spring/enterprise ainda rodam majoritariamente em classpath clássico, não modularizados, mesmo anos depois do Java 9.
- Explica por que ferramentas de reflection pesada (frameworks antigos de mock, algumas libs de serialização) precisaram de ajustes ou flags `--add-opens` para continuar funcionando em JDKs modernos — o encapsulamento forte quebrou acessos que antes eram "por baixo dos panos" via reflection.

## JMM (Java Memory Model)

Definido formalmente no **Capítulo 17 da JLS**, o JMM especifica as regras de como threads enxergam mudanças de memória feitas por outras threads: quando uma escrita de uma thread é **garantidamente visível** para outra, e em que ordem operações podem (ou não) ser reordenadas pelo compilador JIT/CPU. Sem um modelo formal, otimizações de performance (reordenação de instruções, cache de registrador) poderiam causar bugs de concorrência não determinísticos e dependentes de hardware.

**Curiosidades:**
- O modelo original de memória do Java 1.0 tinha **falhas comprovadas** — permitia otimizações que quebravam garantias que os desenvolvedores assumiam como óbvias. Foi completamente reescrito pela **JSR 133** no **Java 5 (2004)**, junto com a introdução do pacote `java.util.concurrent`, criado por Doug Lea. Praticamente todo código concorrente Java moderno (`ConcurrentHashMap`, `AtomicInteger`, `ExecutorService`) depende dessa revisão do JMM para ter garantias formais de correção.
- A relação central do JMM é o **happens-before**: se a ação A "happens-before" a ação B, então os efeitos de A são garantidamente visíveis para B. `synchronized`, `volatile` e as classes de `java.util.concurrent` são, na prática, mecanismos para **estabelecer** relações happens-before — não são "só" sobre exclusão mútua, são sobre visibilidade de memória entre threads.
- `volatile` sozinho **não substitui** `synchronized` para operações compostas (como `i++`, que é ler+incrementar+escrever, não atômico) — ele garante visibilidade, não atomicidade. Confundir os dois é uma fonte clássica de bugs de concorrência sutis.

## Conteúdos técnicos

Declaração mínima de módulo (JPMS):

```java
// module-info.java
module com.exemplo.app {
    requires java.sql;
    exports com.exemplo.app.api;
    // com.exemplo.app.internal fica encapsulado — não exportado
}
```

Efeito de `volatile` estabelecendo happens-before (JMM) — sem ele, a thread leitora poderia nunca enxergar a atualização de `pronto`, por otimização de cache de CPU/JIT:

```java
public class Sinalizacao {
    private volatile boolean pronto = false;

    void produtor() {
        // ... prepara dados ...
        pronto = true; // escrita visível imediatamente para outras threads
    }

    void consumidor() {
        while (!pronto) { /* espera */ }
        // aqui é garantido enxergar tudo que o produtor escreveu antes de "pronto = true"
    }
}
```

## Referências oficiais

- [JEP 261: Module System](https://openjdk.org/jeps/261)
- [The Java Language Specification — Chapter 17: Threads and Locks (JMM)](https://docs.oracle.com/javase/specs/jls/se21/html/jls-17.html)
- [JSR 133: Java Memory Model and Thread Specification Revision](https://jcp.org/en/jsr/detail?id=133)
