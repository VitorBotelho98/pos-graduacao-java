---
aliases:
  - Object, toString e Comparação de Objetos
  - java.lang.Object
tags:
  - java
  - fundamentos
  - orientacao-a-objetos
  - modulo_0
---

# `Object`, `toString` e Comparação de Objetos

## `Object` como raiz da hierarquia

Toda classe em Java, direta ou indiretamente, estende `java.lang.Object` — mesmo sem escrever `extends Object` explicitamente, o compilador insere isso automaticamente. É por isso que qualquer objeto, de qualquer tipo, tem `toString()`, `equals()`, `hashCode()` disponíveis desde o primeiro instante, mesmo sem nenhuma classe própria ter sido escrita ainda.

Os métodos definidos por `Object` ([Javadoc — `java.lang.Object`](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Object.html)):

| Método | Propósito |
|---|---|
| `toString()` | Representação textual do objeto |
| `equals(Object)` | Igualdade lógica entre dois objetos |
| `hashCode()` | Código hash usado por estruturas como `HashMap`/`HashSet` |
| `getClass()` | Retorna o `Class<?>` do tipo real do objeto em runtime |
| `clone()` | Cria uma cópia do objeto (requer `Cloneable`) |
| `wait()` / `notify()` / `notifyAll()` | Coordenação entre threads via o monitor do objeto |
| `finalize()` | Chamado pelo coletor de lixo antes de descartar o objeto — **deprecated** desde o Java 9, removido do caminho recomendado por ser não-determinístico e propenso a erros; prefira `try-with-resources`/`AutoCloseable` |

## `toString()`

A implementação padrão de `Object.toString()` retorna `getClass().getName() + "@" + Integer.toHexString(hashCode())` — algo como `Pessoa@1b6d3586`, sem valor prático para depuração ou log. Sobrescrever `toString()` é a forma idiomática de dar uma representação legível a um objeto:

```java
public class Pessoa {
    private String nome;
    private int idade;

    public Pessoa(String nome, int idade) {
        this.nome = nome;
        this.idade = idade;
    }

    @Override
    public String toString() {
        return "Pessoa{nome='" + nome + "', idade=" + idade + "}";
    }
}

Pessoa p = new Pessoa("Vitor", 30);
System.out.println(p);        // chama toString() implicitamente
System.out.println("P: " + p); // concatenação também chama toString() implicitamente
```

Qualquer contexto que precise converter um objeto para `String` — `System.out.println`, concatenação com `+`, `String.valueOf(obj)`, interpolação em logs — invoca `toString()` implicitamente. Não sobrescrever fica evidente em logs como `Iniciando processamento: Pedido@4f3f5b46`, sem nenhuma informação útil.

## Comparação de objetos: `equals()`

O `equals()` herdado de `Object` compara **identidade de referência** — equivalente a `==` — porque `Object` não sabe nada sobre o significado de "igual" para um tipo específico:

```java
public boolean equals(Object obj) {
    return (this == obj); // implementação padrão de Object
}
```

Para comparar **conteúdo** (dois objetos com os mesmos valores de atributo, mesmo sendo instâncias diferentes), é preciso sobrescrever `equals()`. O contrato que a sobrescrita **deve** respeitar, conforme o [Javadoc de `Object.equals`](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Object.html):

1. **Reflexivo**: `x.equals(x)` é sempre `true`.
2. **Simétrico**: `x.equals(y)` é `true` se e somente se `y.equals(x)` também for.
3. **Transitivo**: se `x.equals(y)` e `y.equals(z)`, então `x.equals(z)`.
4. **Consistente**: chamadas repetidas retornam o mesmo resultado, desde que os atributos usados na comparação não mudem.
5. `x.equals(null)` é sempre `false`.

```java
public class Pessoa {
    private String nome;
    private int idade;

    // ... construtor omitido

    @Override
    public boolean equals(Object obj) {
        if (this == obj) return true;                 // atalho: mesma referência
        if (obj == null || getClass() != obj.getClass()) return false; // tipo diferente nunca é igual
        Pessoa outra = (Pessoa) obj;
        return idade == outra.idade && Objects.equals(nome, outra.nome);
    }
}
```

