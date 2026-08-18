---
aliases:
  - Strings, Imutabilidade e String Pool
tags:
  - java
  - fundamentos
  - sintaxe
  - modulo_0
---

# Strings, Imutabilidade e String Pool

## Manipulação básica de Strings

`String` é a classe mais usada em qualquer programa Java, mas não é um tipo primitivo — é um objeto (`java.lang.String`), embora a linguagem dê a ela tratamento especial: literais entre aspas (`"texto"`) e o operador `+` para concatenar são sintaxe nativa, não chamadas de método explícitas.

```java
String nome = "Vitor";

nome.length();                  // 5 — quantidade de unidades de código UTF-16
nome.charAt(0);                 // 'V'
nome.substring(0, 3);           // "Vit" — [início, fim), fim exclusivo
nome.indexOf("to");             // 2, ou -1 se não encontrar
nome.toUpperCase();             // "VITOR"
nome.replace('o', '0');         // "Vit0r"
nome.strip();                   // remove espaços nas bordas (Unicode-aware, desde o Java 11)
nome.split(",");                // String[] — quebra por regex
String.join("-", "a", "b", "c"); // "a-b-c"
```

Um ponto que passa despercebido por quem vem de outras linguagens: **todo método acima retorna uma String nova — nenhum deles altera `nome`**. `nome.toUpperCase()` sem atribuir o retorno a uma variável é, na prática, um no-op (a String maiúscula é criada e imediatamente descartada). Isso não é um detalhe de implementação — é consequência direta da imutabilidade, tratada a seguir.

