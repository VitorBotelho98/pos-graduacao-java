---
aliases:
  - Métodos e Construtores
tags:
  - java
  - fundamentos
  - orientacao-a-objetos
  - modulo_0
---

# Métodos e Construtores

## Métodos

Um método é um bloco de código nomeado, associado a uma classe, que define comportamento. A assinatura de um método — nome + tipos e ordem dos parâmetros (o tipo de retorno **não** entra na assinatura para fins de sobrecarga) — é o que o compilador usa para decidir qual método chamar.

```java
public class Calculadora {
    public int somar(int a, int b) {
        return a + b;
    }
}
```

### Parâmetros são sempre passados por valor

Como já visto em [[03 - Classes, Atributos e Objetos]], Java não tem passagem por referência. Para tipos primitivos, isso significa que o método recebe uma **cópia** do valor — alterações ao parâmetro dentro do método não escapam para o chamador:

```java
public void dobrar(int numero) {
    numero = numero * 2; // altera apenas a cópia local
}

int valor = 5;
dobrar(valor);
System.out.println(valor); // ainda 5 — o método não tem como alterar a variável do chamador
```

Para objetos, o que é copiado é a referência (o endereço) — então o método pode mutar o **estado interno** do objeto apontado, mas reatribuir o parâmetro não afeta a referência original do chamador (mesmo raciocínio detalhado na nota anterior).

### Sobrecarga (*overloading*)

Java permite múltiplos métodos com o **mesmo nome**, desde que a lista de parâmetros seja diferente em tipo, quantidade ou ordem — o compilador resolve qual variante chamar em tempo de compilação, com base nos tipos estáticos dos argumentos:

```java
public class Calculadora {
    public int somar(int a, int b) { return a + b; }
    public double somar(double a, double b) { return a + b; }
    public int somar(int a, int b, int c) { return a + b + c; }
}
```

Isso é diferente de **sobrescrita** (*override*), que acontece entre classe pai e subclasse com a **mesma assinatura** e é resolvida em runtime — o tema central de [[06 - Herança e Polimorfismo]]. Overloading é resolvido estaticamente (bind em tempo de compilação); override é resolvido dinamicamente (bind em tempo de execução, via *dynamic dispatch*). Confundir os dois é um erro comum: um método com a assinatura exata da classe pai é override; qualquer variação na lista de parâmetros é overload — mesmo que pareça, na leitura, uma tentativa de "especializar" o comportamento.

### Varargs (Java 5)

Introduzido junto com generics e enum no Java 5, `...` permite que um método aceite um número variável de argumentos, tratados internamente como um array:

```java
public int somarTodos(int... numeros) {
    int total = 0;
    for (int n : numeros) {
        total += n;
    }
    return total;
}

somarTodos();            // 0 argumentos — válido
somarTodos(1, 2, 3);      // 3 argumentos
somarTodos(new int[]{1, 2, 3}); // um array também é aceito diretamente
```

Restrição da linguagem: varargs só pode ser o **último** parâmetro da lista, e só pode haver um por método — senão o compilador não teria como determinar onde um vararg termina e o próximo parâmetro começa.

### Métodos estáticos vs. de instância

Um método `static` pertence à **classe**, não a uma instância — não pode acessar atributos de instância diretamente (não existe um `this` implícito) e é chamado via `NomeDaClasse.metodo()`, sem precisar de `new`:

```java
public class MathUtils {
    public static int dobro(int x) {
        return x * 2;
    }
}

int resultado = MathUtils.dobro(21); // sem instanciar MathUtils
```

Use `static` quando o comportamento não depende de estado de instância (funções utilitárias puras, como `Math.max`) — abusar de métodos estáticos para tudo é um antipadrão comum de quem vem de programação puramente procedural, porque joga fora os benefícios de encapsulamento e polimorfismo que motivam usar objetos em primeiro lugar.

## Construtores

