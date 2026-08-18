---
aliases:
  - A Plataforma Java (JDK, JRE, JLS e JVM Spec)
tags:
  - java
  - fundamentos
  - modulo_0
---

# A Plataforma Java: JDK, JRE, JLS e JVM Spec

Quatro siglas que costumam ser confundidas porque descrevem coisas de naturezas diferentes: duas são **especificações** (documentos, JLS e JVM Spec) e duas são **distribuições de software** (JDK e JRE) que implementam essas especificações. A JVM em si já tem nota dedicada — [[A JVM (Java Virtual Machine)]] — então aqui o foco é só posicionar as outras três siglas em relação a ela.

Relação de contenção entre as distribuições:

```
┌────────────────────────────────────────┐
│ JDK (Java Development Kit)              │
│  ┌────────────────────────────────────┐ │
│  │ JRE (Java Runtime Environment)      │ │
│  │  ┌────────────────────────────────┐ │ │
│  │  │ JVM (executa o bytecode)        │ │ │
│  │  └────────────────────────────────┘ │ │
│  │  + bibliotecas core (java.lang,     │ │
│  │    java.util, java.io...)           │ │
│  └────────────────────────────────────┘ │
│  + javac, jlink, jshell, javadoc, jar,  │
│    debugger (jdb), profiler...          │
└────────────────────────────────────────┘
```

## JDK (Java Development Kit)

O kit completo para **desenvolver** software Java: inclui o compilador (`javac`), o runtime (JRE, embutido desde sempre) e ferramentas de linha de comando (`jshell`, `jar`, `javadoc`, `jlink`, `jdeps`, `keytool`, etc.). É o que se instala para programar em Java.

**Curiosidades:**
- Desde o **Java 11**, a Oracle passou a exigir licença comercial paga para o Oracle JDK em produção (fora de uso pessoal/desenvolvimento), o que impulsionou a migração em massa para builds gratuitos do **OpenJDK** — Eclipse Temurin (Adoptium), Amazon Corretto, Azul Zulu, Red Hat build of OpenJDK. Todos compilam o mesmo código-fonte aberto do projeto OpenJDK; a diferença está em suporte, cadência de patches e testes de certificação.
- O `jshell`, introduzido no Java 9 ([JEP 222](https://openjdk.org/jeps/222)), é um REPL (Read-Eval-Print Loop) — permite testar trechos de Java interativamente sem criar classe/arquivo, útil para experimentar APIs rapidamente.

## JRE (Java Runtime Environment)

Contém apenas o necessário para **executar** aplicações Java já compiladas: a JVM + as bibliotecas core. Não inclui `javac` — não compila nada, só roda `.class`/`.jar`.

**Curiosidades:**
- A partir do **Java 11**, a Oracle parou de distribuir o JRE como pacote separado para download. Hoje, na prática, quase ninguém instala "só o JRE" — instala-se o JDK mesmo para rodar aplicações, ou usa-se um runtime minimizado gerado com `jlink` (módulo customizado contendo só as partes da plataforma que a aplicação realmente usa, ver [[Módulos e Memória da JVM (JPMS e JMM)]]).

## JLS (Java Language Specification)

O documento que define a **linguagem Java em si**: sintaxe, regras de tipagem, semântica de cada construção (`if`, `for`, generics, records, sealed classes...). É independente da JVM — descreve o que o código-fonte `.java` significa, não como ele vira bytecode.

**Curiosidades:**
- Mantida sob o **JCP** (ver [[Especificações Java (JSR, JCP, JSP e Servlet)]]) como parte da JSR guarda-chuva de cada versão do Java SE.
- É o motivo pelo qual comportamentos como ordem de avaliação de expressões, overflow de inteiros (`Integer.MAX_VALUE + 1` não lança exceção, dá overflow silencioso) ou regras de *autoboxing* não são "decisões de implementação" da JVM da Oracle — são contrato de linguagem que qualquer JVM compatível precisa respeitar.

## JVM Spec (Java Virtual Machine Specification)

Documento irmão da JLS, mas define o **runtime**: formato binário do `.class`, conjunto de instruções de bytecode, áreas de memória exigidas, regras de verificação. Já coberta em profundidade em [[A JVM (Java Virtual Machine)]] — vale só reforçar que é essa especificação (não a JLS) que permite Kotlin, Scala e Clojure compilarem para a mesma JVM sem terem nada a ver com a sintaxe do Java.

## Conteúdos técnicos

Verificar o que está instalado e distinguir JDK completo de runtime:

```bash
java --version      # versão da JVM/runtime em uso
javac --version     # só existe se houver JDK instalado (JRE standalone não tem)
java --list-modules # módulos disponíveis no runtime atual (JPMS)
```

Gerar um runtime mínimo customizado com `jlink` (substitui o antigo conceito de "JRE genérico" por algo sob medida para a aplicação):

```bash
jlink --module-path $JAVA_HOME/jmods --add-modules java.base,java.logging \
      --output meu-runtime-minimo
```

## Referências oficiais

- [The Java Language Specification (SE 21)](https://docs.oracle.com/javase/specs/jls/se21/html/index.html) — Oracle/OpenJDK
- [The Java Virtual Machine Specification (SE 21)](https://docs.oracle.com/javase/specs/jvms/se21/html/index.html) — Oracle/OpenJDK
- [JEP 222: jshell](https://openjdk.org/jeps/222)
- [OpenJDK](https://openjdk.org)
