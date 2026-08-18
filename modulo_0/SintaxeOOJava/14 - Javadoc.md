---
aliases:
  - Javadoc
tags:
  - java
  - fundamentos
  - ferramentas
  - modulo_0
---

# Javadoc

## O que é

Javadoc é duas coisas ao mesmo tempo: um **formato de comentário de documentação** (`/** ... */`) reconhecido pelo compilador e por IDEs, e uma **ferramenta de linha de comando** (`javadoc`) que lê esses comentários no código-fonte e gera documentação em HTML navegável. É assim que a própria [documentação oficial da API Java](https://docs.oracle.com/en/java/javase/21/docs/api/index.html) é gerada — a partir dos comentários Javadoc no código-fonte do próprio OpenJDK.

```java
/**
 * Calcula o valor total de um pedido, incluindo frete.
 *
 * @param valorBase valor dos itens, sem frete, deve ser positivo
 * @param frete custo do frete, pode ser zero
 * @return o valor total do pedido
 * @throws IllegalArgumentException se valorBase for negativo
 */
public double calcularTotal(double valorBase, double frete) {
    if (valorBase < 0) {
        throw new IllegalArgumentException("valorBase não pode ser negativo");
    }
    return valorBase + frete;
}
```

## Diferença de um comentário comum

Um comentário Javadoc começa com `/**` (dois asteriscos), não `/*`. Essa diferença sintática é o que a ferramenta usa para identificar quais comentários processar — um `/* comentário normal */` é ignorado pelo `javadoc`, mesmo estando imediatamente acima de um método público.

```java
/* Isto é um comentário comum — ignorado pela ferramenta javadoc */
/** Isto é um comentário Javadoc — processado pela ferramenta javadoc */
```

## Estrutura de um comentário Javadoc

Um comentário Javadoc tem duas partes: uma **descrição** em prosa (cuja primeira frase vira o resumo exibido em listagens) seguida de **tags de bloco**, cada uma iniciando uma nova linha com `@`:

| Tag | Uso |
|---|---|
| `@param nome descrição` | Documenta um parâmetro do método/construtor |
| `@return descrição` | Documenta o valor de retorno (omitido se o método for `void`) |
| `@throws Tipo descrição` (ou `@exception`) | Documenta uma exceção que o método pode lançar |
| `@see referência` | Referência cruzada para outra classe/método relacionado |
| `@since versão` | Versão em que o elemento foi introduzido |
| `@deprecated motivo` | Marca o elemento como obsoleto, com orientação de substituição |
| `@author nome` | Autor da classe (uso mais raro em código de aplicação) |

Tags **inline**, usadas dentro do texto entre chaves, incluem `{@code trecho}` (formata como código, sem interpretar como HTML) e `{@link Classe#metodo}` (cria um link clicável para outro elemento documentado):

```java
/**
 * Representa um {@link Pedido} já processado.
 * Use {@code Pedido.vazio()} para criar uma instância sem itens.
 *
 * @since 1.0
 */
public class PedidoProcessado { }
```

## Gerando a documentação HTML

O comando `javadoc` lê os arquivos-fonte e produz um site HTML navegável, equivalente em estrutura ao site oficial da API Java:

```bash
javadoc -d docs -sourcepath src com.exemplo.pedidos
```

`-d docs` define o diretório de saída; `-sourcepath` aponta para a raiz do código-fonte; o último argumento é o pacote (ou pacotes) a documentar. Ferramentas de build como Maven (`maven-javadoc-plugin`) e Gradle (`javadoc` task nativa) encapsulam esse comando, gerando o site como parte do processo de build/publicação — é assim que bibliotecas publicadas no Maven Central costumam disponibilizar Javadoc pronto (ex: no javadoc.io).

## Por que documentar assim, e não com comentários soltos

Comentários Javadoc ficam **acoplados à assinatura** que documentam — a IDE exibe o conteúdo do `/** ... */` como tooltip ao passar o mouse sobre uma chamada de método, mesmo sem abrir o arquivo-fonte. Isso muda o propósito do comentário: não é uma nota para quem lê o código-fonte, é a **documentação pública do contrato** de uma API — o que um método promete fazer, quais entradas aceita, o que pode dar errado — consumida por quem usa a classe sem necessariamente ler sua implementação.

Por isso, boas práticas de Javadoc documentam **o quê e o contrato** (comportamento observável, pré-condições, exceções possíveis), não a implementação interna — o "como" por trás do código já é responsabilidade de comentários normais dentro do corpo do método, quando algo não-óbvio justificar um.

## Referências

- [Oracle — How to Write Doc Comments for the Javadoc Tool](https://www.oracle.com/technical-resources/articles/java/javadoc-tool.html)
- [Documentation Comment Specification (`jdk.javadoc` module)](https://docs.oracle.com/en/java/javase/21/docs/specs/javadoc/doc-comment-spec.html)
- [`javadoc` — Ferramenta de linha de comando (JDK Tool Specifications)](https://docs.oracle.com/en/java/javase/21/docs/specs/man/javadoc.html)
- [Java SE 21 API Documentation (exemplo de saída gerada pela ferramenta)](https://docs.oracle.com/en/java/javase/21/docs/api/index.html)

## Ver também

- [[03 - Classes, Atributos e Objetos]]
- [[15 - Empacotamento com JAR]]
