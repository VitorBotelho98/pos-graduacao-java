---
aliases:
  - Arrays e Laços de Repetição
tags:
  - java
  - fundamentos
  - sintaxe
  - modulo_0
---

# Arrays e Laços de Repetição

## Arrays

Um array em Java é uma estrutura de **tamanho fixo**, homogênea (todos os elementos do mesmo tipo) e é, ela mesma, um **objeto** — mesmo um `int[]` é alocado no heap e tem uma referência, diferente de C onde um array é essencialmente um ponteiro para memória contígua sem metadados. Essa decisão de design traz uma consequência prática direta: todo array em Java carrega seu próprio tamanho (`.length`, um campo, não um método) e é verificado em bounds em runtime — acessar um índice fora do intervalo lança `ArrayIndexOutOfBoundsException` em vez de corromper memória silenciosamente, como aconteceria em C.

```java
int[] numeros = new int[5];          // array de 5 int, inicializado com zeros
numeros[0] = 10;
numeros[4] = 50;

int[] literal = {1, 2, 3, 4, 5};     // criação com inicialização literal

String[] nomes = new String[3];      // array de referências: cada posição começa null
nomes[0] = "Ana";
```

Como o tamanho é fixo na criação, "redimensionar" um array na prática significa criar um novo array e copiar os elementos — é exatamente isso que `java.util.ArrayList` faz internamente quando cresce além da capacidade atual (dobra o array interno e copia via `Arrays.copyOf`). Entender essa mecânica ajuda a entender por que `ArrayList.add` é O(1) amortizado, não O(1) garantido em toda chamada.

### Arrays multidimensionais

Java não tem arrays multidimensionais "verdadeiros" como matrizes de C — o que existe é **array de arrays** (*jagged arrays*), o que significa que cada linha pode, em tese, ter um tamanho diferente:

```java
int[][] matriz = new int[3][3];       // matriz 3x3 retangular
matriz[0][0] = 1;
matriz[2][2] = 9;

int[][] triangular = new int[3][];    // cada linha com tamanho próprio
triangular[0] = new int[]{1};
triangular[1] = new int[]{1, 2};
triangular[2] = new int[]{1, 2, 3};
```

Isso é mais flexível que uma matriz C contígua, mas tem custo: cada `int[]` interno é um objeto separado no heap, então acessar `matriz[i][j]` é duas indireções de ponteiro, não aritmética de endereço direta — relevante para código sensível a performance com matrizes grandes (ex: processamento numérico), onde às vezes um array 1D "achatado" (`int[] flat = new int[linhas * colunas]`, acessado via `flat[i * colunas + j]`) é escolhido deliberadamente para ganhar localidade de cache.

## Laços de repetição

### `for` clássico, `while` e `do-while`

As três formas herdadas da família C, cada uma comunicando uma intenção diferente ao leitor:

```java
// for: quando o número de iterações é conhecido/contável
for (int i = 0; i < 5; i++) {
    System.out.println(i);
}

// while: quando a condição de parada não depende de um contador
while (scanner.hasNextLine()) {
    processar(scanner.nextLine());
}

// do-while: quando o corpo precisa rodar ao menos uma vez, antes de checar a condição
do {
    entrada = lerEntradaDoUsuario();
} while (!entradaValida(entrada));
```

### `for` aprimorado (*enhanced for* / foreach) — Java 5

Introduzido no Java 5 (JSR 201, junto com generics, autoboxing, enum e varargs), o *enhanced for* elimina o boilerplate de gerenciar índice manualmente quando o objetivo é só "percorrer todos os elementos":

```java
int[] numeros = {10, 20, 30};

// Antes (Java 1.0-1.4): controle manual de índice
for (int i = 0; i < numeros.length; i++) {
    System.out.println(numeros[i]);
}

// Java 5+: enhanced for — mais legível, sem risco de off-by-one
for (int n : numeros) {
    System.out.println(n);
}
```

O trade-off: o `for` clássico dá acesso ao índice (necessário para modificar o array em posições específicas, percorrer de trás para frente, ou pular elementos) e o *enhanced for* não — ele itera estritamente do início ao fim, entregando **cópias** dos elementos primitivos ou **referências** dos objetos, então reatribuir a variável do loop (`n = 99`) não altera o array original. Sob o capô, o *enhanced for* sobre qualquer `Iterable` (não só arrays) é açúcar sintático para chamar `.iterator()`, `.hasNext()` e `.next()` — é por isso que `ConcurrentModificationException` pode aparecer se a coleção for modificada estruturalmente durante um `for` desse tipo.

### Streams como alternativa funcional (Java 8)

Desde o Java 8, iteração sobre coleções e arrays ganhou uma alternativa declarativa via `java.util.stream.Stream`, parte do [JEP 126 (Project Lambda)](https://openjdk.org/jeps/126). Não substitui o `for` em todos os casos (loops com efeitos colaterais complexos ou que precisam de `break` continuam mais claros em forma imperativa), mas é preferível quando a operação é uma transformação/filtragem/agregação:

```java
int[] numeros = {1, 2, 3, 4, 5, 6};

// Imperativo: soma dos números pares
int somaImperativa = 0;
for (int n : numeros) {
    if (n % 2 == 0) {
        somaImperativa += n;
    }
}

// Declarativo com Streams: a intenção ("filtrar pares, somar") fica explícita na leitura
int somaDeclarativa = Arrays.stream(numeros)
        .filter(n -> n % 2 == 0)
        .sum();
```

A diferença central não é só sintaxe: Streams descrevem **o que** deve ser feito (filtrar, mapear, reduzir) e deixam a JVM decidir a estratégia de execução — incluindo paralelização trivial via `.parallelStream()` sem reescrever a lógica. O custo é uma curva de aprendizado maior e, em loops simples, overhead de criação de objetos intermediários (lambdas, pipeline de Stream) que o JIT nem sempre consegue eliminar completamente — para loops curtos e simples, o `for` tradicional ainda costuma ser mais direto de ler e mais previsível em performance.

### `break`, `continue` e labels

`break` interrompe o loop mais interno; `continue` pula para a próxima iteração. Quando é preciso controlar um loop **externo** de dentro de um loop aninhado, Java oferece **labels** — um recurso pouco usado no dia a dia, mas que evita variáveis de controle (`boolean encontrado = false`) só para simular um `break` de múltiplos níveis:

```java
busca:
for (int i = 0; i < matriz.length; i++) {
    for (int j = 0; j < matriz[i].length; j++) {
        if (matriz[i][j] == alvo) {
            System.out.println("Encontrado em [" + i + "][" + j + "]");
            break busca; // interrompe o loop externo, não só o interno
        }
    }
}
```

## Referências

- [Java Language Specification — Chapter 10: Arrays](https://docs.oracle.com/javase/specs/jls/se21/html/jls-10.html)
- [Java Language Specification — Chapter 14: Blocks, Statements, and Patterns](https://docs.oracle.com/javase/specs/jls/se21/html/jls-14.html)
- [java.util.stream.Stream — Javadoc](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/stream/Stream.html)
- [JEP 126: Lambda Expressions & Virtual Extension Methods (Project Lambda)](https://openjdk.org/jeps/126)

## Ver também

- [[01 - Variáveis, Operadores e Condicionais]]
- [[05 - Arrays de Objetos, Composição e Enums]]
