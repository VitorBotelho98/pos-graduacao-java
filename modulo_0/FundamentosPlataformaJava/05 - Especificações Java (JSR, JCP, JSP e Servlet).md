---
aliases:
  - Especificações Java (JSR, JCP, JSP e Servlet)
tags:
  - java
  - fundamentos
  - modulo_0
  - web
---

# Especificações Java: JSR, JCP, JSP e Servlet

JSR e JCP são sobre **processo** (como uma feature da linguagem/plataforma nasce oficialmente); JSP e Servlet são sobre **tecnologia web** concreta. Agrupadas aqui porque a JSP só existe por causa da Servlet, e ambas nasceram formalizadas como JSRs dentro do JCP.

## JCP (Java Community Process)

A organização/processo formal pelo qual mudanças na plataforma Java (Java SE e, historicamente, Java EE) são propostas, discutidas e padronizadas por um comitê de empresas e desenvolvedores independentes (Oracle, Red Hat, IBM, Google, membros individuais...), em vez de decisões unilaterais de um único fornecedor.

**Curiosidade:** apesar de a Oracle ter transferido as especificações **Jakarta EE** para a Eclipse Foundation em 2017 (ver [[Edições da Plataforma Java (Java SE, EE, Jakarta EE e ME)]]), o **JCP continua existindo e ativo** — ele hoje governa apenas Java SE e a JVM (JLS, JVM Spec). É um erro comum achar que o JCP "acabou" com a saída do Java EE; na verdade ele só perdeu esse escopo específico.

## JSR (Java Specification Request)

O documento formal de uma proposta individual dentro do JCP — é o "pedido" para criar ou revisar uma especificação. Cada feature relevante da linguagem/plataforma passou por um JSR com um número identificador.

**Curiosidades:**
- Exemplos famosos: **JSR 133** reescreveu o Java Memory Model no Java 5 (ver [[Módulos e Memória da JVM (JPMS e JMM)]]); **JSR 335** trouxe lambdas e a Streams API no Java 8; **JSR 376** foi o Project Jigsaw (JPMS) no Java 9.
- Desde o Java 9, features menores e mais rápidas de evoluir passaram a usar também o processo de **JEP (JDK Enhancement Proposal)**, mais leve que um JSR completo — por isso hoje é comum ver JEP citado no lugar de JSR para mudanças específicas do OpenJDK, enquanto o JSR "guarda-chuva" ainda existe para a versão do Java SE como um todo.

## JSP (JavaServer Pages)

Tecnologia para gerar HTML dinâmico misturando marcação HTML com código Java embutido (`<% ... %>`) diretamente no arquivo. Por baixo dos panos, cada página `.jsp` é **compilada para um Servlet** na primeira requisição — JSP não é um mecanismo concorrente ao Servlet, é açúcar sintático em cima dele.

**Curiosidade:** JSP é considerada tecnologia legada/em modo de manutenção no ecossistema Java moderno. Misturar lógica de negócio com marcação HTML no mesmo arquivo (o "código-espaguete" clássico de JSP com muito scriptlet) foi amplamente abandonado em favor de template engines mais limpos (Thymeleaf, FreeMarker) ou, mais comumente hoje, de front-ends totalmente desacoplados (SPA em React/Vue/Angular) consumindo APIs REST — daí a relevância maior de JAX-RS (ver [[XML em Java (JAXB, JAX-RS e JAX-WS)]]) sobre JSP em projetos novos.

## Servlet

A API fundamental para lidar com requisições HTTP em Java: uma classe que estende `HttpServlet` e sobrescreve métodos como `doGet`/`doPost` para processar uma requisição e escrever uma resposta. É a camada mais baixa da stack web Java — frameworks como Spring MVC, JSF e o próprio JSP são construídos **sobre** a API de Servlet, não a substituem.

**Curiosidade:** mesmo ao usar Spring Boot, que abstrai quase tudo, o `DispatcherServlet` do Spring MVC é, na raiz, um `HttpServlet` comum registrado no container (Tomcat embutido, por padrão). Entender Servlet ajuda a entender o que Spring está fazendo por trás da injeção de `@RequestMapping`.

## Conteúdos técnicos

Servlet é código, não só teoria — exemplo mínimo de um Servlet processando GET:

```java
@WebServlet("/ola")
public class OlaServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp)
            throws IOException {
        resp.setContentType("text/plain");
        resp.getWriter().write("Olá, mundo!");
    }
}
```

JCP e JSR são puramente processo/documento — não há código associado a eles diretamente.

## Referências oficiais

- [Java Community Process](https://jcp.org)
- [Jakarta Servlet Specification](https://jakarta.ee/specifications/servlet/)
- [Jakarta Server Pages Specification](https://jakarta.ee/specifications/pages/)