Desde o Java 15 ([JEP 378](https://openjdk.org/jeps/378)), *text blocks* (`"""`) resolvem o problema de strings multilinha sem concatenação nem `\n` manual:

```java
// Antes (concatenação manual)
String json = "{\n" +
              "  \"nome\": \"Vitor\"\n" +
              "}";

// Java 15+ (text block)
String jsonTextBlock = """
        {
          "nome": "Vitor"
        }""";
```

## Imutabilidade da String

Uma vez criado, o conteúdo de um objeto `String` **nunca muda**. Não existe operação que altere os caracteres internos de uma instância já existente — `substring`, `replace`, `concat`, `toUpperCase` etc. sempre alocam e retornam uma instância nova.

```java
String a = "java";
String b = a.concat("script");
System.out.println(a); // "java" — 'a' continua intacto
System.out.println(b); // "javascript" — objeto novo
```

Por que a linguagem foi desenhada assim, e não com Strings mutáveis (como `char[]` ou o `StringBuilder`)? Alguns motivos que a documentação e a JLS deixam explícitos, direta ou indiretamente:

- **Segurança**: Strings são amplamente usadas como parâmetros sensíveis — nomes de arquivo, URLs de conexão, credenciais, argumentos de classloading. Se fossem mutáveis, um código que recebe uma `String` como parâmetro não teria garantia de que o valor não muda *depois* de validado por outro trecho de código com referência à mesma instância (um ataque clássico do tipo TOCTOU — *time-of-check to time-of-use*).
- **Compartilhamento seguro (thread-safety)**: um objeto imutável pode ser compartilhado livremente entre threads sem sincronização — não existe estado mutável para gerar condição de corrida. É o motivo pelo qual `String` nunca precisou de `synchronized` em seus métodos de leitura.
- **Cache do hash code**: `String` calcula e armazena internamente o resultado de `hashCode()` na primeira chamada, porque sabe que o valor nunca vai mudar depois. Isso torna `String` como chave de `HashMap`/`HashSet` particularmente barata — o hash não precisa ser recalculado a cada lookup.
- **Viabiliza o String Pool**: só é seguro duas variáveis apontarem para o mesmo objeto `String` porque nenhuma das duas pode alterá-lo por baixo dos pés da outra. É essa garantia que sustenta a otimização descrita a seguir.

A documentação oficial resume: *"Strings are constant; their values cannot be changed after they are created (...) Because String objects are immutable they can be shared"* ([Javadoc — `java.lang.String`](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/String.html)).

## Java String Pool

O **String Pool** (ou *string constant pool*) é uma área especial da JVM onde literais de String são armazenados e reutilizados — em vez de criar um objeto novo para cada ocorrência do mesmo literal, a JVM devolve uma referência à instância já existente.

```java
String s1 = "java";
String s2 = "java";

System.out.println(s1 == s2); // true — mesma referência, veio do pool
```

Isso funciona porque, segundo a *Java Language Specification*, literais de String (e expressões constantes resolvidas em tempo de compilação) são **internados** (*interned*) automaticamente:

> *"String literals — and strings that are values of constant expressions — are interned so as to share unique instances, using the method [`String.intern`]."*
> — [JLS §3.10.5 — String Literals](https://docs.oracle.com/javase/specs/jls/se21/html/jls-3.html#jls-3.10.5)

O detalhe que costuma confundir: `"Hel" + "lo"` é uma **expressão constante** (ambos operandos são literais, resolvidos em tempo de compilação) — então é internada e vai para o pool, igual a `"Hello"`. Mas `"Hel" + variavel` é resolvida em **tempo de execução**, e o resultado é uma String nova, fora do pool, mesmo que o conteúdo final seja idêntico:

```java
String hello = "Hello";
String lo = "lo";

System.out.println(hello == "Hel" + "lo");        // true  — constante, internada em compile-time
System.out.println(hello == "Hel" + lo);          // false — concatenação em runtime, objeto novo
System.out.println(hello == ("Hel" + lo).intern()); // true — intern() força a busca/entrada no pool
```

`new String(...)` tem o mesmo efeito de "furar" o pool: sempre cria um objeto novo na heap, mesmo que o conteúdo já exista no pool.

```java
String pooled = "java";
String heap = new String("java");

System.out.println(pooled == heap);        // false — objetos diferentes
System.out.println(pooled.equals(heap));   // true  — mesmo conteúdo
System.out.println(pooled == heap.intern()); // true — intern() busca/insere no pool e retorna a referência de lá
```

Desde o Java 7 ([JDK-6962931](https://bugs.openjdk.org/browse/JDK-6962931)), o pool deixou de viver na *PermGen* e passou a residir na **heap principal**, o que o torna elegível para coleta de lixo normal — antes disso, pools muito grandes (ex.: aplicações que faziam `intern()` de forma descontrolada) podiam causar `OutOfMemoryError: PermGen space`.

## Comparação de Referência (`==`) vs. Conteúdo (`equals`)

É a armadilha mais comum de quem começa em Java: `==` em objetos compara **identidade de referência** (mesmo endereço na memória), não conteúdo. Para `String`, comparar conteúdo é sempre trabalho de `equals()` (ou `equalsIgnoreCase()`):

```java
String a = "teste";
String b = "teste";
String c = new String("teste");

a == b;          // true  — ambos vieram do pool, mesma referência
a == c;          // false — c é um objeto novo na heap
a.equals(c);      // true  — mesmo conteúdo, é isso que importa na prática
```

| Comparação | O que verifica | Quando usar |
|---|---|---|
| `==` | Se as duas variáveis apontam para **o mesmo objeto** na memória | Praticamente nunca, para `String` — só faz sentido se a intenção for testar identidade, não igualdade |
| `.equals()` | Se o **conteúdo** (sequência de caracteres) é igual | Sempre que a intenção é comparar o valor textual |

O motivo de `==` funcionar "por acidente" em exemplos didáticos como `a == b` acima é justamente o String Pool: literais idênticos colapsam para a mesma instância. Isso é uma otimização de implementação, não uma garantia da linguagem para todo par de Strings com o mesmo conteúdo — código que depende de `==` para strings vindas de `new String(...)`, leitura de arquivo, `substring`, concatenação em runtime, `Scanner`, JSON parseado, etc. quebra silenciosamente, porque nada garante que essas Strings passaram pelo pool.

## `StringBuilder` vs. Concatenação em Loop

A imutabilidade tem um custo direto de performance quando exposta ao padrão errado: **concatenar Strings dentro de um loop com `+`** recria um objeto novo a cada iteração, descartando o anterior.

```java
// Ruim: O(n²) no pior caso — cada '+=' aloca uma String inteira nova,
// copiando todo o conteúdo acumulado até então
String resultado = "";
for (int i = 0; i < 10_000; i++) {
    resultado += i; // nova String a cada iteração
}
```

A própria JLS observa que compiladores *podem* otimizar concatenação com técnicas equivalentes a um buffer mutável (["implementations may choose to perform conversion and concatenation in one step to avoid creating and then discarding an intermediate String object"](https://docs.oracle.com/javase/specs/jls/se21/html/jls-15.html#jls-15.18.1)) — mas essa otimização se aplica a uma **expressão** de concatenação isolada (`a + b + c` numa linha vira, por baixo dos panos, um único `StringBuilder`/`invokedynamic`). Ela **não** se estende através de iterações de loop: o compilador não sabe, em tempo de compilação, quantas vezes o loop vai rodar, então não pode fundir todas as iterações num único buffer — cada `resultado += i` produz seu próprio `StringBuilder` implícito, usado uma vez e descartado.

A solução é gerenciar o buffer manualmente com `StringBuilder`, uma sequência de caracteres **mutável** ([Javadoc — `java.lang.StringBuilder`](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/StringBuilder.html)) que cresce um array interno sob demanda, em vez de realocar uma String inteira a cada modificação:

```java
// Bom: um único buffer mutável reaproveitado a cada append — O(n) amortizado
StringBuilder sb = new StringBuilder();
for (int i = 0; i < 10_000; i++) {
    sb.append(i);
}
String resultado = sb.toString(); // materializa a String final uma única vez
```

| | `String` (`+=` em loop) | `StringBuilder` |
|---|---|---|
| Mutabilidade | Imutável — cada operação cria um objeto novo | Mutável — mesmo objeto cresce internamente |
| Custo em loop de N iterações | O(n²) — cada iteração copia tudo que já foi acumulado | O(n) amortizado — `append` reaproveita o buffer, realocando só quando excede a capacidade |
| Thread-safety | Sim (por ser imutável) | Não (use `StringBuffer` — API idêntica, métodos `synchronized` — se precisar de acesso concorrente) |
| Quando usar | Concatenação simples, poucas operações, fora de loops | Qualquer acumulação incremental (loops, construção condicional de texto, parsing) |

Vale registrar a evolução: até o Java 8, o compilador traduzia `a + b` para chamadas explícitas a `StringBuilder.append`. A partir do Java 9, o [JEP 280 (Indify String Concatenation)](https://openjdk.org/jeps/280) mudou essa estratégia para usar `invokedynamic` com `java.lang.invoke.StringConcatFactory`, desacoplando o bytecode gerado da estratégia de concatenação — permitindo que a JVM troque a implementação (inclusive otimizações futuras) sem exigir recompilação do código-fonte. Na prática, isso não muda como se escreve código Java, mas explica por que decompilar uma classe compilada com Java 9+ não mostra mais `StringBuilder` explícito onde antes aparecia.

## Referências

- [Javadoc — `java.lang.String` (Java SE 21)](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/String.html)
- [Javadoc — `java.lang.StringBuilder` (Java SE 21)](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/StringBuilder.html)
- [JLS §3.10.5 — String Literals](https://docs.oracle.com/javase/specs/jls/se21/html/jls-3.html#jls-3.10.5)
- [JLS §15.18.1 — String Concatenation Operator +](https://docs.oracle.com/javase/specs/jls/se21/html/jls-15.html#jls-15.18.1)
- [Oracle Java Tutorials — Strings](https://docs.oracle.com/javase/tutorial/java/data/strings.html)
- [JEP 280: Indify String Concatenation](https://openjdk.org/jeps/280)
- [JEP 378: Text Blocks](https://openjdk.org/jeps/378)

## Ver também

- [[01 - Variáveis, Operadores e Condicionais]]
- [[03 - Classes, Atributos e Objetos]]
- [[12 - Object, toString e Comparação de Objetos]]
- [[13 - Classes Wrapper e Autoboxing]]
