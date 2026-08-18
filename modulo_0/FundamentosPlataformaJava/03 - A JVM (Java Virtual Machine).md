---
aliases:
  - A JVM (Java Virtual Machine)
tags:
  - java
  - jvm
  - fundamentos
  - modulo_0
---

# A JVM (Java Virtual Machine)

## O que é

A JVM é uma **máquina virtual baseada em pilha** (stack-based) que executa bytecode Java. Ela é o que torna o "write once, run anywhere" possível: `javac` compila `.java` para um formato binário intermediário e portável (`.class`), e é a JVM — não o hardware — quem entende esse formato. Cada sistema operacional/arquitetura tem sua própria implementação de JVM, mas o bytecode que ela consome é sempre o mesmo.

Ponto importante para não confundir os três conceitos:
- **JVM Specification**: o documento que define o comportamento (formato do `.class`, semântica das instruções, áreas de memória exigidas). É uma especificação, não código.
- **JDK (Java Development Kit)**: inclui o compilador (`javac`), ferramentas e uma implementação de JVM.
- **Implementações concretas**: **HotSpot** (a JVM de referência da Oracle/OpenJDK, a mais usada no mercado), **OpenJ9** (Eclipse/IBM, otimizada para footprint de memória) e **GraalVM** (JIT alternativo com suporte a compilação nativa via `native-image`). Todas seguem a mesma especificação, mas diferem em como implementam GC, JIT e otimizações internas.

Isso também explica por que Kotlin, Scala, Clojure e Groovy rodam "de graça" sobre a JVM: qualquer linguagem que compile para bytecode válido conforme a especificação ganha o mesmo runtime, GC e JIT que o Java usa (ver [[O que é java?]]).

