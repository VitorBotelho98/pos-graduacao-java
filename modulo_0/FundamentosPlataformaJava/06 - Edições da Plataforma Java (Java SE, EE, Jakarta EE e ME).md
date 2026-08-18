---
aliases:
  - Edições da Plataforma Java (Java SE, EE, Jakarta EE e ME)
tags:
  - java
  - fundamentos
  - modulo_0
---

# Edições da Plataforma Java: SE, EE, Jakarta EE e ME

"Java" sozinho não diz muito — a plataforma é dividida em edições com escopos e públicos bem diferentes. Entender qual edição uma sigla pertence evita confundir, por exemplo, uma API que vem "de graça" no JDK (Java SE) com uma que só existe se um servidor de aplicação ou uma dependência externa a implementar (Jakarta EE).

## Java SE (Standard Edition)

O núcleo da linguagem e da plataforma: JVM, coleções, I/O, concorrência, streams, etc. — tudo que sai "de fábrica" com o JDK. É a base sobre a qual as outras edições são construídas; toda aplicação Java, seja um script simples ou um sistema corporativo gigante, roda sobre Java SE.

**Curiosidade:** quando alguém diz "Java 21" sem qualificar, está falando de Java SE 21. As outras edições têm seus próprios números de versão, independentes do Java SE (ex.: Jakarta EE 10 roda sobre Java SE 11+).

## Java EE → Jakarta EE (Enterprise Edition)

Conjunto de especificações **sobre** o Java SE voltado a aplicações corporativas: web (Servlet, JSP), persistência (JPA), injeção de dependência (CDI), mensageria (JMS), transações (JTA), etc. — cada uma dessas siglas tem nota própria neste vault. Diferente do Java SE, essas APIs não vêm no JDK: são implementadas por um **servidor de aplicação** (WildFly, Payara, WebLogic, Open Liberty) ou por bibliotecas isoladas.

**Curiosidades:**
- Em **2017**, a Oracle transferiu o Java EE para a **Eclipse Foundation**, e um impasse sobre o direito de uso da marca registrada "Java" forçou o rebatismo para **Jakarta EE**.
- A mudança mais disruptiva veio na **Jakarta EE 9 (2020)**: todo o namespace de pacotes migrou de `javax.*` para `jakarta.*` (ex.: `javax.persistence.Entity` → `jakarta.persistence.Entity`). Foi uma quebra de compatibilidade binária em toda a plataforma, exigindo migração de código em praticamente todo projeto enterprise Java existente — inclusive o próprio Spring Framework precisou de uma major version (Spring Framework 6 / Spring Boot 3) só para acompanhar essa troca.
- Hoje as especificações Jakarta EE são desenvolvidas sob o **Jakarta EE Specification Process**, não mais sob o JCP tradicional (ver [[Especificações Java (JSR, JCP, JSP e Servlet)]]).

## Java ME (Micro Edition)

Subconjunto reduzido do Java SE, projetado para dispositivos com recursos muito limitados: celulares antigos (pré-smartphone), cartões inteligentes, set-top boxes, sistemas embarcados simples.

**Curiosidade:** foi a plataforma por trás de jogos e apps em celulares com teclado físico no início dos anos 2000 (a era pré-iPhone/Android) — o "J2ME" que rodava Snake e afins em Nokias. Com a ascensão do Android (que usa sua própria VM/runtime, não Java ME) e do iOS, o Java ME praticamente desapareceu do mercado de consumo, sobrevivendo hoje em nichos bem específicos de sistemas embarcados e IoT de baixíssimo recurso.

## Conteúdos técnicos

Não há setup de código aqui — é uma distinção de escopo/ecossistema, não de API a ser chamada diretamente. O ponto prático a reter: ao ver uma dependência Maven/Gradle com `jakarta.*` no groupId/import, ela pertence à edição Enterprise (Jakarta EE), não ao Java SE puro, e precisa de uma implementação por trás (servidor de aplicação ou biblioteca standalone como Hibernate para JPA).

## Referências oficiais

- [Jakarta EE — Eclipse Foundation](https://jakarta.ee)
- [Jakarta EE Specification Process](https://jakarta.ee/committees/specification/jesp/)
- [Java SE Documentation — Oracle](https://docs.oracle.com/javase)
