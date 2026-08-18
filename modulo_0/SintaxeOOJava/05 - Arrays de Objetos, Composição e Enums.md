---
aliases:
  - Arrays de Objetos, Composição e Enums
tags:
  - java
  - fundamentos
  - orientacao-a-objetos
  - modulo_0
---

# Arrays de Objetos, Composição e Enums

## Arrays de objetos

Um array de tipo de referência (`Produto[]`, `String[]`) segue a mesma mecânica vista em [[02 - Arrays e Laços de Repetição]] — tamanho fixo, verificação de bounds em runtime —, mas com uma diferença importante em relação a um array de primitivos: cada posição do array **não contém o objeto**, contém uma **referência** para um objeto (ou `null`). Criar o array não cria os objetos.

```java
Produto[] produtos = new Produto[3]; // array de 3 referências, todas null
System.out.println(produtos[0]); // null — nenhum Produto foi instanciado ainda

produtos[0] = new Produto("Teclado", 250.0);
produtos[1] = new Produto("Mouse", 80.0);
// produtos[2] continua null

for (Produto p : produtos) {
    if (p != null) { // checagem necessária, senão NullPointerException na posição 2
        System.out.println(p.getNome());
    }
}
```

Esse é um erro comum de quem vem de linguagens onde arrays de objetos vêm "populados por padrão": em Java, `new Produto[3]` aloca só o array (três slots de referência), não três objetos `Produto`. Cada posição precisa ser explicitamente inicializada com `new` antes do uso, ou o acesso a métodos/atributos nela lança `NullPointerException`.

## Composição

Composição é a relação **"tem um"** (*has-a*) entre classes: um objeto guarda, como atributo, uma referência para outro objeto, delegando a ele parte da responsabilidade. É a alternativa mais flexível à herança (relação **"é um"**, *is-a*, tema de [[06 - Herança e Polimorfismo]]) para reutilizar comportamento entre classes.

```java
public class Motor {
    private int potenciaCv;

    public Motor(int potenciaCv) {
        this.potenciaCv = potenciaCv;
    }

    public void ligar() {
        System.out.println("Motor de " + potenciaCv + "cv ligado");
    }
}

public class Carro {
    // Composição: Carro "tem um" Motor — não é um Motor, mas depende de um
    private final Motor motor;
    private String modelo;

    public Carro(String modelo, int potenciaCvDoMotor) {
        this.modelo = modelo;
        this.motor = new Motor(potenciaCvDoMotor);
    }

    public void ligar() {
        motor.ligar(); // delega a responsabilidade ao objeto composto
    }
}
```

A regra prática (princípio "favoreça composição em vez de herança", popularizado pelo livro *Design Patterns* da Gang of Four) existe porque herança cria um acoplamento muito mais rígido: uma subclasse depende dos detalhes internos de implementação da superclasse, e mudanças na superclasse podem quebrar subclasses de formas não óbvias (o "problema da classe base frágil"). Composição delega através de uma interface pública bem definida, o que é mais fácil de trocar, testar isoladamente (mockar `Motor` é trivial; "mockar" uma superclasse é estruturalmente mais difícil) e evita hierarquias profundas e difíceis de raciocinar. Herança ainda tem seu lugar quando a relação **é**, de fato, uma especialização genuína de tipo (não só reuso de código) — o critério para decidir qual usar é aprofundado na próxima nota.

## Enums

Antes do Java 5, um conjunto fixo de constantes relacionadas (dias da semana, status de pedido, naipes de carta) era tipicamente modelado como constantes `int`/`String` soltas — um padrão frágil porque o compilador não impede passar qualquer `int` onde um "dia da semana" era esperado, nem garante que os valores usados batem com os declarados:

```java
// Padrão pré-Java 5 (ainda comum em código legado) — sem segurança de tipo
public static final int SEGUNDA = 0;
public static final int TERCA = 1;
// nada impede diaSemana(42) ou diaSemana(SEGUNDA + TERCA) de compilar
```

O [JSR 201](https://jcp.org/en/jsr/detail?id=201) introduziu `enum` no Java 5 como um **tipo próprio**, com segurança de tipo garantida pelo compilador: uma variável do tipo `DiaDaSemana` só pode receber um dos valores declarados, nunca um `int` arbitrário.

```java
public enum DiaDaSemana {
    SEGUNDA, TERCA, QUARTA, QUINTA, SEXTA, SABADO, DOMINGO
}

DiaDaSemana hoje = DiaDaSemana.QUARTA;

// Integra nativamente com switch expression (ver 01 - Variáveis, Operadores e Condicionais)
String tipo = switch (hoje) {
    case SABADO, DOMINGO -> "Fim de semana";
    default -> "Dia útil";
};
```

### Enum não é só uma lista de nomes — é uma classe

A diferença que mais surpreende quem vem de linguagens onde enum é só um inteiro nomeado (C, por exemplo): em Java, cada constante de um `enum` é, sob o capô, uma **instância única** (singleton) da classe enum, e a classe pode ter atributos, construtores e métodos — inclusive corpo específico por constante:

```java
public enum StatusPedido {
    CRIADO(1, "Pedido criado"),
    PAGO(2, "Pagamento confirmado"),
    ENVIADO(3, "Pedido a caminho"),
    ENTREGUE(4, "Pedido entregue");

    private final int codigo;
    private final String descricao;

    // Construtor de enum é implicitamente private — não é possível instanciar de fora
    StatusPedido(int codigo, String descricao) {
        this.codigo = codigo;
        this.descricao = descricao;
    }

    public int getCodigo() {
        return codigo;
    }

    public String getDescricao() {
        return descricao;
    }
}

StatusPedido status = StatusPedido.PAGO;
System.out.println(status.getDescricao()); // "Pagamento confirmado"
System.out.println(status.ordinal());       // 1 — posição declarada (índice zero-based)
System.out.println(status.name());          // "PAGO" — nome exato da constante
```

Como cada constante é um objeto singleton gerenciado pela própria JVM, `==` funciona corretamente para comparar enums (diferente de `String`, não há necessidade de `.equals()`, embora ambos funcionem identicamente aqui) — e `enum` automaticamente implementa `Comparable` (pela ordem de declaração) e é seguro para usar como chave de `HashMap` ou em estruturas otimizadas como `EnumMap`/`EnumSet`, que usam a posição ordinal internamente para performance O(1) real, não só amortizada.

Vale registrar por que isso importa em modelagem de domínio: um `enum` é a ferramenta certa quando o conjunto de valores é **fechado e conhecido em tempo de compilação** (status de um pedido, dias da semana, naipes). Quando o conjunto de valores possíveis é aberto ou definido em runtime (categorias configuráveis por um usuário, por exemplo), `enum` é a ferramenta errada — nesse caso o valor pertence a uma tabela de dados, não ao código-fonte.

## Referências

- [Java Language Specification — §8.9: Enum Classes](https://docs.oracle.com/javase/specs/jls/se21/html/jls-8.html#jls-8.9)
- [Oracle Java Tutorials — Enum Types](https://docs.oracle.com/javase/tutorial/java/javaOO/enum.html)
- [`java.lang.Enum` — Javadoc](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Enum.html)
- [`java.util.EnumMap` — Javadoc](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/EnumMap.html)

## Ver também

- [[02 - Arrays e Laços de Repetição]]
- [[03 - Classes, Atributos e Objetos]]
- [[06 - Herança e Polimorfismo]]
