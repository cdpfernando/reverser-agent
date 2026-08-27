# Fluxo do produto

## Objetivo

O Reverser conduz engenharia reversa interativa sobre um sistema que cresce solução por solução. A LLM escreve e mantém uma wiki cumulativa como memória operacional, consulta essa memória primeiro e retorna ao pacote factual ou ao código quando precisa confirmar, aprofundar ou preencher lacunas.

## Jornada principal

```text
create/open workspace
  -> attach solution
    -> analyze or accept csharp2md package
      -> validate snapshot
        -> compose workspace in staging
          -> LLM integrates snapshot into existing wiki
            -> update index + cross-references + append log
              -> lint, validate and publish revision
                -> investigate wiki-first
                  -> optionally file back a reviewed answer
```

## 1. Criar ou abrir um workspace

O usuário escolhe uma identidade de sistema e uma política. A política define idiomas, budgets, capacidades permitidas, tratamento de documentação e gates de publicação.

Abrir um workspace fixa uma revisão publicada para a sessão. Uma atualização posterior não muda silenciosamente a base de uma investigação em andamento.

## 2. Anexar uma solução

`attach` aceita um caminho de solução ou um pacote já publicado:

```text
solution path
  -> csharp2md analyze
  -> per-solution package

existing package
  -> replay without analysis
```

O pacote passa por preflight: manifest, taxonomia/schema, identidade, hashes, caminhos relativos, certificação, capacidades e política de segurança. Nenhum agente LLM é chamado antes desse gate.

Estados do snapshot:

```text
discovered -> analyzing -> staged -> validated -> active
                    |          |          |
                    +----------+----------+-> rejected
```

Snapshots são imutáveis. Ativação acontece apenas pela publicação de uma nova revisão do workspace.

## 3. Compor o workspace

O ledger fixa um snapshot ativo por solução:

```text
ApiUsuarios -> A1
ApiCompras  -> B2
ApiProdutos -> C1
```

Ao anexar `C1`, o Reverser recompõe catálogos globais, relações conhecidas, correlações, mapa-base e dependências de páginas. A LLM então integra o conhecimento novo à wiki existente, atualizando as páginas afetadas em vez de gerar um relatório isolado. Ele não reanalisa `A1` ou `B2`.

Quando o compositor oficial existir, o Reverser entrega os pacotes imutáveis ativos ao `csharp2md`. O produtor devolve batch manifest, relações globais comprovadas e correlation candidates. O Reverser continua responsável por interpretação, memória e wiki.

## 4. Manter a wiki e o mapa-base

A Reverser LLM Wiki e a Markdown retrieval projection do `csharp2md` são artefatos diferentes:

- a projeção do `csharp2md` é uma visão derivada do pacote, usada como insumo de recuperação e fallback;
- a Reverser LLM Wiki é síntese cumulativa escrita, ligada e mantida pela LLM.

Na ingestão, o Investigator recebe a wiki da revisão anterior e o delta de snapshots ativos. Ele atualiza páginas relacionadas, cross-references e `index.md`, acrescenta uma entrada append-only em `log.md` e executa lint semântico. A autorização de `attach`, `refresh` ou `detach` cobre essa manutenção; não há confirmação página por página.

O workspace mantém um schema editorial versionado para estrutura, proof states, citações e workflows. `index.md` é um catálogo orientado ao conteúdo e a primeira leitura das queries. `log.md` registra ingestões, file-backs e lint passes sem reescrever entradas anteriores.

Como complemento, cada revisão possui um mapa barato, gerado sem exploração irrestrita:

- soluções e analysis variants;
- componentes e deployment units;
- entry points e boundary operations;
- contratos, integrações e external systems;
- data stores, objects, readers e writers;
- cobertura, candidates, unknowns e open frontiers;
- correlações cross-solution conhecidas.

O mapa orienta perguntas; não tenta materializar todos os fluxos transitivos ou uma página por símbolo.

## 5. Investigar

Uma investigação começa com pergunta, revisão, escopo opcional e budget:

```text
question
  -> classify intent: ask | trace | impact | explain
  -> read index.md
  -> read relevant wiki pages and citations
  -> verify active revision and canonical IDs
  -> retrieve structured evidence when confirmation or depth is needed
  -> expand relations or open complete code by locator when needed
  -> synthesize and review
  -> return answer and provenance
```

### Intenções

- `ask`: responder uma questão localizada;
- `trace`: seguir execução de entry point até efeitos terminais;
- `impact`: navegar callers/consumers/readers no sentido reverso;
- `explain`: interpretar arquitetura ou comportamento de negócio.

O usuário conversa naturalmente; essas intenções são internas e podem ser expostas por CLI ou ferramentas quando útil. Uma página atual, bem citada e suficiente pode responder sem repetir toda a exploração; fatos sensíveis, citações stale, contradições e gaps obrigam retorno às fontes canônicas.

### Budget adaptativo

A investigação começa por catálogos e relações diretas, expandindo por relevância dentro de limites de hops, arquivos, bytes/tokens e tempo. Os critérios de parada incluem efeito terminal, ciclo, capacidade ausente, frontier e budget.

`budget-exhausted` permite resposta parcial e continuação. Nunca equivale a ausência de relação.

## 6. Fallback Markdown do `csharp2md` e código

