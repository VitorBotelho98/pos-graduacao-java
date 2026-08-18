---
aliases:
  - O que é java?
tags:
  - java
  - fundamentos
  - modulo_0
---

# O que é Java?

## Um pouco de história

Java nasceu em 1991 dentro da Sun Microsystems, num projeto interno chamado *Green*, liderado por James Gosling. O objetivo original nem era programação para web ou backend corporativo: era criar uma linguagem para eletrônicos de consumo (TVs a cabo, set-top boxes), onde o hardware variava muito entre fabricantes. Daí nasceu o requisito central que define a linguagem até hoje: **"Write Once, Run Anywhere" (WORA)** — compilar uma vez e rodar em qualquer plataforma sem recompilar.

Lançada oficialmente em 1995, a linguagem pegou carona no boom da web logo em seguida (applets), mas o que realmente sustentou sua adoção em larga escala foi o lado server-side: Servlets, J2EE (depois Java EE, hoje Jakarta EE) e um ecossistema corporativo gigantesco nos anos 2000.

Pontos que um líder técnico deveria saber sobre a trajetória:

- **Sun → Oracle (2010)**: Oracle comprou a Sun e assumiu a governança da linguagem. Isso trouxe mudanças no modelo de licenciamento do JDK ao longo do tempo (vale acompanhar qual distribuição de JDK seu time usa — Oracle JDK, Eclipse Temurin/Adoptium, Amazon Corretto, etc., pois têm termos de suporte e licenciamento diferentes).
- **Mudança de cadência de releases (2017 em diante)**: até o Java 8, as versões saíam sem prazo fixo (e o Java 8, de 2014, ficou muito tempo como padrão de mercado). A partir do Java 9, Oracle adotou um **ciclo de release a cada 6 meses**, com versões **LTS (Long-Term Support)** a cada poucos ciclos (Java 8, 11, 17, 21, 25...). Isso muda a forma como um time deve planejar upgrades: a estratégia sensata é migrar entre versões LTS, não tentar acompanhar cada release intermediária em produção.
- **JCP (Java Community Process) e JEPs**: a evolução da linguagem é formalizada via JSRs/JEPs, com múltiplos vendors participando (não é decisão unilateral da Oracle). Isso dá estabilidade, mas também torna o processo de introdução de novas features mais lento e conservador — é uma linguagem que evita "quebrar" código antigo.

## Como a linguagem funciona (o modelo técnico)

O que diferencia Java de linguagens compiladas nativamente (C/C++) ou interpretadas puras (Python, no modelo clássico) é o modelo de **máquina virtual + bytecode**:

1. **Compilação para bytecode**: o código-fonte (`.java`) é compilado pelo `javac` para um formato intermediário chamado **bytecode** (`.class`), que não é código de máquina nativo — é a instrução de uma CPU hipotética, a JVM.
2. **Execução pela JVM**: a Java Virtual Machine carrega esse bytecode e o executa. Como o bytecode é o mesmo em qualquer sistema operacional, apenas a JVM precisa ser específica para cada plataforma — daí o "compile uma vez, rode em qualquer lugar".
3. **JIT (Just-In-Time compilation)**: na prática, a JVM não fica só interpretando bytecode instrução por instrução. Ela identifica trechos "quentes" (hot paths, executados com frequência) e os compila para código de máquina nativo em tempo de execução, otimizando o desempenho conforme o programa roda. É um modelo híbrido: interpretado no início, compilado nativamente onde importa.
4. **Gerenciamento automático de memória (Garbage Collector)**: Java não expõe ponteiros nem exige `free`/`delete` manual. O GC roda em background, identificando e liberando objetos que não são mais referenciados. Isso remove uma classe inteira de bugs (use-after-free, double-free, vazamentos clássicos de ponteiro), mas introduz outra: pausas de GC e necessidade de tuning (heap size, escolha de algoritmo de GC — G1, ZGC, Shenandoah, etc.) em aplicações de alta performance/baixa latência. Um líder técnico precisa saber que "gerenciamento automático de memória" não é sinônimo de "não preciso pensar em memória" — ainda existem vazamentos lógicos (referências não removidas de coleções, listeners não desregistrados) e a escolha do GC importa em produção.

## Características centrais da linguagem

