---
aliases:
  - Empacotamento com JAR
  - JAR
tags:
  - java
  - fundamentos
  - ferramentas
  - modulo_0
---

# Empacotamento com JAR

## O que é um JAR

Um arquivo **JAR** (*Java ARchive*) empacota múltiplos arquivos `.class` (e recursos associados — imagens, arquivos de propriedades, outros JARs) num único arquivo distribuível. Estruturalmente, um JAR é um arquivo **ZIP padrão** com uma convenção adicional: um diretório `META-INF/` contendo, opcionalmente, um arquivo de **manifesto** (`MANIFEST.MF`) com metadados sobre o conteúdo ([Javadoc — pacote `java.util.jar`](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/jar/package-summary.html)).

A motivação é a mesma de qualquer formato de empacotamento: distribuir uma biblioteca ou aplicação como um único arquivo, em vez de exigir que quem for usá-la copie uma árvore inteira de `.class` soltos preservando a estrutura de pacotes.

## Compilar vs. empacotar

`javac` compila `.java` em `.class` — isso não é empacotamento, é só a etapa de compilação. `jar` empacota os `.class` já compilados (e outros recursos) num único arquivo:

```bash
javac -d out src/com/exemplo/*.java   # gera .class dentro de out/com/exemplo/
jar cf app.jar -C out .               # empacota tudo que está em out/ dentro de app.jar
```

## Comandos principais da ferramenta `jar`

| Comando | Efeito |
|---|---|
| `jar cf arquivo.jar arquivos...` | **C**ria um novo JAR (`f` indica que o próximo argumento é o nome do arquivo de saída) |
| `jar cfe arquivo.jar Classe arquivos...` | Cria um JAR já definindo a classe principal (**e**ntry point), sem precisar escrever manifesto à mão |
| `jar tf arquivo.jar` | **T**abula (lista) o conteúdo do JAR, sem extrair |
| `jar xf arquivo.jar` | E**x**trai o conteúdo do JAR para o diretório atual |
| `jar uf arquivo.jar arquivos...` | **U**pdate — adiciona/atualiza entradas num JAR já existente |

```bash
jar cfe app.jar com.exemplo.Main -C out .
```

## O manifesto (`MANIFEST.MF`)

O manifesto é um arquivo de texto simples, no formato `Atributo: valor`, que vive em `META-INF/MANIFEST.MF` dentro do JAR. O atributo mais comum em aplicações é `Main-Class`, que indica qual classe tem o `public static void main(String[])` a ser executado:

```
Manifest-Version: 1.0
Main-Class: com.exemplo.Main
```

Um JAR com `Main-Class` definida no manifesto é um **JAR executável** — pode ser rodado diretamente:

```bash
java -jar app.jar
```

Sem essa entrada, `java -jar` falha porque a JVM não sabe qual classe conter o ponto de entrada — nesse caso, é preciso informar a classe manualmente e o classpath (incluindo o próprio JAR):

```bash
java -cp app.jar com.exemplo.Main
```

## Dependências: por que um JAR normal não basta sozinho

Um JAR só contém o código da própria aplicação/biblioteca — se o projeto depende de outras bibliotecas (outros `.jar`), elas precisam estar disponíveis separadamente no classpath em tempo de execução (`java -cp app.jar:dependencia.jar com.exemplo.Main`), ou o manifesto pode declarar `Class-Path:` apontando para JARs relativos. Ferramentas de build como Maven e Gradle resolvem isso automaticamente, e frequentemente produzem um **fat JAR** (ou *uber JAR*) — um único JAR contendo tanto o código da aplicação quanto o conteúdo de todas as dependências, para distribuição autocontida (ex: plugins `maven-shade-plugin`, `spring-boot-maven-plugin`).

## JAR e módulos (desde o Java 9)

Com o Java Platform Module System ([JEP 261](https://openjdk.org/jeps/261), Java 9), um JAR pode se tornar um **JAR modular** ao incluir um `module-info.class` na raiz — descrevendo explicitamente quais pacotes o módulo exporta e de quais outros módulos ele depende. Um JAR sem `module-info.class` continua funcionando normalmente no *classpath* tradicional, ou é tratado como parte do *automatic module* quando colocado no *module path* — a introdução de módulos não quebrou compatibilidade com JARs pré-existentes.

## Referências

- [Javadoc — pacote `java.util.jar`](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/jar/package-summary.html)
- [`jar` — Ferramenta de linha de comando (JDK Tool Specifications)](https://docs.oracle.com/en/java/javase/21/docs/specs/man/jar.html)
- [`java` — Ferramenta de linha de comando, opção `-jar`](https://docs.oracle.com/en/java/javase/21/docs/specs/man/java.html)
- [Oracle Java Tutorials — Packaging Programs in JAR Files](https://docs.oracle.com/javase/tutorial/deployment/jar/index.html)
- [JEP 261: Module System](https://openjdk.org/jeps/261)

## Ver também

- [[07 - Pacotes, Modificadores de Acesso, Getters e Setters]]
- [[14 - Javadoc]]