Um construtor inicializa um objeto recém-alocado. Sintaticamente parece um método, mas tem duas diferenças estruturais: **não tem tipo de retorno** (nem mesmo `void`) e **o nome é obrigatoriamente igual ao da classe**.

```java
public class ContaBancaria {
    private String titular;
    private double saldo;

    public ContaBancaria(String titular, double saldoInicial) {
        this.titular = titular;
        this.saldo = saldoInicial;
    }
}
```

### Construtor padrão (*default constructor*)

Se uma classe não declara **nenhum** construtor, o compilador gera automaticamente um construtor público sem parâmetros e sem corpo (além de chamar implicitamente `super()`). No momento em que a classe declara qualquer construtor — mesmo um só, com parâmetros — esse construtor padrão implícito **deixa de existir**, e `new ContaBancaria()` sem argumentos passa a ser erro de compilação, a menos que um construtor sem parâmetros seja escrito explicitamente.

### Sobrecarga de construtores e encadeamento com `this(...)`

Assim como métodos, construtores podem ser sobrecarregados. Para evitar duplicar lógica de inicialização entre eles, um construtor pode chamar outro da mesma classe via `this(...)` — obrigatoriamente como a **primeira instrução** do corpo:

```java
public class ContaBancaria {
    private String titular;
    private double saldo;

    public ContaBancaria(String titular, double saldoInicial) {
        this.titular = titular;
        this.saldo = saldoInicial;
    }

    // Encadeia para o construtor acima, evitando repetir a atribuição de titular/saldo
    public ContaBancaria(String titular) {
        this(titular, 0.0);
    }
}
```

### Ordem de inicialização de um objeto

Quando `new ContaBancaria(...)` executa, a JVM segue uma ordem determinística (relevante para depurar comportamento inesperado quando inicializadores e construtores interagem):

1. Alocação de memória no heap para o objeto, com todos os atributos recebendo o valor padrão da tabela vista em [[03 - Classes, Atributos e Objetos]] (zero/`false`/`null`).
2. Chamada implícita ou explícita ao construtor da **superclasse** (`super(...)`, sempre a primeira instrução efetiva de qualquer construtor — se omitido, o compilador insere `super()` sem argumentos automaticamente).
3. Execução dos **inicializadores de instância**, na ordem em que aparecem no código-fonte: atribuições inline de atributos (`private double saldo = 0.0;`) e blocos de inicialização de instância (`{ ... }` soltos no corpo da classe, um recurso raro na prática, mas válido).
4. Execução do **corpo do construtor** propriamente dito.

Inicializadores e blocos `static { ... }` seguem uma linha do tempo separada e anterior a tudo isso: rodam **uma única vez**, no carregamento da classe pela JVM (não a cada `new`), o que os torna o lugar correto para inicializar estado compartilhado por todas as instâncias (ex: um cache estático, uma constante calculada em tempo de carregamento).

```java
public class Configuracao {
    private static final Map<String, String> DEFAULTS;

    static {
        // roda uma vez, quando a classe Configuracao é carregada — não a cada instância
        DEFAULTS = new HashMap<>();
        DEFAULTS.put("timeout", "30s");
    }
}
```

## Referências

- [Java Language Specification — §8.4: Method Declarations](https://docs.oracle.com/javase/specs/jls/se21/html/jls-8.html#jls-8.4)
- [Java Language Specification — §8.8: Constructor Declarations](https://docs.oracle.com/javase/specs/jls/se21/html/jls-8.html#jls-8.8)
- [Java Language Specification — §12.5: Creation of New Class Instances](https://docs.oracle.com/javase/specs/jls/se21/html/jls-12.html#jls-12.5)
- [Oracle Java Tutorials — Passing Information to a Method or a Constructor](https://docs.oracle.com/javase/tutorial/java/javaOO/arguments.html)

## Ver também

- [[03 - Classes, Atributos e Objetos]]
- [[06 - Herança e Polimorfismo]]
