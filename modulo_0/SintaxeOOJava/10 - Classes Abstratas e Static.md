---
aliases:
  - Classes Abstratas e Static
  - Classes abstratas
  - static
tags:
  - java
  - fundamentos
  - orientacao-a-objetos
  - modulo_0
---

# Classes Abstratas e `static`

## Classes abstratas

Uma classe `abstract` é uma classe que **não pode ser instanciada diretamente** — ela existe para ser estendida, fornecendo uma base parcial de implementação que subclasses concretas completam.

```java
public abstract class FormaGeometrica {
    protected String cor;

    public FormaGeometrica(String cor) {
        this.cor = cor;
    }

    // Método abstrato: sem corpo, cada subclasse concreta é obrigada a implementar
    public abstract double calcularArea();

    // Método concreto: implementação compartilhada, herdada sem repetição
    public String descricao() {
        return "Forma " + cor + " com área " + calcularArea();
    }
}
```

```java
FormaGeometrica f = new FormaGeometrica("azul"); // erro de compilação: classe abstrata
```

Pontos que a JLS e o compilador aplicam:

- Uma classe abstrata **pode ter construtor**, mesmo não sendo instanciável diretamente — o construtor só é executado quando uma subclasse concreta é instanciada, via `super(...)` implícito ou explícito. Isso permite à classe abstrata inicializar estado comum (`cor`, no exemplo) antes que a subclasse assuma.
- Uma classe abstrata **pode ter atributos de instância**, métodos concretos, estáticos, construtores e blocos de inicialização — a única restrição real é não poder ser instanciada com `new` diretamente.
- Se uma classe tem **pelo menos um** método abstrato, ela é obrigatoriamente `abstract` — o compilador não permite uma classe concreta com método sem corpo. O inverso não é verdade: uma classe pode ser `abstract` sem nenhum método abstrato, só para impedir instanciação direta.
- Uma subclasse que não implementa **todos** os métodos abstratos herdados também precisa ser declarada `abstract` — a obrigação de implementar "desce" até alguma classe concreta assumir o contrato completo.

## `static`

`static` marca um membro como pertencente **à classe**, não a uma instância específica — existe uma única cópia compartilhada por todos os objetos, e é acessível mesmo sem nenhuma instância existir.

### Atributos estáticos

```java
public class ContadorDeInstancias {
    private static int totalCriado = 0; // uma única cópia, compartilhada por todas as instâncias
    private int id;

    public ContadorDeInstancias() {
        totalCriado++;
        this.id = totalCriado;
    }
}

new ContadorDeInstancias();
new ContadorDeInstancias();
System.out.println(ContadorDeInstancias.totalCriado); // 2 — acessado pela classe, não por uma instância
```

Cada `new ContadorDeInstancias()` cria um `id` próprio (atributo de instância), mas todos compartilham e incrementam o **mesmo** `totalCriado`.

### Métodos estáticos

Um método `static` não recebe uma referência implícita `this` — por isso, não pode acessar atributos ou métodos de instância diretamente (não há instância associada à chamada). É o caso de métodos utilitários que não dependem de estado de objeto: `Math.sqrt(x)`, `Integer.parseInt(s)`, `Collections.sort(lista)`.

```java
public class Calculadora {
    public static int somar(int a, int b) { // sem 'this', sem estado de instância
        return a + b;
    }
}

Calculadora.somar(2, 3); // chamado pela classe, sem precisar de "new Calculadora()"
```

Consequência direta e frequentemente esquecida: **um método não pode ser `abstract` e `static` ao mesmo tempo**. Polimorfismo (*dynamic dispatch*, ver [[06 - Herança e Polimorfismo]]) resolve qual implementação chamar em tempo de execução, com base no tipo real do objeto apontado por uma referência — mas métodos estáticos são resolvidos em **tempo de compilação**, pelo tipo declarado, sem *dispatch* dinâmico. Não faz sentido um método que não tem *dispatch* dinâmico ser, ao mesmo tempo, uma obrigação de contrato a ser cumprida polimorficamente por subclasses — por isso a JLS proíbe a combinação.

### Bloco de inicialização estático

Executa **uma única vez**, quando a classe é carregada pela JVM (no *class loading*) — antes de qualquer instância ser criada ou membro estático acessado:

```java
public class ConfiguracaoApp {
    static final Map<String, String> PROPRIEDADES;

    static {
        // lógica de inicialização que uma simples atribuição em linha não expressaria
        PROPRIEDADES = new HashMap<>();
        PROPRIEDADES.put("versao", "1.0");
        PROPRIEDADES.put("ambiente", "producao");
    }
}
```

Se houver múltiplos blocos `static { }` na mesma classe, eles executam **na ordem em que aparecem no código-fonte**, entrelaçados com as inicializações de campos estáticos com valor inicial — conforme a ordem textual do arquivo ([JLS §12.4.2 — Detailed Initialization Procedure](https://docs.oracle.com/javase/specs/jls/se21/html/jls-12.html#jls-12.4.2)).

### Classes aninhadas estáticas (`static nested class`)

Uma classe aninhada `static` não carrega referência implícita à instância externa que a contém — diferente de uma *inner class* não-estática, que mantém um vínculo implícito com a instância envolvente (`Outer.this`) e por isso só pode ser instanciada a partir de (ou associada a) uma instância de `Outer`.

```java
public class Arvore {
    class No { }              // inner class: carrega referência implícita a uma Arvore específica

    static class Configuracao { } // static nested class: independente, sem vínculo com instância de Arvore
}

Arvore.Configuracao cfg = new Arvore.Configuracao(); // não precisa de "new Arvore()" antes
```

Na prática, quando a classe aninhada não precisa acessar o estado da instância externa, `static` é a escolha correta — evita manter uma referência implícita desnecessária (o que também tem implicação de memória: uma *inner class* não-estática impede a coleta de lixo da instância externa enquanto a inner class viver).

### `import static`

Permite referenciar membros estáticos de outra classe sem qualificar pelo nome da classe, útil para constantes e utilitários usados com frequência:

```java
import static java.lang.Math.PI;
import static java.lang.Math.sqrt;

double area = PI * sqrt(raio); // em vez de Math.PI, Math.sqrt(raio)
```

## Referências

- [Java Language Specification — §8.1.1.1 abstract Classes](https://docs.oracle.com/javase/specs/jls/se21/html/jls-8.html#jls-8.1.1.1)
- [Java Language Specification — §8.4.3.2 abstract Methods](https://docs.oracle.com/javase/specs/jls/se21/html/jls-8.html#jls-8.4.3.2)
- [Java Language Specification — §12.4.2 Detailed Initialization Procedure](https://docs.oracle.com/javase/specs/jls/se21/html/jls-12.html#jls-12.4.2)
- [Oracle Java Tutorials — Understanding Class Members (static)](https://docs.oracle.com/javase/tutorial/java/javaOO/classvars.html)
- [Oracle Java Tutorials — Nested Classes](https://docs.oracle.com/javase/tutorial/java/javaOO/nested.html)

## Ver também

- [[06 - Herança e Polimorfismo]]
- [[09 - Interfaces]]
- [[11 - Interface vs Classe Abstrata]]
