---
aliases:
  - Variáveis, Operadores e Condicionais
tags:
  - java
  - fundamentos
  - sintaxe
  - modulo_0
---

# Variáveis, Operadores e Condicionais

## Variáveis e tipos

Java é **estaticamente tipada**: o compilador precisa saber o tipo de toda variável antes de gerar bytecode, e esse tipo não muda depois (ao contrário de linguagens dinamicamente tipadas como Python ou JavaScript). Isso move uma classe inteira de erros (somar `String` com `int` sem conversão, chamar método inexistente num tipo) de runtime para tempo de compilação — o custo é escrever mais texto por declaração.

A linguagem separa dois mundos de tipo, e essa separação tem implicações de performance e de semântica que valem entender:

- **Tipos primitivos** (8 no total: `byte`, `short`, `int`, `long`, `float`, `double`, `char`, `boolean`): não são objetos, vivem diretamente na stack (quando são variáveis locais) ou inline dentro do objeto que os contém, e têm tamanho fixo conhecido em tempo de compilação. Não há overhead de alocação no heap nem de ponteiro/referência — por isso um `int[]` de um milhão de elementos é muito mais compacto e rápido de percorrer que um `Integer[]` equivalente.
- **Tipos de referência** (classes, interfaces, arrays): a variável guarda uma referência (endereço) para um objeto alocado no heap, gerenciado pelo Garbage Collector. Toda `String`, todo array, toda instância de classe cai aqui.

```java
int idade = 30;              // primitivo: valor vive direto na variável
Integer idadeBoxed = 30;     // referência: autoboxing cria um objeto Integer no heap
String nome = "Vitor";       // referência: String é objeto (imutável)
```

O **autoboxing/unboxing** (conversão automática entre `int` ↔ `Integer`, introduzido no Java 5) existe para permitir que primitivos sejam usados onde a linguagem exige objetos — coleções genéricas como `List<Integer>`, por exemplo, já que generics em Java não aceitam primitivos diretamente (uma limitação de implementação, não de design: generics usam *type erasure*, e o bytecode gerado trata todo elemento como `Object`). O trade-off é sutil e pega muita gente: `==` entre dois `Integer` compara **referência**, não valor, e por causa do *cache de Integer* (valores entre -128 e 127 são reaproveitados internamente pela JVM), `Integer a = 100; Integer b = 100; a == b` dá `true`, mas com 200 dá `false`. A forma correta de comparar valores é sempre `.equals()` ou usar o primitivo `int` quando não há necessidade real de nullability.

### Declaração de tipo explícito vs. inferência com `var`

