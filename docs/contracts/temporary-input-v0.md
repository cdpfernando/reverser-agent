# Contrato temporário de entrada `reverser-fixture/v0`

## Status

Contrato provisório, test-only e sem compatibilidade prometida. Pertence ao Reverser Agent; não é um schema, pacote ou versão antecipada do `csharp2md`.

## Decisão

O Reverser pode iniciar seu vertical slice antes dos wire contracts e retrieval projections oficiais usando uma fixture mínima. O objetivo é desenvolver preflight, ingestão cumulativa da LLM Wiki, investigação wiki-first, revisão, proveniência e publicação atômica sem consumir a implementação legada do produtor.

O seam estável fica entre o leitor e o workflow: evidence bundles e work items. O layout descrito aqui é implementação temporária e pode ser substituído por inteiro.

## Autoridade

A fixture deve ser construída somente a partir de:

- taxonomia, identidades, proof states e invariantes concluídos pelo workstream 1;
- arquitetura normativa vigente do `csharp2md`;
- uma solução sintética controlada;
- relações e locators declarados manualmente para o cenário, sem inferência apresentada como saída real do gerador.

Ela deve registrar o commit normativo e a versão da taxonomia usados. Seu conteúdo comprova apenas o comportamento do Reverser diante do cenário montado; não comprova que o `csharp2md` já consegue produzi-lo.

## Capacidade mínima

O primeiro cenário precisa representar:

- uma solução e seus documentos;
- um entry point;
- callables e pelo menos uma relação direta confirmada;
- evidência e source locator válidos para cada afirmação factual;
- pelo menos um candidato, unknown ou open frontier;
- uma pergunta de engenharia reversa com seeds, budget e resposta esperada;
- código suficiente para uma interpretação explicitamente rotulada;
- um caso inválido para cada gate estrutural essencial.

Isso basta para criar o workspace, fazer a LLM manter `index.md`, `log.md` e uma página de entry point, gerar o mapa-base, responder uma investigação começando pela wiki e fazer file-back de um insight candidate. Componentes, contratos, integrações e persistência podem ser acrescentados em fixtures posteriores conforme os workstreams 5A–5D estabilizarem suas semânticas.

## Layout lógico provisório

```text
fixture/
  manifest.json
  facts.ndjson
  relations.ndjson
  unresolved.ndjson
  catalogs/
    entry-points.json
  sources/
    documents.json
  markdown/                 # retrieval projection csharp2md opcional simulada
    entry-points.md
```

O layout é propositalmente pequeno. Não deve antecipar sharding, posting ordinals, composição multi-solução ou a estrutura física final do produtor.

Cada fixture representa uma única solução. Ledger, revisão e composição pertencem ao knowledge workspace do Reverser, não ao pacote. Cenários multi-solução usam duas ou mais fixtures independentes, permitindo provar que `attach` e `refresh` não reanalisam snapshots existentes.

### Manifest

O manifest identifica, no mínimo:

- `contract`: valor exato `reverser-fixture/v0`;
- `purpose`: valor que deixe explícito `test-only`;
- commit/revisão normativa do `csharp2md`;
- versão da taxonomia;
- identidade lógica da solução sintética;
- caminhos relativos para todos os artefatos;
- hashes determinísticos dos artefatos;
- capacidades presentes e deliberadamente ausentes.

Caminhos absolutos, timestamps dentro do conteúdo determinístico e referências fora da raiz da fixture são inválidos.

### Fatos, relações e unresolveds

Os registros reutilizam nomes canônicos e eixos de prova, mas seu wire shape é local ao contrato temporário. Relações confirmadas carregam origem, destino e evidência. Candidates, unknowns e open frontiers permanecem fora do conjunto confirmado e preservam seu estado.

### Source locators

Cada locator referencia documento, span/seção, assinatura quando aplicável e hash da fonte. O texto necessário ao cenário fica dentro da fixture; o teste não depende de um caminho absoluto para `D:\workspace\csharp2md`.

### Markdown simulado

Este Markdown é opcional e representa somente a futura retrieval projection do `csharp2md`. Ele deve referenciar IDs e locators da mesma fixture e nunca introduzir fatos ausentes dos payloads. Não representa, antecipa nem compartilha layout com a Reverser LLM Wiki escrita pela LLM.

O Reverser só o consulta depois de um resultado estruturado explícito como `no-match`, `ambiguous` ou `insufficient-evidence`. Se o texto levar a evidências ou source locators verificáveis em dois snapshots, o resultado pode ser `correlated`; narrativa ou similaridade isolada permanece `interpretation`. Ausência no Markdown nunca é prova de ausência no pacote.

## Regras de uso

- aceitar somente em modo explícito de desenvolvimento/teste;
- rejeitar em qualquer caminho de produção;
- não expor o layout da fixture a Investigator ou Reviewer;
- fixar workspace revision, snapshots e budget em cada cenário de investigação;
- validar que ingestão cria ou atualiza `index.md`, páginas, cross-references e `log.md` sem tratar a wiki como autoridade;
- validar que query lê a wiki antes de recorrer aos payloads para confirmação ou gaps;
- não gerar fixtures lendo schemas ou outputs legados do `csharp2md`;
- versionar fixture e casos inválidos junto dos testes;
- tratar todo conteúdo-fonte da fixture como dado não confiável;
- nunca usar Markdown isolado para produzir `correlated` ou `confirmed`;
- não adicionar compatibilidade retroativa entre versões provisórias.

## Ciclo de substituição

1. **Workstream 3 — factual storage:** comparar tipos, schemas e validações oficiais com a fixture; corrigir conceitos locais que divergirem, sem preservar o wire provisório.
2. **Workstreams 5A–5D:** ampliar os cenários apenas com semânticas estabilizadas de entry points, relações, persistência, componentes e contratos.
3. **Workstream 6 — retrieval projections:** substituir Markdown/source simulados e fazer o leitor oficial produzir os mesmos evidence bundles/work items.
4. **Workstream 7 — multi-solution composition:** confrontar `correlated` com relações globais confirmadas e candidates oficiais.
5. **Workstream 8 — CLI final:** integrar execução do produtor sem alterar o workflow de investigação e memória.

## Critério de retirada

O contrato temporário deixa de ser entrada operacional quando o leitor do pacote oficial:

- passa pelos mesmos cenários positivos e negativos;
- preserva identidades, proof states, evidências e locators necessários;
- produz páginas e relatórios semanticamente equivalentes para a fixture correspondente.

Depois disso, `reverser-fixture/v0` pode continuar apenas como fixture unitária isolada ou ser removido. Nenhum consumidor externo recebe garantia sobre ele.