Referência oficial da especificação: [The Java Virtual Machine Specification](https://docs.oracle.com/javase/specs/jvms/se21/html/index.html) (Oracle/OpenJDK).

## O fluxo completo: do `.java` ao código de máquina

```
MeuPrograma.java
      │  javac (compilador)
      ▼
MeuPrograma.class  (bytecode)
      │
      ▼
┌─────────────────────────────────────────────┐
│                     JVM                      │
│                                               │
│  1. Class Loader Subsystem                   │
│     (loading → linking → initialization)     │
│                     │                         │
│                     ▼                         │
│  2. Runtime Data Areas                       │
│     (Heap, Stack, Metaspace, PC Register,    │
│      Native Method Stack)                    │
│                     │                         │
│                     ▼                         │
│  3. Execution Engine                         │
│     (Interpreter + JIT Compiler (C1/C2))     │
│                     │                         │
│                     ▼                         │
│  4. Garbage Collector (atua sobre a Heap,     │
│     rodando em paralelo/concorrente)          │
└─────────────────────────────────────────────┘
      │
      ▼
 código de máquina nativo (executado pela CPU)
```

O ciclo de vida, passo a passo:

1. **`javac` compila** o `.java` para `.class`, verificando tipos e sintaxe em tempo de compilação.
2. O **Class Loader Subsystem** carrega o `.class` para dentro da JVM, valida o bytecode (para impedir bytecode malformado ou malicioso de comprometer a VM) e prepara a classe para uso.
3. A JVM aloca as **Runtime Data Areas** necessárias: memória de heap para objetos, uma stack por thread para frames de execução, metaspace para metadados de classe.
4. O **Execution Engine** roda o bytecode: por padrão, via **interpretação** instrução a instrução; conforme identifica trechos "quentes" (hot paths), o **JIT compiler** os recompila para código de máquina nativo.
5. O **Garbage Collector** atua continuamente sobre a heap, liberando memória de objetos que não têm mais referências alcançáveis, sem que o desenvolvedor precise gerenciar isso manualmente.

Nenhuma dessas etapas é isolada — elas coexistem e cooperam durante toda a execução do processo Java, não são fases sequenciais e únicas.

## Class Loader Subsystem

Responsável por localizar e carregar bytecode `.class` na JVM em três estágios:

1. **Loading**: lê os bytes do `.class` (de um JAR, do classpath, da rede, etc.) e cria o objeto `Class` correspondente na memória.
2. **Linking**, dividido em três sub-fases:
   - **Verification**: garante que o bytecode é estruturalmente válido e respeita as regras da JVM (não viola tipos, não acessa memória fora dos limites, não corrompe a stack). É a principal linha de defesa contra bytecode malicioso ou corrompido — essencial porque, ao contrário do `javac`, a JVM não pode assumir que todo `.class` que recebe foi gerado por um compilador confiável.
   - **Preparation**: aloca memória para variáveis estáticas da classe e as inicializa com valores-padrão (zero, `null`, `false`).
   - **Resolution**: resolve referências simbólicas (nomes de outras classes/métodos/campos) em referências diretas.
3. **Initialization**: executa o bloco estático (`static {}`) e inicializa variáveis estáticas com seus valores reais. Só acontece na primeira vez que a classe é ativamente usada (lazy initialization).

A hierarquia de class loaders segue o **modelo de delegação por parent-first**: um loader sempre pergunta primeiro ao seu pai antes de tentar carregar a classe ele mesmo. Isso evita que uma classe do usuário substitua acidentalmente (ou maliciosamente) uma classe core do Java, como `java.lang.String`.

```java
public class ClassLoaderDemo {
    public static void main(String[] args) {
        // Bootstrap ClassLoader carrega java.lang.* (implementado em C++, por isso aparece como null)
        System.out.println(String.class.getClassLoader()); // null

        // Application ClassLoader carrega as classes do próprio projeto/classpath
        System.out.println(ClassLoaderDemo.class.getClassLoader());
        // jdk.internal.loader.ClassLoaders$AppClassLoader@...
    }
}
```

Class loaders customizados são a base de mecanismos como hot-reload de plugins e isolamento de dependências em servidores de aplicação (cada WAR/EAR pode ter seu próprio loader, evitando conflito de versões de bibliotecas entre aplicações no mesmo servidor).

## Runtime Data Areas

A especificação da JVM define áreas de memória com escopos e ciclos de vida diferentes ([JVMS §2.5](https://docs.oracle.com/javase/specs/jvms/se21/html/jvms-2.html#jvms-2.5)):

### Heap

Área **compartilhada entre todas as threads**, onde vivem todos os objetos e arrays alocados com `new`. É a única área gerenciada pelo Garbage Collector — quando se fala em "tuning de GC", é o comportamento da heap que está em jogo. Internamente é dividida por geração (ver seção de GC abaixo). Configurada via `-Xms` (tamanho inicial) e `-Xmx` (tamanho máximo).

### Stack

Cada **thread tem sua própria stack**, criada junto com a thread. Armazena **frames**: um frame é empilhado a cada chamada de método e contém variáveis locais, valores intermediários de operações e o endereço de retorno. Quando o método termina, seu frame é descartado. É por isso que variáveis locais primitivas e referências de objetos (não os objetos em si) vivem na stack — rápida de alocar/desalocar porque segue estritamente LIFO, sem necessidade de GC.

Recursão profunda demais estoura o limite da stack (configurável via `-Xss`), lançando `StackOverflowError`:

```java
public class StackDemo {
    static int profundidade = 0;

    static void recursaoInfinita() {
        profundidade++;
        recursaoInfinita(); // sem condição de parada
    }

    public static void main(String[] args) {
        try {
            recursaoInfinita();
        } catch (StackOverflowError e) {
            System.out.println("Estourou em profundidade: " + profundidade);
        }
    }
}
```

### Metaspace

Armazena **metadados de classe**: estrutura de métodos, campos, informações de tipo, o pool de constantes — não os objetos em si, mas a "planta" de cada classe carregada. Até o Java 7, essa área ficava dentro do heap com o nome **PermGen (Permanent Generation)** e tinha tamanho fixo, sendo uma fonte clássica de `OutOfMemoryError: PermGen space` em aplicações com muitos class loaders dinâmicos (containers de aplicação, frameworks com muita geração de proxy/bytecode em runtime).

A partir do **Java 8**, o PermGen foi removido e substituído pelo Metaspace, que vive em **memória nativa (off-heap)**, fora do heap Java, e cresce dinamicamente por padrão (limitável via `-XX:MaxMetaspaceSize`). Isso resolveu boa parte dos `OutOfMemoryError` de metadados em produção, ao custo de mover a responsabilidade para a memória nativa do processo — vale monitorar `-XX:MaxMetaspaceSize` em ambientes com muita geração dinâmica de classes (proxies do Spring/Hibernate, por exemplo). Faz parte do esforço mais amplo de redução de footprint da JVM documentado nas JEP 147 (Reduce Class Metadata Footprint) e JEP 148 (Small VM).

### PC Register e Native Method Stack (por completude)

Cada thread também tem um **PC (Program Counter) Register**, que guarda o endereço da instrução JVM atualmente em execução, e uma **Native Method Stack**, usada quando código Java chama métodos nativos via JNI (C/C++). São menos discutidas no dia a dia, mas fazem parte da especificação e completam o modelo por-thread da JVM.

## Execution Engine e o JIT Compiler

A JVM não usa só um modo de execução — ela é **híbrida**:

1. **Interpretação**: no início, cada instrução de bytecode é interpretada e executada uma a uma. Rápido para começar (sem custo de compilação), mas lento em loops/métodos executados repetidamente.
2. **Profiling**: enquanto interpreta, a JVM (no HotSpot) coleta estatísticas de execução — quais métodos são chamados com mais frequência, quais branches são mais tomados.
3. **JIT (Just-In-Time) compilation**: métodos identificados como "quentes" (hot spots — daí o nome HotSpot) são compilados para código de máquina nativo em tempo de execução, e as chamadas seguintes passam a usar essa versão compilada, muito mais rápida que a interpretada.

O HotSpot usa **tiered compilation** com dois compiladores JIT distintos, atuando em camadas:

- **C1 (client compiler)**: compila rápido, com otimizações mais simples. Prioriza tempo de warm-up baixo.
- **C2 (server compiler)**: aplica otimizações agressivas (inlining, escape analysis, loop unrolling), mas leva mais tempo para compilar. Usado nos métodos que se provam realmente quentes ao longo da execução.

Um método pode passar por múltiplos níveis: interpretado → compilado pelo C1 com profiling → recompilado pelo C2 com otimizações completas, se o profiling confirmar que o método é crítico. Esse modelo em camadas é o motivo pelo qual aplicações Java tendem a ficar mais rápidas depois de alguns segundos/minutos rodando (o "warm-up" da JVM) — algo relevante para benchmarks (nunca medir a primeira execução) e para ambientes serverless/cold-start, onde o warm-up do JIT é puro overhead.

Referência: [HotSpot Group @ OpenJDK](https://openjdk.org/groups/hotspot) mantém os múltiplos compiladores JIT e o runtime de alta performance da JVM de referência.

## Garbage Collector

O GC automatiza a liberação de memória da heap, eliminando `free`/`delete` manual e a classe de bugs associada (use-after-free, double-free) — ao custo de pausas e necessidade de tuning em cenários de baixa latência.

O design se apoia na **hipótese geracional** (generational hypothesis): a observação empírica de que **a maioria dos objetos morre jovem**. Por isso a heap é dividida em gerações com estratégias de coleta diferentes:

- **Young Generation** (dividida em **Eden** e duas áreas de **Survivor**, S0/S1): onde todo objeto novo é alocado. Coletada com frequência ("Minor GC"), de forma rápida, porque a maior parte do Eden costuma estar morta a cada coleta. Objetos que sobrevivem a coletas sucessivas migram entre os survivors e, eventualmente, são **promovidos** para a Old Generation.
- **Old Generation (Tenured)**: objetos de vida longa. Coletada com menos frequência, mas de forma mais cara ("Major/Full GC"), já que a taxa de mortalidade ali é muito menor.

Algoritmos de GC disponíveis no HotSpot (escolhidos via flag, cada um com trade-off diferente entre throughput, latência e uso de memória):

- **Serial GC**: single-threaded, para heaps pequenas/aplicações single-core.
- **Parallel GC**: multi-threaded, otimizado para **throughput** (bom para batch jobs, onde pausas ocasionais são aceitáveis).
- **G1 (Garbage-First)**: default desde o Java 9 ([JEP "Make G1 the Default Garbage Collector"](https://openjdk.org/jeps/0)). Divide a heap em regiões e prioriza coletar primeiro as regiões com mais lixo, equilibrando throughput e pausas previsíveis.
- **ZGC**: coletor de **baixíssima latência**, com pausas sub-milissegundo mesmo em heaps de até 16 TB, usando técnicas concorrentes que evitam parar todas as threads da aplicação por muito tempo (ver [ZGC — The Z Garbage Collector](https://openjdk.org/projects/zgc)).
- **Shenandoah**: alternativa de baixa pausa mantida pela Red Hat, com objetivo similar ao ZGC (pausas independentes do tamanho da heap).

Escolher o GC certo é uma decisão de arquitetura: throughput (Parallel), equilíbrio (G1, o default sensato na maioria dos casos) ou latência extrema (ZGC/Shenandoah, comuns em trading systems, APIs com SLA de latência agressivo).

## Referências oficiais

- [The Java Virtual Machine Specification (SE 21)](https://docs.oracle.com/javase/specs/jvms/se21/html/index.html) — Oracle/OpenJDK
- [HotSpot Group — OpenJDK](https://openjdk.org/groups/hotspot)
- [ZGC — The Z Garbage Collector](https://openjdk.org/projects/zgc)
- [JEP 147: Reduce Class Metadata Footprint](https://openjdk.org/jeps/0) / [JEP 148: Small VM](https://openjdk.org/jeps/0)
- [OpenJDK JEP Index](https://openjdk.org/jeps/0)