| Forma | Desde | Exemplo | Quando usar |
|---|---|---|---|
| Tipo explícito | Java 1.0 | `Map<String, List<Integer>> cache = new HashMap<>();` | Quando o tipo não é óbvio pelo lado direito, ou em campos/parâmetros/retornos (onde `var` não é permitido) |
| `var` (inferência local) | Java 10 ([JEP 286](https://openjdk.org/jeps/286)) | `var cache = new HashMap<String, List<Integer>>();` | Variáveis locais onde o tipo já é evidente pela inicialização — reduz ruído visual sem perder tipagem estática |

`var` **não é tipagem dinâmica** — é só o compilador inferindo o tipo estático a partir do lado direito da atribuição durante a compilação; o bytecode gerado é idêntico ao da forma explícita. Restrições importantes: só funciona em variáveis locais (não em campos de classe, parâmetros de método ou tipos de retorno), e exige inicialização na mesma linha (`var x;` não compila, pois não haveria como inferir o tipo).

## Operadores

Java segue a família de sintaxe C/C++ para operadores, o que facilita a transição de quem já programou nessas linguagens (ou em C#, JavaScript). As categorias relevantes:

- **Aritméticos**: `+ - * / %`. Atenção: `/` entre dois `int` faz divisão inteira (trunca, não arredonda) — `7 / 2` é `3`, não `3.5`. Para resultado fracionário é preciso que pelo menos um operando seja `float`/`double`, ou fazer cast explícito: `(double) 7 / 2`.
- **Relacionais**: `== != > < >= <=`. Para tipos de referência (objetos), `==` compara identidade (mesmo endereço de memória), não igualdade estrutural — daí a regra prática: **nunca compare `String` ou objetos com `==`, use `.equals()`**.
- **Lógicos**: `&& ||` (curto-circuito) vs `& |` (avaliação completa, também usados como operadores bit a bit). A diferença importa em performance e em correção: `if (obj != null && obj.getValor() > 0)` só chama `getValor()` se `obj != null` for verdadeiro, evitando `NullPointerException`. Trocar por `&` avaliaria os dois lados sempre, quebrando essa proteção.
- **Bit a bit e shift**: `& | ^ ~ << >> >>>`. Pouco usados em código de aplicação, mas centrais em manipulação de flags, protocolos binários e otimizações de baixo nível. `>>>` (shift à direita sem sinal) é uma peculiaridade do Java: preenche com zero independente do sinal do número, diferente de `>>` que propaga o bit de sinal.
- **Atribuição composta**: `+= -= *= /= %=` etc. Não é só açúcar sintático puro — `x += y` faz um cast implícito de volta ao tipo de `x`, enquanto `x = x + y` pode não compilar se os tipos forem incompatíveis sem cast explícito. Exemplo: `byte b = 10; b += 5;` compila; `b = b + 5;` **não compila** (o resultado de `byte + int` é `int`, e atribuir `int` a `byte` exige cast explícito).
- **Ternário**: `condicao ? valorSeVerdadeiro : valorSeFalso`. Único operador ternário da linguagem; útil para atribuições condicionais simples, mas aninhar múltiplos ternários prejudica legibilidade — prefira `if/else` ou `switch` expression quando a lógica cresce.

```java
int a = 10, b = 3;
System.out.println(a / b);        // 3 (divisão inteira)
System.out.println((double) a / b); // 3.3333333333333335 (cast promove para double)

String status = (a > b) ? "maior" : "menor ou igual"; // ternário
```

A [Java Language Specification (JLS) §15](https://docs.oracle.com/javase/specs/jls/se21/html/jls-15.html) define a tabela completa de precedência e associatividade de operadores — vale consultar quando uma expressão composta gera dúvida sobre ordem de avaliação, em vez de confiar em memória ou em parênteses "por garantia" que às vezes mascaram um entendimento errado.

## Estruturas condicionais

### `if` / `else if` / `else`

A estrutura condicional mais básica, avaliando uma expressão booleana (Java, ao contrário de C, **não aceita inteiros como condição** — `if (1)` não compila, precisa ser `if (x != 0)` ou similar). Essa exigência de tipo `boolean` explícito é uma decisão de design deliberada para evitar a classe de bugs de C onde `if (x = 1)` (atribuição por engano no lugar de `==`) compila silenciosamente.

```java
int nota = 75;
if (nota >= 90) {
    System.out.println("A");
} else if (nota >= 70) {
    System.out.println("B");
} else {
    System.out.println("C");
}
```

### `switch`: statement clássico vs. expression (Java 14)

O `switch` clássico existe desde o Java 1.0 e sofre de um problema histórico conhecido: **fall-through implícito**. Sem `break` explícito em cada `case`, a execução "cai" para o próximo `case`, o que é fonte recorrente de bugs sutis.

```java
// Forma clássica (statement) — fall-through é o padrão, break é manual
int diaSemana = 3;
String nome;
switch (diaSemana) {
    case 1:
        nome = "Segunda";
        break;
    case 2:
        nome = "Terça";
        break;
    case 3:
        nome = "Quarta";
        break;
    default:
        nome = "Inválido";
}
```

O [JEP 361](https://openjdk.org/jeps/361) (padronizado no Java 14) introduziu a **switch expression**, com sintaxe de seta (`->`) que elimina fall-through por padrão e permite usar o `switch` como expressão que retorna valor diretamente — reduzindo boilerplate e a classe de bug do `break` esquecido:

```java
// Switch expression (Java 14+) — sem fall-through, retorna valor direto
String nome = switch (diaSemana) {
    case 1 -> "Segunda";
    case 2 -> "Terça";
    case 3 -> "Quarta";
    default -> "Inválido";
};

// Múltiplos labels no mesmo case e bloco com yield quando a lógica não cabe em uma expressão
String tipo = switch (diaSemana) {
    case 1, 2, 3, 4, 5 -> "Dia útil";
    case 6, 7 -> "Fim de semana";
    default -> {
        System.out.println("Valor fora do intervalo esperado: " + diaSemana);
        yield "Desconhecido";
    }
};
```

Vale registrar por que a mudança aconteceu: não foi só estética. `switch` como **expression** obriga o compilador a verificar exaustividade (todos os casos cobertos, ou um `default`), o que pega em compilação um bug que no `switch` statement clássico só apareceria em runtime (variável não inicializada em algum branch esquecido). Essa base também é o que permitiu evoluções posteriores como o *pattern matching for switch* ([JEP 441](https://openjdk.org/jeps/441), Java 21), que estende `switch` para testar o **tipo** do valor, não só igualdade — fora do escopo deste módulo introdutório, mas útil saber que a evolução do `switch` não parou em 2020.

## Referências

- [Java Language Specification — Chapter 4: Types, Values, and Variables](https://docs.oracle.com/javase/specs/jls/se21/html/jls-4.html)
- [Java Language Specification — Chapter 15: Expressions](https://docs.oracle.com/javase/specs/jls/se21/html/jls-15.html)
- [JEP 286: Local-Variable Type Inference (`var`)](https://openjdk.org/jeps/286)
- [JEP 361: Switch Expressions](https://openjdk.org/jeps/361)

## Ver também

- [[01 - O que é java?]]