- **Fortemente tipada e estaticamente tipada**: tipos são verificados em tempo de compilação, o que pega uma classe de erros antes de ir para produção — trade-off contra a velocidade de escrita de linguagens dinamicamente tipadas.
- **Orientada a objetos (com pragmatismo)**: tudo (exceto tipos primitivos) é objeto, com os pilares clássicos (herança, encapsulamento, polimorfismo, abstração). Desde o Java 8, a linguagem incorporou conceitos funcionais (lambdas, Streams, interfaces funcionais) sem abandonar o paradigma OO — é um modelo híbrido, não uma reescrita.
- **Sem herança múltipla de classes, mas com múltipla implementação de interfaces**: decisão de design deliberada para evitar o "diamond problem" do C++.
- **Portabilidade real**: o mesmo `.jar`/`.class` roda em Linux, Windows, macOS, contanto que haja uma JVM compatível — isso é especialmente relevante para times que rodam em containers/Kubernetes com infraestrutura heterogênea.
- **Ecossistema de build e dependências**: Maven e Gradle são os gerenciadores de build dominantes; ambos resolvem dependências transitivas e definem o ciclo de vida de compilação/teste/empacotamento. Entender minimamente o `pom.xml`/`build.gradle` do time é parte da base técnica esperada de uma liderança.
- **Módulos (desde o Java 9, o "Project Jigsaw")**: introduziu o sistema de módulos (`module-info.java`), permitindo encapsulamento mais forte do que pacotes/JARs sozinhos ofereciam. Adoção no mercado ainda é parcial — muita coisa em produção ainda roda no "classpath clássico".

## Onde Java é aplicado hoje

Java está longe de ser só "aquela linguagem de sistema bancário legado". Os grandes domínios de uso atuais:

- **Backend / aplicações corporativas**: o caso de uso mais forte e o que sustenta a linguagem no mercado — APIs REST, microsserviços, sistemas financeiros, ERPs. Frameworks como **Spring/Spring Boot** e application servers (WildFly, Quarkus, entre outros) dominam esse espaço.
- **Android**: até a chegada do Kotlin (hoje a linguagem preferida pelo Google), todo o desenvolvimento Android nativo era feito em Java, e uma enorme base de apps ainda roda código Java (a Android Runtime, ART, não é a JVM padrão, mas o modelo — bytecode + VM — é o mesmo espírito).
- **Big Data**: boa parte do ecossistema de processamento distribuído é escrito em Java ou roda sobre a JVM — **Hadoop, Kafka, Elasticsearch, Cassandra** são exemplos de infraestrutura crítica de dados construída em Java.
- **Aplicações desktop**: menos comum hoje, mas ainda existe via Swing/JavaFX — IDEs como **IntelliJ IDEA e Eclipse** são, elas mesmas, aplicações Java rodando sobre a JVM.
- **Sistemas embarcados e cartões inteligentes**: o nicho original da linguagem (Java Card é usado até hoje em SIM cards e cartões bancários com chip).
- **Ambientes de missão crítica**: sistemas de bolsas de valores, bancos, seguradoras e até parte do controle de tráfego aéreo usam Java pela combinação de maturidade, tooling e previsibilidade (apesar do GC).

## Curiosidades

- **O nome "Java" não tem nada a ver com programação**: a equipe queria chamar a linguagem de "Oak" (por causa de um carvalho perto da janela do escritório de James Gosling), mas o nome já era registrado. "Java" veio de uma sessão de brainstorming e é uma referência ao café — a ilha indonésia conhecida por sua produção cafeeira. É por isso que o logo é uma xícara de café fumegante.
- **Java quase se chamou "Silk" ou "WebRunner"**: outros nomes cogitados antes de "Oak" virar problema.
- **A JVM não é exclusiva do Java**: várias outras linguagens rodam sobre ela e se beneficiam do mesmo runtime, JIT e GC — **Kotlin, Scala, Clojure e Groovy** são os exemplos mais conhecidos. Isso significa que "escrever para a JVM" e "escrever em Java" não são a mesma coisa.
- **"Write Once, Run Anywhere" virou piada interna**: por causa de inconsistências reais entre JVMs de fabricantes diferentes nos anos 90, a comunidade apelidou a promessa de **"Write Once, Debug Everywhere"**.
- **A Netscape ajudou a popularizar Java**: o navegador Netscape Navigator foi um dos primeiros a rodar applets Java, o que deu à linguagem sua primeira grande visibilidade pública em 1995 — embora applets tenham praticamente desaparecido hoje (mortos por questões de segurança e pela ascensão do JavaScript no browser).
- **Java e JavaScript não têm relação técnica entre si**: apesar do nome parecido, foi uma jogada de marketing da época (Netscape queria surfar no hype de Java, que estava em alta, ao batizar sua nova linguagem de script). São linguagens com sintaxe, tipagem e propósitos completamente diferentes.
- **O rei do "legado que nunca morre"**: por sua estabilidade de compatibilidade retroativa, é comum encontrar em produção, ainda hoje, sistemas escritos originalmente em Java 6 ou 7 rodando por décadas sem reescrita — um reflexo direto do compromisso da linguagem em nunca quebrar código antigo.
