# Reverser Agent

Assistente interativo de engenharia reversa para sistemas C#/.NET. Ele explora pacotes factuais e projeções de código produzidos pelo [`csharp2md`](https://github.com/cdpfernando/csharp2md), responde perguntas com proveniência e mantém uma wiki versionada como memória durável do conhecimento descoberto.

## Produto

O produto é o assistente; sua memória operacional é uma LLM Wiki cumulativa, escrita e mantida pela LLM e consultada antes de uma nova exploração de baixo nível. Ela não é um relatório final separado do uso: ingestões atualizam o conhecimento existente e respostas úteis podem retornar à wiki por file-back explícito.

```text
ApiUsuarios.sln -> csharp2md -> package A ─┐
ApiCompras.sln  -> csharp2md -> package B ─┼─> knowledge workspace
ApiProdutos.sln -> csharp2md -> package C ─┘          |
                                                       v
                                 LLM Wiki <-> investigação
```

O `csharp2md` possui a autoridade factual sobre código, configuração, identidades, relações confirmadas e evidências. A LLM do Reverser possui a autoria editorial da wiki — páginas, cross-references, `index.md`, `log.md` e consistência semântica — enquanto módulos determinísticos validam e persistem sem escrever síntese.

## Estado

Este repositório está na fase de contrato e primeiro vertical slice. O `csharp2md` ainda executa seu roadmap de substituição; para avançar sem consumir o legado, o desenvolvimento usa [`reverser-fixture/v0`](docs/contracts/temporary-input-v0.md), um contrato test-only e descartável.

O primeiro slice exercita uma solução e uma investigação completa. A interface e os artefatos internos já consideram um workspace que acumula várias soluções sem reanalisar as anteriores.

## O que o assistente faz

- executa ou consome uma análise `csharp2md` por solução;
- valida e anexa snapshots imutáveis a um knowledge workspace;
- substitui ou remove uma solução sem corromper a última revisão válida;
- integra cada snapshot à wiki existente e atualiza `index.md`, cross-references e `log.md`;
- constrói um mapa-base de arquitetura, entry points, integrações, contratos, persistência e gaps;
- consulta primeiro a wiki e retorna aos pacotes/código para confirmar, aprofundar ou preencher lacunas;
- responde perguntas, segue fluxos e avalia impacto direto ou reverso;
- interpreta regras de negócio por meio do código projetado;
- correlaciona soluções sem promover inferência a fato confirmado;
- confronta documentação declarada com comportamento observado;
- gera páginas e diagramas rastreáveis;
- arquiva descobertas revisadas de investigações por file-back explícito.

## Estados do conhecimento

| Estado | Autoridade |
| --- | --- |
| `confirmed` | fato ou relação comprovada pelo `csharp2md` |
| `correlated` | ligação proposta pelo Reverser com evidência verificável nos dois pacotes |
| `interpretation` | leitura semântica da LLM, incluindo regras de negócio |
| `unresolved` | evidência insuficiente, ambiguidade, frontier ou capacidade ausente |
| `curated` | conhecimento aceito ou documentado por uma pessoa |

A Markdown retrieval projection do `csharp2md`, isoladamente, nunca produz `correlated`. Ela pode servir como fallback para localizar identidades, evidências e source locators; narrativa sem suporte permanece `interpretation`.

## Fluxo externo

A interface do Reverser concentra três intenções:

1. alterar o workspace — `attach`, `refresh` ou `detach`;
2. investigar — `ask`, `trace`, `impact` ou `explain`;
3. promover conhecimento — fazer file-back de um insight candidate revisado e publicar uma nova revisão da wiki.

Uma mudança de workspace autorizada inclui a manutenção editorial necessária da wiki; não exige uma segunda confirmação para cada página afetada. Orquestração, agentes, seleção de contexto, budgets, retries, composição, lint e publicação atômica permanecem dentro da implementação.

## Estrutura

```text
agents/                  papéis e protocolo do conjunto de agentes
docs/
  architecture.md       módulos, seams, autoridade e artefatos
  product-flow.md       fluxo ponta a ponta e comportamento da ferramenta
  contracts/            contratos provisórios e futuros contratos estáveis
  roadmap.md            fases e gates de evolução
  scope.md              produto, limites e critérios do primeiro slice
  research/             análise das referências e decisões investigadas
AGENTS.md                instruções para agentes que alteram este repositório
```

Comece por [escopo](docs/scope.md), [fluxo do produto](docs/product-flow.md), [arquitetura](docs/architecture.md), [contrato temporário](docs/contracts/temporary-input-v0.md), [papéis dos agentes](agents/README.md) e [roadmap](docs/roadmap.md).

## Referências

- [Egonex-AI/Understand-Anything](https://github.com/Egonex-AI/Understand-Anything)
- [compozy/kb](https://github.com/compozy/kb)
- [pesquisa comparativa](docs/research/reference-architectures.md)
- [viabilidade do fallback Markdown](docs/research/markdown-fallback-feasibility.md)
- [modelo LLM Wiki de Karpathy](docs/research/karpathy-llm-wiki-model.md)
- [gist oficial de Karpathy](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)
- arquitetura normativa local do `csharp2md`: `D:\workspace\csharp2md\docs\architecture\README.md`
