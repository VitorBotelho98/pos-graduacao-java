# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## O que é este projeto

Vault Obsidian de estudos da pós-graduação, com foco em **Java**. Não é um projeto de software (sem build, testes ou linter) — é uma base de conhecimento em Markdown organizada por módulos (`modulo_0/`, `modulo_1/`, ...), pensada para funcionar como **guia de estudos e referência técnica** de consulta contínua, não como anotações soltas.

O objetivo de cada nota não é só explicar "o que é" um conceito, mas:
- Conectar o conceito a uma **implementação real** (trecho de código, exemplo prático rodável) sempre que possível.
- Explicar o **porquê** das decisões de design da linguagem/ecossistema, não só o "como usar".
- Apontar **referências oficiais** para aprofundamento (documentação Oracle, OpenJDK/JEPs, Spring), evitando fontes secundárias como fonte primária de verdade.

## Estrutura

- Cada módulo é uma pasta (`modulo_0/`, `modulo_1/`, ...) correspondendo a uma etapa do curso.
- Cada nota é um arquivo `.md` com frontmatter de tags do Obsidian, por exemplo:
  ```yaml
  ---
  tags:
    - java
    - fundamentos
    - modulo_0
  ---
  ```
  Sempre inclua ao menos a tag do módulo (`modulo_N`) e tags temáticas relevantes (ex: `java`, `jvm`, `spring`, `concorrencia`).
- Use `[[link]]` do Obsidian para conectar notas relacionadas entre módulos, em vez de duplicar explicações.

## Convenções ao escrever/editar notas

- **Priorize exemplos práticos**: quando o conceito permitir, inclua um bloco de código Java compilável/executável (não pseudocódigo), e comente o trecho não-óbvio (o porquê, não o quê).
- **Explique o porquê, não só o quê**: trade-offs de design (ex: por que Java não tem herança múltipla de classes, por que a JVM adotou JIT híbrido) são mais valiosos que definições de dicionário.
- **Cite fontes oficiais**: ao referenciar uma feature, prefira linkar para:
  - Documentação oficial Oracle (docs.oracle.com/javase)
  - JEPs / OpenJDK (openjdk.org/jeps)
  - Spring (docs.spring.io) quando o tópico envolver o framework
  Evite basear uma afirmação técnica só em blogs/tutoriais de terceiros.
- **Atualização incremental**: este documento e as notas devem crescer junto com o curso — ao adicionar um módulo novo, adicione a pasta correspondente e, se necessário, atualize este CLAUDE.md com novas convenções específicas do módulo (não reescreva o histórico já registrado).
- **Tom**: direto e técnico, voltado a quem já programa mas está aprendendo Java especificamente (ver `modulo_0/O que é java?.md` como referência de profundidade e estilo esperados).
- **Enumere itens em ordem**: ao listar etapas, características ou itens onde a sequência importa (ex: fases de execução da JVM, ordem de inicialização de classes, passos de um algoritmo), use listas numeradas (`1.`, `2.`, `3.`) em vez de marcadores (`-`). Reserve marcadores para itens sem relação de ordem/precedência entre si.
- **Compare entre versões do Java quando aplicável**: se o recurso documentado mudou de sintaxe/abordagem ao longo das versões da linguagem, mostre a evolução lado a lado (ex: `for`/`foreach` → Streams com lambda; `switch` statement clássico → `switch` expression do Java 14/JEP 361; laço tradicional → `var` do Java 10). Indique a versão em que cada forma foi introduzida e o porquê da mudança (legibilidade, imutabilidade, redução de boilerplate), não só a sintaxe nova.

## Ferramentas obrigatórias

- **Skill `/obsidian-cli`**: use sempre que a tarefa envolver operações no vault (criar, ler, buscar, mover ou atualizar notas; gerenciar propriedades/frontmatter; listar tags ou backlinks). Prefira essa skill em vez de manipular os arquivos `.md` diretamente com ferramentas genéricas de sistema de arquivos, para manter a integridade dos links `[[...]]` e do frontmatter do Obsidian.
- **MCP `context7` (skill `/context7-mcp`)**: use sempre que uma nota fizer afirmação técnica sobre uma biblioteca/framework/API (Java, Spring, JVM, ferramentas de build, etc.) — consulte a documentação atual via `context7` antes de escrever, em vez de confiar só em conhecimento prévio ou em blogs de terceiros. Isso reforça a convenção de citar fontes oficiais acima.
