---
aliases:
  - Interface vs Classe Abstrata
  - Interface vs. Classe Abstrata
tags:
  - java
  - fundamentos
  - orientacao-a-objetos
  - modulo_0
---

# Interface vs. Classe Abstrata

Interface e classe abstrata resolvem problemas parecidos — permitir que múltiplos tipos concretos compartilhem um contrato ou comportamento comum — mas partem de premissas de design diferentes. Ver [[09 - Interfaces]] e [[10 - Classes Abstratas e Static]] para os detalhes de cada um isoladamente; esta nota foca na **comparação** e em quando escolher um ou outro.

## Comparação direta

| | Classe abstrata | Interface |
|---|---|---|
| Relação modelada | "É um" (*is-a*), com estado e implementação parcial compartilhados | "Pode fazer isso" (*can-do*), um contrato de capacidade |
| Herança/implementação | Uma subclasse só pode estender **uma** classe (`extends`) | Uma classe pode implementar **várias** interfaces (`implements A, B, C`) |
| Estado (atributos de instância) | Pode ter, com qualquer modificador de acesso | Não pode — só constantes `public static final` implícitas |
| Construtor | Pode ter, chamado via `super(...)` pela subclasse | Não tem |
| Modificadores de acesso dos métodos | `public`, `protected`, `private`, pacote-privado — livre | Só `public` (implícito) ou `private` (Java 9+); nunca `protected` nem pacote-privado |
| Métodos com implementação | Sim, livremente, desde sempre | Só via `default`/`static`/`private` (Java 8+/9+) |
| Blocos de inicialização de instância | Pode ter | Não pode (não há estado de instância a inicializar) |
| Versionamento de API | Adicionar método novo (não-abstrato) não quebra subclasses — elas já herdam via `extends` | Antes do Java 8, adicionar método quebrava toda implementação existente; `default` resolveu isso |
| Quando usar | Subclasses compartilham **estado** e uma base de implementação, além do contrato | Definir um contrato de comportamento implementável por classes não relacionadas entre si |

## Por que a diferença de "quantos posso estender/implementar" existe

A restrição de herança simples de classes (`extends` aceita só um nome) evita o **problema do diamante**: se uma classe herdasse estado e implementação de duas superclasses que definem o mesmo campo ou o mesmo método de formas conflitantes, não haveria uma resolução não-ambígua. Interfaces, historicamente, não tinham esse risco porque não carregavam nem estado nem implementação — só assinaturas. Métodos `default` (Java 8) reabriram parcialmente a possibilidade de conflito, e a linguagem resolve isso **obrigando a classe implementadora a sobrescrever explicitamente** o método quando duas interfaces fornecem `default`s conflitantes — o detalhe completo, com exemplo de código, está em [[06 - Herança e Polimorfismo]] (seção "Por que Java não tem herança múltipla de classes").

## Um mesmo tipo pode combinar as duas

Uma classe concreta tipicamente **estende uma** classe abstrata (herdando estado e implementação de base) **e implementa várias** interfaces (assumindo múltiplos contratos independentes):

```java
public interface Comparavel<T> {
    int comparar(T outro);
}

public interface Serializavel {
    String serializar();
}

public abstract class Funcionario {
    protected String nome;
    protected double salarioBase;

    public Funcionario(String nome, double salarioBase) {
        this.nome = nome;
        this.salarioBase = salarioBase;
    }

    public abstract double calcularSalario(); // cada tipo de funcionário calcula diferente

    public String identificacao() { // comportamento comum, compartilhado por herança
        return "Funcionário: " + nome;
    }
}

public class Gerente extends Funcionario implements Comparavel<Gerente>, Serializavel {
    private double bonus;

    public Gerente(String nome, double salarioBase, double bonus) {
        super(nome, salarioBase);
        this.bonus = bonus;
    }

    @Override
    public double calcularSalario() {
        return salarioBase + bonus;
    }

    @Override
    public int comparar(Gerente outro) {
        return Double.compare(this.calcularSalario(), outro.calcularSalario());
    }

    @Override
    public String serializar() {
        return nome + ";" + calcularSalario();
    }
}
```

`Gerente` herda **estado e implementação parcial** de `Funcionario` (um relacionamento *is-a* forte, com dados compartilhados) e assume **contratos independentes** de `Comparavel` e `Serializavel` (capacidades que não têm relação hierárquica entre si — uma `String` também pode ser `Comparavel`, sem ter nenhuma relação de herança com `Gerente`).

## Regra prática de decisão

1. Se as implementações precisam compartilhar **estado** (atributos) ou uma base de implementação com lógica não-trivial reutilizada por todas → **classe abstrata**.
2. Se o objetivo é só declarar **"este tipo sabe fazer X"**, possivelmente implementado por classes completamente não relacionadas (ex: `Comparable`, `Runnable`, `AutoCloseable` do próprio `java.lang`/`java.util`) → **interface**.
3. Se é preciso que um tipo assuma **múltiplos** papéis independentes simultaneamente → precisa ser interface (ou múltiplas interfaces), porque `extends` não permite combinar múltiplas classes.
4. Se a API é pública e pode precisar evoluir (adicionar métodos) sem quebrar quem já implementa → interface com métodos `default` dá essa flexibilidade desde o Java 8; numa classe abstrata, isso já era possível adicionando métodos concretos (não-abstratos) sem quebrar subclasses.

## Referências

- [Java Language Specification — Chapter 8: Classes](https://docs.oracle.com/javase/specs/jls/se21/html/jls-8.html)
- [Java Language Specification — Chapter 9: Interfaces](https://docs.oracle.com/javase/specs/jls/se21/html/jls-9.html)
- [Oracle Java Tutorials — Abstract Methods and Classes](https://docs.oracle.com/javase/tutorial/java/IandI/abstract.html)
- [Oracle Java Tutorials — Interfaces and Inheritance](https://docs.oracle.com/javase/tutorial/java/IandI/index.html)

## Ver também

- [[06 - Herança e Polimorfismo]]
- [[09 - Interfaces]]
- [[10 - Classes Abstratas e Static]]