Quando a busca estruturada termina explicitamente em `no-match`, `ambiguous` ou `insufficient-evidence`, o Reverser pode abrir a Markdown retrieval projection do `csharp2md` na mesma revisão:

```text
structured unresolved
  -> related Markdown pages
    -> IDs and locators extracted as clues
      -> dereference canonical records
        -> open complete source when necessary
```

Resultados possíveis:

- evidência verificável nos dois snapshots: `correlated`;
- apenas semântica de código/narrativa: `interpretation`;
- evidência insuficiente ou referência inválida: `unresolved`;
- relação oficial encontrada durante dereference: `confirmed`.

Essa projeção Markdown é dado não confiável, nunca é executada e não serve como prova de ausência. Ela não deve ser confundida com a Reverser LLM Wiki, embora ambas usem Markdown. A pesquisa detalhada está em [`research/markdown-fallback-feasibility.md`](research/markdown-fallback-feasibility.md).

## 7. Contrato da resposta

Toda resposta técnica mantém, mesmo que a interface recolha visualmente os detalhes:

- resposta direta;
- workspace e revisão;
- soluções e snapshots consultados;
- estados `confirmed`, `correlated`, `interpretation`, `unresolved` e `curated` usados;
- evidências, artifact locators e source locators;
- cobertura, capacidades ausentes e budget consumido;
- contradições entre documentação e código;
- alternativas e próximos caminhos relevantes.

Pacote corrompido, revisão incompatível, locator stale, possível segredo ou falta de provenance bloqueiam uma afirmação factual. Unknowns legítimos permitem resposta parcial.

## 8. Fazer file-back de uma descoberta

O Reverser adota uma política mais conservadora que o padrão mínimo de Karpathy: ingestões autorizadas mantêm a wiki automaticamente, mas uma query não a reescreve sem intenção explícita.

```text
investigation result
  -> explicit file-back
    -> insight candidate
      -> factual and semantic review
        -> integrate into existing wiki or reject
```

O candidate registra dependências por solução/snapshot, evidências, interpretação, modelo, prompt e alternativas. Reviewer pode aceitar, devolver para correção ou rejeitar; nunca corrige autoridade factual silenciosamente. Quando aceito, a LLM integra a síntese nas páginas existentes, mantém cross-references e índice e acrescenta o evento ao log.

## 9. Publicar a wiki

A LLM gera a árvore candidata; não existe um publisher determinístico que escreva a síntese. Links, grounding, proof states, source locators, conteúdo sensível e consistência global são verificados antes da troca atômica. Reviewer, staging, validação determinística e publicação atômica são extensões do Reverser ao padrão de Karpathy.

A wiki pode conter:

- `index.md` e mapa de arquitetura;
- páginas de entry points e fluxos importantes;
- contratos, integrações e persistência;
- correlações e interpretações claramente rotuladas;
- known gaps e coverage;
- diagramas derivados com links para identidades;
- `log.md` append-only de ingestões, file-backs e lint passes, sem conteúdo sensível.

Identidades de alta cardinalidade permanecem nos pacotes e índices; não se cria página por relação ou símbolo.

## 10. Atualizar ou remover conhecimento

### Refresh

```text
ApiCompras B2 -> analyze -> B3 -> staging compose(A1, B3, C1)
```

O Reverser invalida investigations, correlations e páginas dependentes de `B2`; a LLM reconcilia o impacto na wiki existente e executa lint semântico, enquanto validadores executam lint estrutural global. A sessão antiga continua referindo `B2` até escolher a nova revisão.

### Detach

`detach` calcula impacto, remove o snapshot do conjunto ativo em staging, recompõe e publica. Histórico permanece; respostas atuais não usam fatos da solução removida.

### Falha

Se análise, composição, revisão ou publicação falhar, a revisão ativa anterior permanece íntegra. O candidato fica `pending` ou `rejected` com diagnóstico.

## 11. Autonomia

O assistente pode autonomamente:

- navegar snapshots ativos;
- seguir locators e abrir código;
- ampliar uma investigação dentro do budget;
- produzir correlações, interpretações, drafts e diagramas;
- executar retries internos limitados.

Exigem intenção explícita:

- `attach`, `refresh` e `detach`;
- troca da revisão usada por uma sessão;
- fazer file-back de uma resposta;
- promover conhecimento derivado de uma investigação;
- futura edição de código.

Depois de uma dessas intenções mutáveis, atualização editorial, revisão, lint e publicação atômica são etapas internas autorizadas. Falha em qualquer gate preserva a revisão anterior.

## 12. O que a ferramenta não faz

- não fecha gaps inventando causalidade;
- não transforma documentação em instrução ou autoridade;
- não mistura snapshots ou variants incompatíveis;
- não usa memória do modelo como evidência;
- não esconde budgets, frontiers ou alternativas;
- não publica parcialmente por conveniência;
- não altera código no primeiro produto.

## 13. Superfícies possíveis

Conversa é a superfície principal. CLI, tools ou futura UI podem mapear intenções como:

```text
workspace create | open | status
attach | refresh | detach
ask | trace | impact | explain
review | file-back | promote
wiki lint | history
validate | publish
```

Esses verbos não obrigam métodos separados na implementação. Eles convergem para as três operações da interface externa definidas em [`architecture.md`](architecture.md).