`Objects.equals(a, b)` (de `java.util.Objects`, desde o Java 7) compara com *null-safety* embutida — equivalente a `a == null ? b == null : a.equals(b)` — evitando `NullPointerException` ao comparar campos que podem ser `null`.

## `hashCode()` — o contrato acoplado a `equals()`

O contrato de `hashCode()` ([Javadoc — `Object.hashCode`](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Object.html)) exige: **se dois objetos são iguais segundo `equals()`, eles devem ter o mesmo `hashCode()`**. O inverso não é exigido — objetos diferentes podem colidir no mesmo hash, embora hashes bem distribuídos melhorem a performance de tabelas hash.

```java
@Override
public int hashCode() {
    return Objects.hash(nome, idade); // combina os hashes dos campos usados em equals()
}
```

**Sempre que `equals()` é sobrescrito, `hashCode()` deve ser sobrescrito também, usando os mesmos campos.** Quebrar esse acoplamento é uma fonte clássica de bugs sutis: um objeto pode ser considerado igual por `equals()`, mas "desaparecer" de um `HashSet`/`HashMap` porque o hash não bate — a estrutura procura no *bucket* errado e nunca encontra o elemento, mesmo ele estando logicamente presente na coleção.

| | `equals()` | `hashCode()` |
|---|---|---|
| Responde | "Estes dois objetos são logicamente iguais?" | "Em qual *bucket* de uma tabela hash este objeto deveria estar?" |
| Comportamento padrão (`Object`) | Identidade de referência (`==`) | Derivado do endereço/identidade interna do objeto |
| Quando sobrescrever | Sempre que "igual" deve significar "mesmo conteúdo", não "mesma instância" | Sempre que `equals()` for sobrescrito — mesmos campos, mesma consistência |

Records ([JEP 395](https://openjdk.org/jeps/395), Java 16) automatizam esse par: o compilador gera `equals()`, `hashCode()` e `toString()` a partir dos componentes declarados, já respeitando o acoplamento entre os dois métodos — ver [Javadoc — `java.lang.Record`](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Record.html).

```java
public record PontoXY(int x, int y) { } // equals, hashCode e toString gerados automaticamente
```

## `getClass()` e `instanceof`

`getClass()` retorna o objeto `Class<?>` correspondente ao **tipo real** do objeto em tempo de execução — diferente de `instanceof`, que verifica compatibilidade de tipo incluindo subtipos:

```java
Object o = "texto";
o.getClass() == String.class;      // true — tipo exato
o instanceof CharSequence;         // true — String é subtipo de CharSequence, mesmo não sendo o tipo exato
```

Essa diferença importa dentro de `equals()`: usar `getClass() != obj.getClass()` (tipo exato) em vez de `!(obj instanceof Pessoa)` evita que uma subclasse de `Pessoa` seja considerada igual a uma instância da classe-base, o que poderia violar simetria do contrato de `equals()` se a subclasse adicionar campos relevantes à comparação.

## O pacote `java.lang`

`java.lang` é o único pacote **importado implicitamente** em todo arquivo Java — não precisa de `import` para usar `String`, `Object`, `Math`, `System`, `Integer` e as demais classes wrapper, `Thread`, `Runnable`, `Comparable`, `Iterable`, `AutoCloseable`, e toda a hierarquia de exceções (`Throwable`, `Exception`, `RuntimeException`, `Error`). A justificativa é puramente prática: são os tipos usados em praticamente todo programa Java, então exigir `import java.lang.String` em cada arquivo seria ruído puro. Ver [Javadoc — pacote `java.lang`](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/package-summary.html) para a lista completa.

## Referências

- [Javadoc — `java.lang.Object` (Java SE 21)](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Object.html)
- [Javadoc — `java.util.Objects`](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Objects.html)
- [Javadoc — `java.lang.Record`](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Record.html)
- [Javadoc — pacote `java.lang`](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/package-summary.html)
- [JEP 395: Records](https://openjdk.org/jeps/395)
- [Oracle Java Tutorials — Object as a Superclass](https://docs.oracle.com/javase/tutorial/java/IandI/objectclass.html)

## Ver também

- [[03 - Classes, Atributos e Objetos]]
- [[08 - Strings, Imutabilidade e String Pool]]
- [[13 - Classes Wrapper e Autoboxing]]
