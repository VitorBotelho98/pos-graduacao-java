---
aliases:
  - Pacotes, Modificadores de Acesso, Getters e Setters
tags:
  - java
  - fundamentos
  - orientacao-a-objetos
  - modulo_0
---

# Pacotes, Modificadores de Acesso, Getters e Setters

## Pacotes

Um pacote (`package`) agrupa classes relacionadas e cria um namespace — evita colisão de nomes entre classes de mesmo nome em bibliotecas diferentes (`com.empresa.modelo.Pedido` e `com.outraempresa.api.Pedido` coexistem sem conflito) e organiza o código por responsabilidade. Diferente de um simples agrupamento lógico, o pacote em Java tem uma exigência estrutural forte: **a declaração `package` no topo do arquivo precisa corresponder exatamente ao caminho de diretórios do arquivo `.java`**.

```java
// Arquivo: com/empresa/modelo/Pedido.java
package com.empresa.modelo;

public class Pedido {
    // ...
}
```

Uma classe sem declaração `package` explícita cai no chamado **default package** (pacote sem nome) — tecnicamente válido, mas evitado em qualquer projeto real: classes no default package não podem ser importadas por classes que pertencem a um pacote nomeado, o que praticamente inviabiliza reuso além de scripts descartáveis.

### `import`: acesso qualificado vs. simples

Para usar uma classe de outro pacote, é preciso `import` (ou referenciá-la pelo nome totalmente qualificado toda vez, o que raramente compensa):

```java
import java.util.List;
import java.util.ArrayList;
// import java.util.*; // import "on-demand": importa todas as classes públicas do pacote

List<String> nomes = new ArrayList<>();
```

