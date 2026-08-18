---
aliases:
  - Herança e Polimorfismo
tags:
  - java
  - fundamentos
  - orientacao-a-objetos
  - modulo_0
---

# Herança e Polimorfismo

## Herança

Herança modela a relação **"é um"** (*is-a*): uma subclasse estende uma superclasse via `extends`, herdando seus atributos e métodos não-privados e podendo adicionar ou sobrescrever comportamento.

```java
public class Veiculo {
    protected String modelo;
    protected int velocidadeAtual;

    public Veiculo(String modelo) {
        this.modelo = modelo;
        this.velocidadeAtual = 0;
    }

    public void acelerar(int incremento) {
        velocidadeAtual += incremento;
    }

    public String status() {
        return modelo + " a " + velocidadeAtual + " km/h";
    }
}

public class Carro extends Veiculo {
    private int numeroPortas;

    public Carro(String modelo, int numeroPortas) {
        super(modelo); // chama o construtor da superclasse — obrigatoriamente a 1ª instrução
        this.numeroPortas = numeroPortas;
    }
}
```

`super(...)` invoca o construtor da superclasse e deve ser a primeira instrução do construtor da subclasse (se omitido, o compilador insere `super()` sem argumentos automaticamente — o que só compila se a superclasse tiver um construtor sem parâmetros disponível). Fora de um construtor, `super.metodo()` chama explicitamente a versão da superclasse de um método que a subclasse sobrescreveu — útil quando a subclasse quer *estender* o comportamento herdado, não substituí-lo por completo.

### Por que Java não tem herança múltipla de classes

C++ permite que uma classe estenda várias outras diretamente, o que gera o **problema do diamante**: se as classes `B` e `C` herdam de `A` e ambas sobrescrevem um método `m()`, e uma classe `D` herda de `B` e `C` simultaneamente, qual versão de `m()` `D` deveria herdar? A resposta não é óbvia e diferentes linguagens resolvem isso de formas ad-hoc e propensas a confusão.

