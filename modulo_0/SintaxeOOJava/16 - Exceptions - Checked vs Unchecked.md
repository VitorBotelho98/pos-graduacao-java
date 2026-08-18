---
aliases:
  - Exceptions - Checked vs Unchecked
  - Checked vs Unchecked Exceptions
tags:
  - java
  - fundamentos
  - modulo_0
---

# Exceptions: Checked vs. Unchecked

## Hierarquia de `Throwable`

Toda condição excepcional em Java é representada por uma instância de `Throwable` ou subclasse — só objetos `Throwable` podem ser lançados (`throw`) ou capturados (`catch`). A hierarquia se divide em duas ramificações diretas ([Javadoc — `java.lang.Throwable`](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Throwable.html)):

```
Throwable
├── Error              (java.lang.Error)
│   └── ex: OutOfMemoryError, StackOverflowError
└── Exception           (java.lang.Exception)
    ├── RuntimeException (java.lang.RuntimeException)
    │   └── ex: NullPointerException, IllegalArgumentException, IndexOutOfBoundsException
    └── (demais subclasses diretas de Exception)
        └── ex: IOException, SQLException
```

A distinção **checked vs. unchecked** não é sobre gravidade do erro — é sobre **se o compilador exige tratamento explícito**:

| | Checked | Unchecked |
|---|---|---|
| Definição | `Exception` e suas subclasses, **exceto** `RuntimeException` e suas subclasses | `RuntimeException` e suas subclasses, e `Error` e suas subclasses |
| Verificação em compile-time | Sim — o compilador exige `catch` ou declaração via `throws` | Não — o compilador não exige nada |
| Exemplos | `IOException`, `SQLException`, `ParseException` | `NullPointerException`, `IllegalArgumentException`, `IndexOutOfBoundsException`, `ArithmeticException` |
| Representa, tipicamente | Condições externas previsíveis, fora do controle do código (arquivo inexistente, falha de rede) | Erros de programação (bug) ou violação de pré-condição do próprio código |

