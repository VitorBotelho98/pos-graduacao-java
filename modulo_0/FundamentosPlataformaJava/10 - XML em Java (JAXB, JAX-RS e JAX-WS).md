---
aliases:
  - XML em Java (JAXB, JAX-RS e JAX-WS)
tags:
  - java
  - fundamentos
  - modulo_0
  - web
---

# XML em Java: JAXB, JAX-RS e JAX-WS

Três siglas "JAX-" fáceis de confundir pelo nome parecido, mas com propósitos distintos: **JAXB** converte objeto ↔ XML; **JAX-WS** constrói web services **SOAP** (que usam XML como formato de mensagem); **JAX-RS** constrói APIs **REST** (que hoje usam majoritariamente JSON, apesar do nome do grupo "XML" as reunir historicamente).

## JAXB (Jakarta XML Binding)

API para **marshalling/unmarshalling**: transformar objetos Java em XML (`marshal`) e XML de volta em objetos Java (`unmarshal`), usando anotações (`@XmlRootElement`, `@XmlElement`) para controlar o mapeamento.

**Curiosidade:** JAXB fazia parte do JDK core até o **Java 8**. A partir do **Java 11** ([JEP 320: Remove the Java EE and CORBA Modules](https://openjdk.org/jeps/320)), foi removida do JDK junto com outros módulos Java EE legados — quem precisa de JAXB hoje precisa adicionar a dependência explicitamente (`jakarta.xml.bind`). Pegou muita gente de surpresa em migrações de Java 8 para versões mais novas, com `NoClassDefFoundError` inesperado até adicionar a dependência.

## JAX-WS (Jakarta XML Web Services)

API para construir e consumir **web services SOAP** (Simple Object Access Protocol): contratos formais definidos em **WSDL** (Web Services Description Language), mensagens estritamente tipadas em XML, geração de client/server a partir do contrato.

**Curiosidade:** SOAP/JAX-WS é considerado tecnologia legada frente a REST/JAX-RS para APIs novas — o overhead de XML, o acoplamento forte ao WSDL e a complexidade de ferramentas tornaram REST+JSON o padrão de facto. Ainda assim, JAX-WS continua relevante em integrações **enterprise/bancárias/governamentais**, onde contratos formais e tipagem estrita (e às vezes exigências regulatórias) favorecem SOAP sobre a flexibilidade "solta" do REST.

## JAX-RS (Jakarta RESTful Web Services)

API padrão Java para construir APIs **REST** via anotações declarativas (`@Path`, `@GET`, `@POST`, `@Produces`, `@Consumes`), mapeando métodos Java para endpoints HTTP. Implementada por frameworks como **Jersey** (implementação de referência) e **RESTEasy**.

**Curiosidade:** apesar de ser a API "REST" oficial da plataforma Jakarta EE, o **Spring MVC/Spring WebFlux não implementa JAX-RS** — o Spring tem seu próprio conjunto de anotações (`@RestController`, `@GetMapping`) que nasceu em paralelo e é hoje muito mais usado no ecossistema Java do que JAX-RS puro. É importante não confundir os dois modelos de anotação ao ler código de projetos diferentes — visualmente parecidos, mas de especificações distintas.

## Conteúdos técnicos

Exemplo mínimo de mapeamento JAXB (objeto ↔ XML):

```java
@XmlRootElement
public class Cliente {
    @XmlElement
    private String nome;
    // getters/setters omitidos
}

JAXBContext ctx = JAXBContext.newInstance(Cliente.class);
Marshaller marshaller = ctx.createMarshaller();
marshaller.marshal(new Cliente(), System.out);
// <cliente><nome>...</nome></cliente>
```

Exemplo mínimo de recurso JAX-RS:

```java
@Path("clientes/{id}")
public class ClienteResource {

    @GET
    @Produces("application/json")
    public Cliente buscar(@PathParam("id") long id) {
        return clienteService.buscarPorId(id);
    }
}
```

## Referências oficiais

- [Jakarta XML Binding Specification](https://jakarta.ee/specifications/xml-binding/)
- [Jakarta XML Web Services Specification](https://jakarta.ee/specifications/xml-web-services/)
- [Jakarta RESTful Web Services Specification](https://jakarta.ee/specifications/restful-ws/)
- [JEP 320: Remove the Java EE and CORBA Modules](https://openjdk.org/jeps/320)