Java evita o problema pela raiz: **uma classe só pode estender uma única superclasse** (`extends` aceita apenas um nome). Em compensação, uma classe pode **implementar múltiplas interfaces**, porque até o Java 7 interfaces só continham assinaturas de método sem implementação — não havia ambiguidade possível, já que não existia corpo de método para colidir. Desde o [JEP 126 / Java 8](https://openjdk.org/jeps/126), interfaces podem ter **métodos `default`** (com corpo), o que reabriu parcialmente a possibilidade de colisão — e a linguagem resolve isso obrigando a classe implementadora a sobrescrever explicitamente o método quando duas interfaces fornecem `default` conflitantes, em vez de escolher uma automaticamente. É uma decisão de design deliberada: prefere forçar o desenvolvedor a resolver a ambiguidade a decidir por ele de forma implícita.

```java
public interface Flutuavel {
    default void mover() { System.out.println("Flutuando"); }
}

public interface Rolavel {
    default void mover() { System.out.println("Rolando"); }
}

// Obrigatório sobrescrever mover() explicitamente — o compilador não escolhe por você
public class Anfibio implements Flutuavel, Rolavel {
    @Override
    public void mover() {
        Flutuavel.super.mover(); // possível invocar a versão de uma interface específica
    }
}
```

### Classe abstrata vs. interface

| | Classe abstrata | Interface |
|---|---|---|
| Herança/implementação | Uma subclasse só pode estender **uma** | Uma classe pode implementar **várias** |
| Estado (atributos de instância) | Pode ter | Não pode (só constantes `static final` implícitas) |
| Construtor | Pode ter (chamado via `super()`) | Não tem |
| Métodos com implementação | Sim, livremente | Só via `default`/`static` (desde Java 8) |
| Quando usar | Quando subclasses compartilham **estado** e uma base de implementação comum, além do contrato | Quando o objetivo é só definir um **contrato de comportamento**, possivelmente implementado por classes não relacionadas entre si |

```java
public abstract class FormaGeometrica {
    protected String cor;

    public FormaGeometrica(String cor) {
        this.cor = cor;
    }

    // Método abstrato: sem corpo, força cada subclasse concreta a implementar
    public abstract double calcularArea();

    // Método concreto: compartilhado por todas as subclasses, sem repetição
    public String descricao() {
        return "Forma " + cor + " com área " + calcularArea();
    }
}

public class Circulo extends FormaGeometrica {
    private double raio;

    public Circulo(String cor, double raio) {
        super(cor);
        this.raio = raio;
    }

    @Override
    public double calcularArea() {
        return Math.PI * raio * raio;
    }
}
```

Não é possível instanciar `new FormaGeometrica(...)` diretamente — uma classe `abstract` existe justamente para ser estendida, nunca instanciada sozinha; o compilador impede a criação de instância direta.

## Polimorfismo

Polimorfismo é a capacidade de tratar objetos de tipos diferentes de forma uniforme através de um tipo comum (superclasse ou interface), com o comportamento correto sendo resolvido **em tempo de execução** de acordo com o tipo real do objeto — não o tipo declarado da variável.

```java
FormaGeometrica[] formas = {
    new Circulo("vermelho", 5.0),
    new Circulo("azul", 2.0)
};

for (FormaGeometrica f : formas) {
    // Mesma chamada de método, comportamento diferente conforme o tipo REAL do objeto
    System.out.println(f.descricao());
}
```

Esse mecanismo — chamado *dynamic dispatch* (ou *virtual method invocation*) — funciona porque, sob o capô, a JVM não decide qual implementação de `calcularArea()` chamar olhando o tipo da variável (`FormaGeometrica`); ela consulta, em runtime, uma tabela de métodos virtuais associada à classe **real** do objeto apontado pela referência. É por isso que a mesma linha de código (`f.calcularArea()`) executa implementações diferentes dependendo se `f` aponta para um `Circulo`, um `Quadrado` etc., mesmo com `f` declarada como `FormaGeometrica` — o tipo declarado só limita **quais métodos são visíveis** ao compilador, não qual implementação roda.

### `@Override` e o contrato herdado

A anotação `@Override` não é obrigatória para sobrescrever um método, mas seu uso é fortemente recomendado: ela instrui o compilador a verificar que o método realmente sobrescreve algo da superclasse/interface, pegando em tempo de compilação o erro clássico de digitar a assinatura errada (ex: `calculararea()` em minúsculo) — sem `@Override`, isso compilaria silenciosamente como um método novo e não relacionado, e o polimorfismo simplesmente não aconteceria.

### `instanceof` e downcasting

Quando o comportamento polimórfico não é suficiente e é preciso tratar um objeto de acordo com seu tipo concreto, `instanceof` verifica o tipo real em runtime antes de um *downcast* (converter a referência de volta para o tipo mais específico):

```java
// Antes do Java 16: verificação e cast em passos separados
if (forma instanceof Circulo) {
    Circulo c = (Circulo) forma;
    System.out.println(c.getRaio());
}

// Java 16+ (JEP 394, pattern matching for instanceof): verifica e faz o cast em um passo
if (forma instanceof Circulo c) {
    System.out.println(c.getRaio()); // "c" já está disponível, com o tipo certo, dentro do if
}
```

O [JEP 394](https://openjdk.org/jeps/394) elimina o boilerplate redundante do cast manual (o compilador já provou, pela checagem do `instanceof`, que o cast é seguro — repeti-lo manualmente era puro ruído) e reduz erros de copiar/colar um cast para o tipo errado.

### `final` para impedir extensão/sobrescrita

`final` aplicado a uma classe impede que ela seja estendida (`final class Utilitario`); aplicado a um método, impede que subclasses o sobrescrevam. É uma ferramenta de design deliberada — sinaliza que o comportamento é garantido e não deve variar por herança, útil para métodos cuja lógica é sensível a segurança ou invariantes internas do objeto que uma subclasse mal implementada poderia quebrar.

## Referências

- [Java Language Specification — Chapter 8: Classes](https://docs.oracle.com/javase/specs/jls/se21/html/jls-8.html)
- [Java Language Specification — Chapter 9: Interfaces](https://docs.oracle.com/javase/specs/jls/se21/html/jls-9.html)
- [Oracle Java Tutorials — Interfaces and Inheritance](https://docs.oracle.com/javase/tutorial/java/IandI/index.html)
- [JEP 126: Lambda Expressions & Virtual Extension Methods (métodos `default`)](https://openjdk.org/jeps/126)
- [JEP 394: Pattern Matching for instanceof](https://openjdk.org/jeps/394)

## Ver também

- [[03 - Classes, Atributos e Objetos]]
- [[05 - Arrays de Objetos, Composição e Enums]]
- [[07 - Pacotes, Modificadores de Acesso, Getters e Setters]]
- [[09 - Interfaces]]
- [[10 - Classes Abstratas e Static]]
- [[11 - Interface vs Classe Abstrata]]
