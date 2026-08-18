---
aliases:
  - Classes Wrapper e Autoboxing
  - Wrapper Classes
  - Autoboxing
tags:
  - java
  - fundamentos
  - modulo_0
---

# Classes Wrapper e Autoboxing

## Por que primitivos precisam de uma versão "objeto"

Java tem oito tipos primitivos (`byte`, `short`, `int`, `long`, `float`, `double`, `char`, `boolean`) que **não são objetos** — não têm métodos, não podem ser `null`, e vivem diretamente na pilha ou embutidos dentro de outro objeto, sem overhead de cabeçalho de objeto. Essa é uma decisão de performance deliberada da linguagem.

O problema aparece quando uma API precisa tratar valores de forma genérica — coleções (`List<T>`, `Map<K,V>`), generics em geral, ou qualquer lugar que exija um `Object`. Generics em Java são implementados via *type erasure* e nunca aceitam primitivos como argumento de tipo (`List<int>` não compila). Para isso, cada primitivo tem uma **classe wrapper** correspondente em `java.lang`, que "embrulha" o valor primitivo num objeto:

| Primitivo | Classe wrapper |
|---|---|
| `byte` | `Byte` |
| `short` | `Short` |
| `int` | `Integer` |
| `long` | `Long` |
| `float` | `Float` |
| `double` | `Double` |
| `char` | `Character` |
| `boolean` | `Boolean` |

```java
List<Integer> numeros = new ArrayList<>(); // não existe List<int>
numeros.add(10); // int convertido para Integer automaticamente
```

## Autoboxing e unboxing

Antes do Java 5, converter entre primitivo e wrapper exigia código explícito (`Integer.valueOf(10)`, `i.intValue()`). Desde o Java 5, o compilador insere essa conversão automaticamente — **autoboxing** (primitivo → wrapper) e **unboxing** (wrapper → primitivo):

```java
int primitivo = 10;
Integer objeto = primitivo;     // autoboxing: compilador insere Integer.valueOf(primitivo)
int devolta = objeto;           // unboxing: compilador insere objeto.intValue()

List<Integer> lista = new ArrayList<>();
lista.add(5);                   // autoboxing implícito
int x = lista.get(0);           // unboxing implícito
```

É açúcar sintático — o bytecode gerado é equivalente a chamar `valueOf`/`xxxValue()` manualmente. Isso importa para entender os dois problemas práticos a seguir.

## Cache de `Integer` (e outros wrappers) — `==` vs. `equals()`

`Integer.valueOf(int)`, usado internamente pelo autoboxing, **cacheia e reutiliza instâncias para valores entre -128 e 127** (`IntegerCache`), de forma equivalente ao String Pool descrito em [[08 - Strings, Imutabilidade e String Pool]]. Segundo o [Javadoc de `Integer.valueOf`](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Integer.html): *"this method will always cache values in the range -128 to 127, inclusive, and may cache other values outside of this range"*.

```java
Integer a = 100;
Integer b = 100;
System.out.println(a == b); // true — ambos vieram do cache, mesma instância

Integer c = 200;
Integer d = 200;
System.out.println(c == d); // false — fora do intervalo cacheado, instâncias diferentes
```

Esse comportamento é a armadilha clássica de wrapper: código que "funciona" em testes com valores pequenos (dentro do cache) e quebra silenciosamente em produção com valores maiores. A regra prática é a mesma de `String`: **`==` compara referência, `equals()` compara valor — para wrappers, use sempre `equals()`** (ou compare os primitivos desembrulhados, se ambos os lados puderem sofrer unboxing sem risco de `NullPointerException`):

```java
c.equals(d);      // true — compara o valor int encapsulado, não a referência
c.intValue() == d.intValue(); // true — compara primitivos diretamente, sem ambiguidade
```

## `NullPointerException` no unboxing

Um wrapper pode ser `null` (é um objeto); um primitivo nunca pode. Quando o compilador insere unboxing automático sobre uma referência `null`, o resultado é `NullPointerException` em runtime — silenciosamente, sem nenhum aviso de compilação:

```java
Integer valor = null;
int primitivo = valor; // compila normalmente; lança NullPointerException em runtime (unboxing de null)
```

Isso aparece com frequência em expressões condicionais e em campos de wrapper não inicializados (atributos de instância de tipo wrapper começam como `null` por padrão, diferente de primitivos que começam com `0`/`false`):

```java
Map<String, Integer> contagem = new HashMap<>();
int total = contagem.get("chave-inexistente") + 1; // NPE: get() retorna null, unboxing falha
```

## Custo de performance em loops

Autoboxing/unboxing repetido dentro de um loop cria um objeto wrapper a cada iteração (fora do intervalo cacheado), com custo de alocação que um primitivo não teria:

```java
// Ruim: cada iteração faz autoboxing de i (int) para somar num Long, criando objetos descartáveis
Long soma = 0L;
for (int i = 0; i < 1_000_000; i++) {
    soma += i; // unboxing de 'soma', soma primitiva, autoboxing de volta para Long — a cada iteração
}

// Bom: acumulador primitivo, sem boxing algum
long somaPrimitiva = 0L;
for (int i = 0; i < 1_000_000; i++) {
    somaPrimitiva += i;
}
```

A regra prática: use wrappers quando a API exigir um objeto (generics, coleções, valores que podem ser `null` para representar "ausente"); use primitivos em qualquer acumulação ou cálculo intensivo onde não há necessidade de tratar o valor como objeto.

## Parsing: `parseXxx` vs. `valueOf`

Ambos convertem `String` para número, mas retornam tipos diferentes — distinção que costuma confundir por parecerem intercambiáveis:

```java
int primitivo = Integer.parseInt("42");   // retorna int primitivo
Integer objeto = Integer.valueOf("42");   // retorna Integer (usa o cache internamente, se aplicável)
```

`parseInt` é preferível quando só o valor primitivo é necessário — evita criar um objeto wrapper desnecessário quando o resultado será usado como `int` de qualquer forma.

## `Character` — um caso especial

Diferente dos wrappers numéricos, `Character` embrulha uma única unidade de código UTF-16 (`char`) e expõe métodos utilitários estáticos de classificação (`Character.isDigit`, `Character.isLetter`, `Character.isWhitespace`, `Character.toUpperCase`) que operam diretamente sobre `char` primitivos, sem exigir boxing:

```java
char c = 'A';
Character.isLetter(c); // true — método estático, recebe char primitivo diretamente
```

## Referências

- [Javadoc — `java.lang.Integer` (Java SE 21)](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Integer.html)
- [Javadoc — `java.lang.Character`](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Character.html)
- [Java Language Specification — §5.1.7 Boxing Conversion](https://docs.oracle.com/javase/specs/jls/se21/html/jls-5.html#jls-5.1.7)
- [Java Language Specification — §5.1.8 Unboxing Conversion](https://docs.oracle.com/javase/specs/jls/se21/html/jls-5.html#jls-5.1.8)
- [Oracle Java Tutorials — Autoboxing and Unboxing](https://docs.oracle.com/javase/tutorial/java/data/autoboxing.html)

## Ver também

- [[08 - Strings, Imutabilidade e String Pool]]
- [[12 - Object, toString e Comparação de Objetos]]
