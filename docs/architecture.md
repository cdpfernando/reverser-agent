# Arquitetura inicial

## Princípio central

O Reverser é um módulo downstream profundo: oferece poucas intenções e esconde a complexidade de ingestão, recuperação, agentes, composição, revisão e publicação.

A Reverser LLM Wiki é a memória operacional dentro desse módulo. A LLM possui sua autoria editorial: cria e atualiza páginas, mantém cross-references, `index.md`, `log.md` e coerência semântica. Módulos determinísticos verificam contratos e persistem os arquivos, mas não escrevem síntese nem decidem significado.

Sua interface externa conceitual possui três operações:

```text
ChangeWorkspace(change) -> WorkspaceResult
Investigate(question)    -> InvestigationResult
Promote(candidate)       -> PublicationResult
```

- `ChangeWorkspace` cobre criação, `attach`, `refresh` e `detach`;
- `Investigate` cobre perguntas, traces, impacto e explicações;
- `Promote` cobre o file-back explícito de conhecimento derivado de uma investigação.

Uma mudança de workspace autorizada inclui a ingestão editorial e a publicação da wiki resultante depois dos gates. Chat, CLI ou futura UI são adapters dessa mesma interface. O chamador não orquestra agentes, páginas, budgets, retries ou arquivos intermediários.

## Fluxo estrutural

```text
solution path or existing package
             |
             v
     csharp2md / fixture reader
             |
       validated snapshot
             |
             v
       workspace staging ------ failure ------> last valid revision
             |
      compose + LLM wiki ingest
      index + pages + log + lint
             |
             v
     published workspace revision
             |
        +----+-------------------+
        |                        |
 investigation              knowledge memory
 wiki-first answer      candidates + wiki revisions
 + evidence             + append-only log
```

O fluxo detalhado e os estados operacionais estão em [`product-flow.md`](product-flow.md).

## Autoridade

```text
source/configuration
  -> csharp2md facts and confirmed relations     authoritative
    -> catalogs, postings, csharp2md Markdown    derived retrieval
      -> Reverser correlated                     downstream evidence-backed
        -> Reverser interpretation               semantic reading
          -> Reverser LLM Wiki                   cumulative derived memory
            -> curated overlay                   human-accepted claim
              -> diagram/answer                  derived presentation
```

Camadas downstream não alteram proof states upstream. `confirmed` só vem do `csharp2md`. Uma pessoa pode aceitar uma interpretação como `curated`, mas isso não a transforma em fato do gerador.

READMEs e documentação configurada formam uma trilha de claims separada. Quando contradizem o pacote factual, o Reverser apresenta “documentado” e “observado” em vez de resolver silenciosamente o conflito.

## Knowledge workspace

Um workspace possui:

- identidade e política;
- ledger de soluções registradas;
- exatamente um snapshot ativo por solução;
- snapshots históricos imutáveis;
- revisão publicada que fixa o conjunto ativo;
- composição factual e correlações downstream;
- mapa-base e Reverser LLM Wiki cumulativa;
- schema editorial, `index.md`, páginas, cross-references e `log.md` append-only;
- investigations e insight candidates;
- relatórios de validação e publicação.

Uma revisão é imutável. Alterações são preparadas em staging e só substituem a revisão publicada depois de todos os gates.

## Módulos internos

### Ingestão de pacotes

Executa ou recebe o pacote de uma solução, valida manifest, versões, hashes, caminhos e capacidades e produz um `SolutionSnapshot`. Não interpreta conhecimento.

O desenvolvimento usa um leitor concreto de [`reverser-fixture/v0`](contracts/temporary-input-v0.md). O leitor oficial o substituirá. O seam estável fica nos evidence bundles/work items; o formato da fixture não atravessa para os agentes.

### Ciclo de vida do workspace

Aplica `attach`, `refresh` e `detach`, mantém o ledger e coordena staging, composição, manutenção da wiki e publicação. Reanalisar uma solução não reabre as demais. Uma falha preserva a última revisão válida.

### Recuperação e investigação

Converte pergunta, escopo e budget em work items; consulta primeiro `index.md` e páginas relevantes da revisão ativa. Retorna aos insumos estruturados e ao código completo para confirmar citações, resolver conteúdo stale, aprofundar uma explicação ou preencher gaps. Produz um `InvestigationResult` sem modificar evidência upstream.

### Composição e correlação

Mantém separados:

- relações globais `confirmed` do futuro compositor `csharp2md`;
- relações `correlated` produzidas pelo Reverser;
- `interpretation` baseada em semântica ou narrativa;
- `unresolved` e alternativas rejeitadas.

Antes do compositor oficial, o Reverser pode correlacionar snapshots. Depois, reconcilia cada correlação com o resultado oficial, sem promoção silenciosa.

### Reverser LLM Wiki