A [JLS §11.2](https://docs.oracle.com/javase/specs/jls/se21/html/jls-11.html#jls-11.2) formaliza exatamente essa regra: *"checked exceptions are Throwables that are not subclasses of RuntimeException or Error"* — a checagem em tempo de compilação (*compile-time checking*) se aplica só a essa categoria.

## Checked: obrigação de tratar ou declarar

Um método que pode lançar uma exceção checked precisa **capturá-la** (`try`/`catch`) ou **declará-la** na assinatura (`throws`), propagando a obrigação para quem o chama:

```java
import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Path;

// Opção 1: declarar via throws — propaga a obrigação para o chamador
public String lerArquivo(String caminho) throws IOException {
    return Files.readString(Path.of(caminho));
}

// Opção 2: capturar e tratar localmente
public String lerArquivoSeguro(String caminho) {
    try {
        return Files.readString(Path.of(caminho));
    } catch (IOException e) {
        return ""; // decide o que fazer com a falha, aqui mesmo
    }
}
```

Se nenhuma das duas opções for feita, o código **não compila** — essa é a diferença prática central. O compilador força quem escreve o método a decidir, no momento em que escreve o código, o que acontece quando a operação falha — em vez de deixar essa decisão implícita e descoberta só em runtime.

## Unchecked: sem exigência do compilador

`RuntimeException` (e `Error`) podem ser lançadas sem nenhuma declaração ou captura obrigatória — o código compila normalmente mesmo que a exceção nunca seja tratada:

```java
public int dividir(int a, int b) {
    return a / b; // pode lançar ArithmeticException (divisão por zero) — não precisa de throws nem try/catch
}
```

`Error` (`OutOfMemoryError`, `StackOverflowError`) representa falhas graves da própria JVM ou do ambiente de execução, que uma aplicação normalmente **não deveria tentar capturar e recuperar** — se a JVM ficou sem memória, capturar o erro raramente resolve o problema de fundo. `RuntimeException` tipicamente sinaliza bug de programação (`NullPointerException`, `ArrayIndexOutOfBoundsException`) ou violação de contrato (`IllegalArgumentException`, `IllegalStateException`) — condições que, em teoria, código correto deveria evitar de acontecer, não precisar tratar toda vez.

## Por que essa distinção existe (e por que é debatida)

A justificativa original de exceções checked é forçar tratamento explícito de falhas **previsíveis e recuperáveis**, vindas de fontes externas ao controle do programa — E/S de arquivo, rede, banco de dados — evitando que essas falhas sejam ignoradas silenciosamente por esquecimento. É uma garantia em tempo de compilação que outras linguagens (C++, C#, a maioria das linguagens modernas) simplesmente não adotaram — nelas, toda exceção se comporta como unchecked.

Na prática, o design gerou controvérsia dentro da própria comunidade Java: código que se propaga por várias camadas de chamada frequentemente acumula `throws IOException, SQLException, ...` em cascata, ou força blocos `try/catch` que só reempacotam a exceção sem realmente tratá-la — o próprio [Java Tutorials da Oracle dedica uma seção à controvérsia](https://docs.oracle.com/javase/tutorial/essential/exceptions/runtime.html), observando que muitas APIs modernas (incluindo boa parte da API de Streams introduzida no Java 8) preferem exceções unchecked justamente para evitar esse acoplamento de assinatura.

## `try`-`catch`-`finally` e `try`-with-resources

```java
try {
    // código que pode lançar exceção
} catch (IOException e) {
    // tratamento específico
} catch (SQLException e) {
    // tratamento de outro tipo
} finally {
    // sempre executa, com ou sem exceção — tipicamente liberação de recursos
}
```

Desde o Java 7, **multi-catch** permite tratar múltiplos tipos de exceção não relacionados no mesmo bloco, quando o tratamento é idêntico:

```java
try {
    processar();
} catch (IOException | SQLException e) {
    log.error("Falha ao processar", e);
}
```

E **try-with-resources** (recurso introduzido no Java 7 junto ao ARM, *Automatic Resource Management* — sem JEP formal, já que o processo JEP só passou a existir a partir do Java 9) fecha automaticamente qualquer recurso que implemente `AutoCloseable`, ao final do bloco, mesmo em caso de exceção — eliminando o padrão repetitivo de fechar recursos manualmente num `finally`:

```java
// Antes do Java 7
BufferedReader br = new BufferedReader(new FileReader("arquivo.txt"));
try {
    return br.readLine();
} finally {
    br.close(); // fácil de esquecer, ou de não executar se close() em si lançar exceção
}

// Java 7+ (try-with-resources)
try (BufferedReader br = new BufferedReader(new FileReader("arquivo.txt"))) {
    return br.readLine();
} // br.close() é chamado automaticamente aqui, garantidamente
```

## Exceções customizadas e encadeamento (*cause*)

Uma exceção customizada estende `Exception` (checked) ou `RuntimeException` (unchecked), conforme o mesmo critério de "o chamador deveria ser obrigado a lidar com isso agora?":

```java
public class SaldoInsuficienteException extends RuntimeException {
    public SaldoInsuficienteException(String mensagem, Throwable causa) {
        super(mensagem, causa); // encadeia a exceção original como causa raiz
    }
}
```

Passar a exceção original como `causa` (segundo argumento de `super(...)`) preserva o *stack trace* completo — visível via `getCause()` — em vez de mascarar a origem real do problema ao relançar uma exceção nova sem contexto.

## Referências

- [Javadoc — `java.lang.Throwable` (Java SE 21)](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Throwable.html)
- [Javadoc — `java.lang.Exception`](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Exception.html)
- [Javadoc — `java.lang.RuntimeException`](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/RuntimeException.html)
- [Java Language Specification — §11.2 Compile-Time Checking of Exceptions](https://docs.oracle.com/javase/specs/jls/se21/html/jls-11.html#jls-11.2)
- [Oracle Java Tutorials — Unchecked Exceptions: The Controversy](https://docs.oracle.com/javase/tutorial/essential/exceptions/runtime.html)
- [Oracle Java Tutorials — The try-with-resources Statement](https://docs.oracle.com/javase/tutorial/essential/exceptions/tryResourceClose.html)

## Ver também

- [[03 - Classes, Atributos e Objetos]]
- [[12 - Object, toString e Comparação de Objetos]]
