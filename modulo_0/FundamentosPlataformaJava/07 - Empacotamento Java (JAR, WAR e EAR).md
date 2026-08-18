---
aliases:
  - Empacotamento Java (JAR, WAR e EAR)
tags:
  - java
  - fundamentos
  - modulo_0
---

# Empacotamento Java: JAR, WAR e EAR

Três formatos de arquivo para distribuir código Java compilado, em ordem crescente de escopo: **JAR** empacota classes/recursos genéricos, **WAR** empacota uma aplicação web específica, **EAR** empacota múltiplos módulos (WARs, JARs de EJB) como uma única aplicação corporativa. Todos são, por baixo dos panos, arquivos **ZIP** com uma estrutura de diretórios e metadados convencionados — nada de mágico no formato binário em si.

## JAR (Java ARchive)

Formato genérico para empacotar classes `.class` compiladas + recursos (arquivos de propriedades, imagens, etc.) em um único arquivo, com um `META-INF/MANIFEST.MF` descrevendo metadados (versão, classpath, classe principal). É o formato usado tanto para bibliotecas (dependências no Maven Central são JARs) quanto para aplicações executáveis standalone.

**Curiosidade:** um "fat JAR"/"uber JAR" (todo o Spring Boot gera um por padrão) é um JAR que embute **todas** as dependências dentro de si, tornando o deploy um único arquivo autocontido executável com `java -jar app.jar` — sem precisar de classpath externo configurado. É essa técnica que praticamente eliminou a necessidade prática de EAR em boa parte dos projetos modernos (ver abaixo).

## WAR (Web Application Archive)

Formato específico para empacotar uma **aplicação web** Java: Servlets, JSPs, classes, bibliotecas (`WEB-INF/lib`), e o descritor `WEB-INF/web.xml` (hoje majoritariamente substituído por anotações, mas ainda suportado). Um servidor de aplicação ou servlet container (Tomcat, Jetty) sabe como "descompactar" e rodar um WAR.

**Curiosidade:** frameworks modernos como Spring Boot, por padrão, **não geram mais WAR** — geram um JAR executável com um servidor (Tomcat/Jetty embutido) dentro do próprio JAR. Gerar WAR hoje é a exceção, reservada a quando é preciso realmente fazer deploy em um servidor de aplicação externo já existente (comum em ambientes corporativos legados que mantêm um único Tomcat/WildFly compartilhado por múltiplas aplicações).

## EAR (Enterprise Application Archive)

Formato que agrupa **múltiplos módulos** — um ou mais WARs, JARs de EJB, bibliotecas compartilhadas — em uma única unidade de deploy corporativa, com um descritor `META-INF/application.xml` amarrando tudo. Pensado para servidores de aplicação Java EE completos (WebLogic, WebSphere, JBoss/WildFly).

**Curiosidade:** EAR é hoje o mais "datado" dos três formatos na prática do mercado. A arquitetura de **microsserviços** foi na direção oposta da ideia de EAR (um monólito corporativo gigante com vários módulos empacotados juntos): cada serviço é hoje tipicamente seu próprio fat JAR independente, implantado e escalado isoladamente — muitas vezes nem em um servidor de aplicação tradicional, mas em container Docker rodando `java -jar`. EAR ainda aparece em manutenção de sistemas legados grandes, raramente em projetos novos.

## Conteúdos técnicos

Inspecionar e manipular JAR com a ferramenta `jar` (vem no JDK):

```bash
jar tf app.jar                    # lista o conteúdo sem extrair
jar xf app.jar                    # extrai
jar cfm app.jar MANIFEST.MF -C classes/ .   # cria um JAR a partir de um manifest + diretório
java -jar app.jar                 # executa (requer Main-Class no manifest)
```

Estrutura de diretórios esperada dentro de um WAR:

```
app.war
├── index.html
├── WEB-INF/
│   ├── web.xml          (opcional em apps modernas baseadas em anotação)
│   ├── classes/         (.class compilados da aplicação)
│   └── lib/             (JARs de dependências)
```

## Referências oficiais

- [Jar File Specification — Oracle](https://docs.oracle.com/javase/8/docs/technotes/guides/jar/jar.html)
- [Jakarta Servlet Specification](https://jakarta.ee/specifications/servlet/) (estrutura do WAR)
- [Jakarta Platform, Enterprise Edition Specification](https://jakarta.ee/specifications/platform/) (estrutura do EAR)