O `import java.util.*` (chamado *on-demand import*) é desencorajado em código de produção pela maioria dos guias de estilo (incluindo o [Google Java Style Guide](https://google.github.io/styleguide/javaguide.html#s3.3.1-wildcard-imports)) porque obscurece de onde cada classe realmente vem e pode gerar ambiguidade silenciosa se dois pacotes importados por `*` tiverem uma classe de mesmo nome. Classes do pacote `java.lang` (`String`, `Object`, `Integer`, `System`) são a única exceção — são importadas implicitamente em todo arquivo Java, sem necessidade de `import`.

Pacotes são a base sobre a qual o **Java Platform Module System** (JPMS, desde o Java 9) constrói encapsulamento mais forte via `module-info.java` — controlando quais pacotes um módulo exporta para fora, além da visibilidade de classe individual descrita abaixo. Esse tema é aprofundado em [[04 - Módulos e Memória da JVM (JPMS e JMM)]].

## Modificadores de acesso

Java define quatro níveis de visibilidade, aplicáveis a classes, atributos, métodos e construtores — do mais restrito ao mais aberto:

| Modificador | Mesma classe | Mesmo pacote | Subclasse (outro pacote) | Qualquer lugar |
|---|:---:|:---:|:---:|:---:|
| `private` | ✅ | ❌ | ❌ | ❌ |
| *(default, sem modificador)* | ✅ | ✅ | ❌ | ❌ |
| `protected` | ✅ | ✅ | ✅ | ❌ |
| `public` | ✅ | ✅ | ✅ | ✅ |

- **`private`**: visível só dentro da própria classe. É o padrão recomendado para atributos de instância (ver encapsulamento abaixo) e para métodos auxiliares que são detalhe de implementação, não parte do contrato público da classe.
- **default / *package-private***: quando nenhum modificador é escrito, a visibilidade é o pacote inteiro. Menos usado explicitamente como escolha deliberada, mas é o padrão real de qualquer membro sem modificador — vale saber que "esquecer" o modificador não significa `public`, significa isso.
- **`protected`**: visível no pacote e para subclasses em qualquer pacote. Existe especificamente para o caso de herança: dá à subclasse acesso a detalhes que o mundo externo não deveria ter, sem abrir mão de encapsulamento completo.
- **`public`**: sem restrição — parte da API pública da classe/pacote/módulo.

Regra prática que orienta a escolha: comece com o modificador **mais restritivo possível** (`private`) e só amplie quando houver uma necessidade concreta de acesso externo. Essa disciplina (conhecida como *princípio do menor privilégio* aplicado a design de API) reduz a superfície de acoplamento entre classes — quanto menos exposto, mais livre a classe fica para mudar sua implementação interna sem quebrar quem a usa.

## Encapsulamento, Getters e Setters

Encapsulamento é o pilar de OO que motiva declarar atributos como `private` e expor acesso controlado via métodos públicos — os chamados *getters* (leitura) e *setters* (escrita), seguindo a convenção de nomenclatura **JavaBeans** (`getX()`/`isX()` para booleanos, `setX(valor)`).

```java
public class ContaBancaria {
    private double saldo; // private: ninguém de fora mexe direto no estado interno

    public double getSaldo() {
        return saldo;
    }

    public void depositar(double valor) {
        if (valor <= 0) {
            throw new IllegalArgumentException("Depósito deve ser positivo");
        }
        this.saldo += valor;
    }

    // Note a ausência deliberada de um setSaldo(double) público:
    // alterar o saldo só é permitido através de operações de domínio (depositar, sacar),
    // nunca por atribuição direta arbitrária — é isso que "encapsular" significa na prática.
}
```

O ponto central que costuma passar despercebido: **getters/setters não são "encapsulamento" por si só** — um par `getX()`/`setX()` que só repassa o valor sem nenhuma regra (o chamado *anemic getter/setter*, sinônimo funcional de expor o atributo como `public`) não protege invariante nenhuma, só adiciona indireção sintática. O valor real do encapsulamento aparece quando o setter (ou um método de domínio mais expressivo, como `depositar` acima) **valida ou transforma** a entrada antes de alterar o estado — garantindo que o objeto nunca fique em um estado inválido, não importa de onde a chamada venha. Nomear o método pela ação de domínio (`depositar`, `sacar`) em vez de por um `setSaldo` genérico também comunica intenção com mais clareza — uma prática associada ao *Tell, Don't Ask principle*.

### Por que não simplesmente deixar o atributo `public`?

Um atributo público permite qualquer atribuição, a qualquer momento, sem chance de o objeto reagir ou validar (`conta.saldo = -500;` compilaria e executaria sem erro). Além disso, expor o atributo diretamente **acopla o código cliente à representação interna** — se amanhã `saldo` precisar virar um objeto `BigDecimal` em vez de `double` (comum em sistemas financeiros, para evitar erros de arredondamento de ponto flutuante), toda a base de código que acessa `conta.saldo` diretamente quebra. Com um getter, só a implementação interna do getter muda; a API pública (`getSaldo()`) permanece estável.

### Registros (`record`) como alternativa moderna para dados imutáveis

Vale uma nota de fechamento sobre onde a linguagem foi depois: para o caso comum de uma classe que é **apenas** um portador de dados imutáveis (sem lógica de validação complexa nem necessidade de setters), o [JEP 395](https://openjdk.org/jeps/395) introduziu `record` no Java 16, gerando automaticamente construtor, getters (sem prefixo `get`, só o nome do campo), `equals()`, `hashCode()` e `toString()`:

```java
// Equivalente, em intenção, a uma classe com atributos private final, getters,
// equals/hashCode/toString e sem setters — mas sem escrever nada disso manualmente
public record Coordenada(double latitude, double longitude) { }

Coordenada c = new Coordenada(-23.55, -46.63);
c.latitude(); // getter gerado automaticamente, sem prefixo "get"
```

`record` não substitui classes tradicionais com getters/setters explícitos quando há lógica de validação não trivial, mutabilidade necessária, ou herança de classe (records não podem estender outras classes) — mas é a escolha correta e idiomática, desde o Java 16, para o caso de um objeto de valor imutável simples.

## Referências

- [Java Language Specification — Chapter 7: Packages and Modules](https://docs.oracle.com/javase/specs/jls/se21/html/jls-7.html)
- [Java Language Specification — §6.6: Access Control](https://docs.oracle.com/javase/specs/jls/se21/html/jls-6.html#jls-6.6)
- [Oracle Java Tutorials — Controlling Access to Members of a Class](https://docs.oracle.com/javase/tutorial/java/javaOO/accesscontrol.html)
- [JEP 395: Records](https://openjdk.org/jeps/395)
- [Google Java Style Guide — §3.3.1 (No wildcard imports)](https://google.github.io/styleguide/javaguide.html#s3.3.1-wildcard-imports)

## Ver também

- [[04 - Módulos e Memória da JVM (JPMS e JMM)]]
- [[03 - Classes, Atributos e Objetos]]
- [[06 - Herança e Polimorfismo]]
