# Roadmap inicial

O roadmap entrega primeiro um assistente útil sobre uma solução e depois amplia composição e integração. A interface externa permanece centrada em alterar workspace, investigar e promover conhecimento; ingestão, query, file-back e lint operam sobre a mesma LLM Wiki cumulativa.

## Fase 0 — contrato e cenários (atual)

- consolidar produto, fluxo, arquitetura e papéis;
- materializar `reverser-fixture/v0` sobre uma solução sintética;
- especificar schema editorial, `index.md`, páginas e `log.md` append-only da wiki mínima;
- especificar workspace revision, work item, evidence bundle, investigation result e insight candidate;
- criar casos positivos e negativos de proveniência, budget e publicação.

**Gate:** a fixture representa snapshot, entry point, relação, source locator, unresolved e uma pergunta completa, sem expor seu layout ao workflow.

## Fase 1 — assistente de uma solução

- criar/open workspace e anexar uma fixture;
- fazer a LLM ingerir o snapshot na wiki existente e manter índice, links e log;
- construir mapa-base;
- executar `ask`, `trace` e `explain` começando por `index.md` sobre uma revisão fixa;
- abrir código completo por locator dentro de budget;
- produzir resposta revisada com estados e proveniência;
- fazer file-back explícito, revisar e promover um insight;
- executar lint semântico da LLM e lint estrutural determinístico;
- publicar wiki mínima atomicamente.

**Gate:** todos os critérios do primeiro vertical slice em [`scope.md`](scope.md), incluindo preservação da última revisão válida.

## Fase 2 — workspace multi-solução

- anexar fixtures/pacotes independentes solução por solução;
- implementar ledger, revisão, `attach`, `refresh` e `detach`;
- integrar cada solução à wiki cumulativa sem substituir páginas não afetadas por um relatório novo;
- correlacionar HTTP, gRPC, mensageria e contratos com evidência nos dois lados;
- responder sobre todo o workspace com filtros por solução/snapshot;
- invalidar dependências afetadas e executar lint global;
- usar rebuild completo como oráculo de equivalência.

**Gate:** adicionar `ApiProdutos` enriquece um workspace existente sem reanalisar `ApiUsuarios` ou `ApiCompras`; falha de staging preserva a revisão anterior.

## Fase 3 — integração progressiva com `csharp2md`

- alinhar o leitor ao wire contract do workstream 3;
- ampliar cenários conforme workstreams 4 e 5A–5D;
- substituir source/Markdown simulados pelas retrieval projections do workstream 6;
- habilitar fallback da Markdown retrieval projection do `csharp2md` somente após falha estruturada explícita;
- medir hops, arquivos, bytes/tokens, relevância e projection loss.

**Gate:** investigar uma solução real sem leitura irrestrita do repositório-fonte e sem consumir qualquer contrato legado.

## Fase 4 — composição oficial e manutenção

- entregar snapshots ativos ao compositor multi-solução dos workstreams 7–8;
- reconciliar `correlated` com `confirmed`, unresolved candidates e rejeições;
- integrar as interfaces finais `analyze`, `validate` e `compose`;
- manter histórico de snapshots e wiki revisions;
- fazer a LLM reconciliar somente páginas/insights afetados, com lint semântico e estrutural global;
- separar conteúdo regenerável de curadoria humana.

**Gate:** atualização seletiva é semanticamente equivalente a rebuild completo e nenhuma correlação muda de estado silenciosamente.

## Fase 5 — superfícies opcionais

- UI, hospedagem e colaboração;
- busca lexical ou vetorial baseada em métricas;
- agendamento/CI e notificações;
- planos de mudança e futura edição de código em módulo autorizado separado;
- fontes externas adicionais com provenance próprio.

## Próximas tarefas concretas

1. Criar fixture v0 de uma solução com pergunta, ingestão da wiki e resposta esperada.
2. Especificar os artefatos internos do vertical slice, incluindo schema, `index.md` e `log.md`.
3. Escolher linguagem/runtime com um spike de ingestão, investigação e publicação atômica.
4. Implementar `create/open`, `attach`, manutenção cumulativa da wiki e uma investigação `trace` wiki-first completa.
5. Implementar file-back/review/promote para uma página de entry point.
6. Acrescentar uma segunda fixture para provar composição sem reanálise.
7. Acompanhar o workstream 3 e substituir o reader quando seu contrato estiver disponível.
