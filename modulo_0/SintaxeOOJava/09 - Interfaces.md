---
aliases:
  - Interfaces
  - Interface
tags:
  - java
  - fundamentos
  - orientacao-a-objetos
  - modulo_0
---

# Interfaces

## O que é uma interface

Uma interface define um **contrato de comportamento**: um conjunto de métodos que qualquer classe implementadora se compromete a fornecer, sem (necessariamente) dizer como. É a ferramenta de Java para expressar polimorfismo por **tipo**, não por herança de implementação.

```java
public interface Pagavel {
    double calcularValor(); // implicitamente public abstract

    double TAXA_PADRAO = 0.05; // implicitamente public static final
}

public class Boleto implements Pagavel {
    private double valorBase;

    public Boleto(double valorBase) {
        this.valorBase = valorBase;
    }

    @Override
    public double calcularValor() {
        return valorBase + (valorBase * TAXA_PADRAO);
    }
}
```

Dois detalhes de sintaxe que a linguagem aplica silenciosamente e valem a pena tornar explícitos:

- Todo método sem corpo declarado numa interface é implicitamente `public abstract` — escrever esses modificadores é redundante e desencorajado pela própria especificação.
- Todo campo declarado numa interface é implicitamente `public static final` — ou seja, uma **constante** compartilhada por todos, não um atributo de instância. Não existe "campo de instância" em interface.

Uma classe pode implementar **múltiplas interfaces** (`implements A, B, C`), o que não é permitido com `extends` de classes — ver [[06 - Herança e Polimorfismo]] para a discussão de por que Java restringe herança múltipla de classes mas permite múltipla implementação de interfaces (o problema do diamante).

## Evolução: de contrato puro a comportamento compartilhado

### Antes do Java 8: só assinaturas

Até o Java 7, uma interface só podia declarar assinaturas de método — zero implementação. Isso trazia um problema prático de evolução de API: adicionar um método novo a uma interface publicada quebrava **todas** as classes que já a implementavam (elas passavam a não compilar, por não implementar o método novo).

### Java 8 — métodos `default` e `static` ([JEP 126](https://openjdk.org/jeps/126))

Métodos `default` trazem corpo de implementação para dentro da interface, servindo como implementação-padrão herdada por quem não sobrescreve:

```java
public interface Pagavel {
    double calcularValor();

    // default: implementação padrão, não obriga quem implementa a reescrevê-la
    default String descricaoFormatada() {
        return "Valor a pagar: R$ " + calcularValor();
    }

    // static: pertence à interface, não a uma instância — chamado como Pagavel.metodo()
    static Pagavel comTaxaFixa(double valor) {
        return () -> valor; // implementação via lambda (interface funcional implícita)
    }
}
```

O motivo declarado pela própria JEP foi viabilizar **expressões lambda e evolução de interface**: a API de Coleções precisava adicionar métodos como `forEach` e `stream()` a `Collection`/`List` sem quebrar todas as implementações existentes no mundo. Um método `default` resolve isso — implementações antigas continuam compilando, herdando o comportamento-padrão, e só precisam sobrescrever se quiserem um comportamento diferente.

### Java 9 — métodos `private` ([JEP 213](https://bugs.openjdk.org/browse/JDK-8071453), consolidado na JLS)

Métodos `default` frequentemente duplicavam lógica interna entre si. Java 9 permitiu métodos `private` (com ou sem `static`) em interfaces — código auxiliar interno, não exposto a quem implementa:

```java
public interface Relatorio {
    default String cabecalho() {
        return formatarLinha("RELATÓRIO");
    }

    default String rodape() {
        return formatarLinha("FIM");
    }

    // método privado: reuso entre defaults, sem virar parte do contrato público
    private String formatarLinha(String texto) {
        return "=== " + texto + " ===";
    }
}
```

Segundo a [JLS §9.4](https://docs.oracle.com/javase/specs/jls/se21/html/jls-9.html#jls-9.4): um método `private` de interface não pode ser `abstract` nem `default`, mas pode ser `static`; e interfaces **não herdam** métodos `private` ou `static` de superinterfaces — eles não fazem parte do contrato visível para quem implementa ou estende.

| Modificador | Tem corpo? | Quem pode chamar | Herdado por subinterface/implementação? |
|---|---|---|---|
| (nenhum) → `abstract` implícito | Não | — (implementação obrigatória) | Sim, como obrigação de contrato |
| `default` | Sim | Instâncias da interface/implementação | Sim, como implementação herdada |
| `static` | Sim | Só via `NomeDaInterface.metodo()` | Não |
| `private` | Sim | Só outros métodos da própria interface | Não |

## Interfaces funcionais e lambdas

Uma **interface funcional** é uma interface com exatamente um método abstrato (métodos `default`/`static` não contam). É o tipo-alvo de uma expressão lambda ou method reference, formalizado desde o Java 8 junto com o pacote `java.util.function` ([Javadoc — `java.util.function`](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/function/package-summary.html)).

```java
@FunctionalInterface
public interface Validador<T> {
    boolean validar(T valor);
}

Validador<String> naoVazio = s -> s != null && !s.isBlank(); // lambda implementa o único método abstrato
```

A anotação `@FunctionalInterface` não é obrigatória, mas documenta a intenção e faz o compilador barrar, em tempo de compilação, a adição futura de um segundo método abstrato que quebraria a compatibilidade com lambda.

## Interfaces `sealed` (Java 17 — [JEP 409](https://openjdk.org/jeps/409))

Por padrão, qualquer classe pode implementar qualquer interface pública. `sealed` restringe explicitamente **quem** pode implementar, listando os tipos permitidos com `permits`:

```java
public sealed interface FormaPagamento permits Boleto, Pix, CartaoCredito {}

public final class Boleto implements FormaPagamento { }
public final class Pix implements FormaPagamento { }
public final class CartaoCredito implements FormaPagamento { }
```

Isso é útil combinado com `switch` de pattern matching ([JEP 441](https://openjdk.org/jeps/441)): o compilador sabe, em tempo de compilação, o conjunto **fechado** de implementações possíveis, e pode verificar exaustividade de um `switch` sem exigir um `default` — algo impossível com interfaces abertas, onde qualquer código externo poderia adicionar uma implementação nova a qualquer momento.

## Referências

- [Java Language Specification — Chapter 9: Interfaces](https://docs.oracle.com/javase/specs/jls/se21/html/jls-9.html)
- [Oracle Java Tutorials — Interfaces](https://docs.oracle.com/javase/tutorial/java/IandI/createinterface.html)
- [JEP 126: Lambda Expressions & Virtual Extension Methods](https://openjdk.org/jeps/126)
- [JEP 409: Sealed Classes](https://openjdk.org/jeps/409)
- [JEP 441: Pattern Matching for switch](https://openjdk.org/jeps/441)
- [Javadoc — `java.util.function`](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/function/package-summary.html)

## Ver também

- [[06 - Herança e Polimorfismo]]
- [[10 - Classes Abstratas e Static]]
- [[11 - Interface vs Classe Abstrata]]