É escrita e mantida pela LLM, não por um compiler determinístico. Na ingestão, a LLM lê o snapshot novo no contexto da wiki existente, atualiza todas as páginas afetadas, mantém cross-references e `index.md` e acrescenta uma entrada a `log.md`. Em queries, lê a wiki primeiro e pode produzir um insight candidate para file-back explícito. Em lint, procura contradições, claims stale, páginas órfãs, links ausentes e gaps.

O schema editorial versionado do workspace define estrutura, proof states, citações e workflows. `index.md` é catálogo orientado ao conteúdo e primeira leitura de uma query; `log.md` é append-only e registra ingestões, file-backs e lint passes. Conteúdo humano fica em overlay separado da área regenerável.

### Verificação e publicação

Valida grounding, proof states, IDs, locators, links, compatibilidade, budgets, conteúdo sensível e consistência global. Publica por staging e troca atômica ou mecanismo equivalente. Esses gates, o Reviewer e a publicação atômica são extensões de segurança do Reverser; não são atribuídos ao padrão LLM Wiki de Karpathy e não retiram da LLM a autoria editorial.

## Workflow de agentes

Os papéis estão em [`../agents/README.md`](../agents/README.md). A divisão em Orchestrator, Investigator e Reviewer é uma escolha do Reverser, não uma prescrição do modelo de Karpathy. O Investigator é a LLM autora e mantenedora da wiki; os papéis trocam artefatos serializáveis e pequenos. Conversa entre agentes é implementação, não estado do produto.

Não se cria um agente separado para planejamento determinístico. Também não se expõe o fornecedor de LLM na interface externa; modelo, prompts e parâmetros pertencem ao registro da execução.

## Escada de consulta e verificação

```text
Reverser LLM Wiki index.md
  -> relevant wiki pages and citations
    -> active workspace revision and canonical IDs
      -> manifest, catalogs and postings
        -> facts and confirmed relations
          -> candidates / unknowns / frontiers
            -> csharp2md Markdown fallback
              -> canonical IDs and source locators
                -> complete projected code
                  -> answer, interpretation or unresolved
```

A wiki é lida primeiro porque concentra a síntese cumulativa. Suas citações são verificadas contra a revisão ativa quando a resposta exige confirmação, quando há risco de staleness ou quando a página declara gap. O fallback seguinte é exclusivamente a Markdown retrieval projection do `csharp2md` e só é acionado após falha estruturada explícita. Se localizar evidência verificável nos dois snapshots, pode sustentar `correlated`; se oferecer apenas narrativa ou similaridade, produz `interpretation`. Ausência nessa projeção nunca prova ausência no pacote.

## Artefatos

- `solution snapshot`: pacote e identidade imutável de uma análise;
- `workspace ledger`: soluções, snapshots ativos e histórico;
- `workspace revision`: conjunto ativo, composição e hashes publicados;
- `baseline map`: visão barata produzida em attach/refresh/detach;
- `wiki schema`: convenções editoriais, proof states, citações e workflows versionados;
- `wiki index`: `index.md` mantido pela LLM e lido primeiro em queries;
- `wiki log`: `log.md` append-only de ingestões, file-backs e lint passes;
- `work item`: objetivo, sementes, escopo, budget e critérios de parada;
- `evidence bundle`: fatos, relações, código e provenance map selecionados;
- `investigation result`: resposta, estados, evidências, cobertura e gaps;
- `insight candidate`: descoberta persistível ainda não promovida;
- `validation report`: gates determinísticos e revisão semântica;
- `wiki revision`: schema, páginas, diagramas, índice e log mantidos cumulativamente pela LLM e publicados após validação.

Timestamps, custo e duração ficam separados do conteúdo determinístico.

## Consistência e falhas

- pacote corrompido, incompatível ou stale: rejeitar antes da investigação factual;
- snapshot novo inválido: manter revisão publicada e registrar staging rejeitado;
- legitimate unknown/frontier: responder parcialmente com cobertura explícita;
- `budget-exhausted`: oferecer continuação, nunca converter em `no-match`;
- evidência ausente: rebaixar para `interpretation`/`unresolved` ou rejeitar;
- possível segredo: bloquear envio e publicação quando redaction não for comprovável;
- falha de agente: retry limitado ou falha do work item, sem publicação parcial silenciosa;
- falha de composição/publicação: preservar última revisão válida.

## Determinismo

São determinísticos: identidades, seleção mecânica, work plans, dependências, links, proof states, validação, composição local e publicação. O append de `log.md` preserva ordem e registros anteriores; metadados temporais voláteis permanecem separados quando prejudicariam a reprodução.

Narrativa e manutenção editorial da LLM não prometem bytes idênticos. Reprodutibilidade significa registrar workspace/revisão, snapshots, evidence bundle, modelo, versão do prompt, parâmetros e decisão do Reviewer, além de impedir variação factual.

## Decisões adiadas

Linguagem/runtime, SDK de agentes, provedor de modelo, cache, paralelismo, UI, hospedagem, busca lexical/vetorial e edição automática de código permanecem adiados até o vertical slice fornecer métricas e requisitos reais.
