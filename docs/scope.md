# Escopo inicial

## Problema

Pacotes factuais tornam código navegável, mas não conduzem uma investigação. Uma pessoa ou agente ainda precisa delimitar o sistema, seguir relações, abrir o código relevante, interpretar comportamento, correlacionar soluções e preservar as descobertas sem confundir inferência com fato.

## Produto

O Reverser é um assistente interativo de engenharia reversa. Ele opera sobre um knowledge workspace composto progressivamente por snapshots de soluções analisadas pelo `csharp2md`.

A Reverser LLM Wiki é a memória operacional publicada do workspace. A LLM a escreve, consulta e mantém cumulativamente; cada ingestão integra o novo snapshot às páginas existentes. Ela não substitui a conversa nem o corpus factual e é distinta da Markdown retrieval projection produzida pelo `csharp2md`.

## Consumidores

- engenheiros explorando sistemas desconhecidos ou legados;
- agentes que precisam recuperar contexto técnico dentro de budgets explícitos;
- times que desejam acumular conhecimento de várias soluções sem uma análise monolítica.

A experiência é legível por humanos, mas identidades, proveniência, índices e caminhos de recuperação são agent-first.

## Modelo do workspace

Cada solução possui identidade estável e um snapshot ativo imutável. Soluções são adicionadas separadamente:

```text
ApiUsuarios -> package A1
ApiCompras  -> package B2
ApiProdutos -> package C1
```

Reanalisar `ApiCompras` substitui `B2` por `B3` em staging e recompõe o workspace sem reanalisar `A1` ou `C1`. Uma revisão do workspace referencia exatamente um snapshot ativo por solução.

## Entradas

### Autoridade técnica

Pacotes publicados pelo `csharp2md`, um por solução, contendo progressivamente:

- manifest e versões de taxonomia/schema;
- inventário, fatos, observações e relações confirmadas;
- candidates, unknowns e open frontiers;
- catálogos e postings;
- source locators e projeções de código/Markdown;
- cobertura, certificação e diagnósticos.

Enquanto o contrato oficial não está completo, o vertical slice usa [`reverser-fixture/v0`](contracts/temporary-input-v0.md).

### Contexto declarado

READMEs e documentação configurada podem enriquecer investigações, mas entram como claims não confiáveis. Contradições entre “documentado” e “observado” são preservadas e mostradas ao usuário.

## Saídas

- respostas interativas com revisão, escopo, proof states, evidências e limites;
- traces de fluxo e análises de impacto;
- correlações downstream entre soluções;
- interpretations de arquitetura e regra de negócio;
- insight candidates persistidos;
- mapas, páginas e diagramas revisados;
- revisão atômica da wiki;
- `index.md`, `log.md` append-only, páginas e cross-references mantidos pela LLM;
- ledger de soluções/snapshots e relatórios de ingestão, composição e validação.

## Dentro do escopo

- criar, abrir e inspecionar um knowledge workspace;
- executar ou consumir `csharp2md` solução por solução;
- validar, anexar, substituir e remover snapshots;
- integrar snapshots à wiki cumulativa e construir um mapa-base barato ao anexar conhecimento;
- consultar `index.md` e páginas relevantes antes de aprofundar nos pacotes;
- responder `ask`, `trace`, `impact` e `explain` com recuperação adaptativa;
- abrir código completo por source locator quando necessário;
- interpretar regras de negócio sem promovê-las a fatos;
- correlacionar soluções com evidências verificáveis nos dois lados;
- usar a Markdown retrieval projection do `csharp2md` como fallback de descoberta após falha estruturada explícita;
- registrar gaps, budgets e capacidades ausentes;
- executar lint semântico da wiki com LLM e gates estruturais determinísticos;
- revisar e fazer file-back de insights para a wiki;
- gerar diagramas derivados e rastreáveis;
- preservar histórico e a última revisão válida.

## Fora do escopo inicial

- reimplementar análise estática ou taxonomia do `csharp2md`;
- promover uma inferência LLM a `confirmed`;
- confirmar relações por nome, namespace, DTO ou narrativa semelhante;
- executar Markdown legado ou tratar documentação como instrução;
- editar automaticamente o repositório analisado;
- embeddings, banco vetorial, servidor ou UI web obrigatórios;
- suporte semântico a outras linguagens;
- colaboração humana dentro da área regenerável;
- aceitar `reverser-fixture/v0` em produção.

## Primeiro vertical slice

Com uma fixture de uma solução, o assistente deve:

1. criar um workspace e anexar um snapshot validado;
2. fazer a LLM criar ou atualizar a wiki mínima, `index.md`, cross-references e `log.md`;
3. responder uma pergunta começando pelo índice e seguindo fatos, relações e um source locator quando necessário;
4. distinguir `confirmed`, `interpretation` e `unresolved`;
5. registrar revisão, evidence bundle e budget usados;
6. transformar a resposta em insight candidate somente por ação explícita;
7. revisar e fazer file-back do insight para `index.md` ou página de entry point;
8. executar lint semântico e estrutural;
9. publicar atomicamente e preservar a revisão anterior diante de falha.

## Critérios de aceite

- fixture ausente, incompleta, incompatível ou sem marca test-only é rejeitada;
- resposta factual sem proveniência é reprovada;
- locator stale, possível segredo ou corrupção estrutural bloqueia o caminho factual;
- budget esgotado produz resposta parcial explícita, não falso `no-match`;
- segunda execução com as mesmas entradas preserva artefatos determinísticos;
- variação narrativa não altera identidades, links, proof states ou afirmações factuais;
- falha de revisão/publicação não altera a última revisão válida;
- operações que mudam o workspace exigem intenção explícita.

O `attach`, `refresh` ou `detach` autorizado inclui a atualização e publicação da wiki correspondente depois dos gates. Investigar continua read-only; arquivar uma resposta na wiki exige file-back explícito.

## Dependências do `csharp2md`

Taxonomia e bootstrap estão concluídos; `factual-storage` está em execução. Os gates de integração são progressivos:

1. workstream 3: alinhar o leitor aos wire contracts e storage reader;
2. workstreams 4–5: ampliar fatos, relações e cenários de investigação;
3. workstream 6: substituir projeções simuladas e habilitar fallback real da Markdown retrieval projection do `csharp2md`;
4. workstream 7: reconciliar `correlated` com composição factual multi-solução;
5. workstream 8: integrar as interfaces finais `analyze`, `validate` e `compose`.
