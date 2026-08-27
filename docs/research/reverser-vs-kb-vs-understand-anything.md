# Reverser vs. `kb` vs. Understand Anything

Pesquisa realizada em 2026-08-25 contra a documentação normativa local do Reverser e as fontes primárias atuais dos repositórios [`compozy/kb`](https://github.com/compozy/kb) e [`Egonex-AI/Understand-Anything`](https://github.com/Egonex-AI/Understand-Anything). Planos históricos, issues e comentários de terceiros não foram usados como definição do comportamento atual.

## Conclusão executiva

Os três projetos se sobrepõem, mas não são substitutos diretos:

- o **Reverser** é um projeto ainda documental para engenharia reversa auditável de sistemas C#/.NET compostos solução por solução; o pacote factual do `csharp2md` continua sendo a autoridade, enquanto a LLM mantém uma wiki cumulativa e o produto responde investigações com estados de conhecimento, evidência e revisão fixa;
- o **`kb`** é uma CLI funcional e generalista para construir knowledge bases temáticas em Markdown; ela cuida da ingestão, estrutura, análise de alguns codebases, lint e busca, enquanto a compilação semântica da wiki permanece no agente LLM;
- o **Understand Anything** é um produto funcional de análise e visualização de projetos; seu centro é um knowledge graph JSON explorável por dashboard, chat e ferramentas especializadas, não uma wiki editorial cumulativa.

O Reverser adota do `kb` o ciclo `ingest -> compile -> query -> lint` e a autoria LLM da wiki, e do Understand Anything a divisão entre extração determinística e interpretação LLM. Sua diferenciação planejada está na autoridade factual externa, proof states, proveniência, budgets, snapshots imutáveis, composição multi-solução e publicação transacional. Seu maior gap não é conceitual: é de execução. Hoje não há runner, fixture, schema executável, teste ou wiki gerada no repositório; o estado atual é contrato e roadmap ([README local](../../README.md#estado), [roadmap local](../roadmap.md#fase-0--contrato-e-cenários-atual)).

## Comparação

| Dimensão | Reverser atual | `compozy/kb` atual | Understand Anything atual |
| --- | --- | --- | --- |
| **Estado** | Projeto documental no estágio de contrato e primeiro vertical slice; não há implementação executável no repositório ([estado](../../README.md#estado), [fase 0](../roadmap.md#fase-0--contrato-e-cenários-atual)). | CLI Go distribuída como binário/npm/Homebrew, acompanhada por uma skill para as fases LLM ([README](https://github.com/compozy/kb#kb)). | Plugin e dashboard executáveis; `/understand` produz o grafo consumido pela UI ([README](https://github.com/Egonex-AI/Understand-Anything#quick-start), [`/understand`](https://github.com/Egonex-AI/Understand-Anything/blob/main/understand-anything-plugin/skills/understand/SKILL.md#understand)). |
| **Propósito** | Assistente interativo de engenharia reversa C#/.NET, com memória operacional e investigações `ask`, `trace`, `impact` e `explain` ([escopo](../scope.md#produto), [fluxo](../product-flow.md#5-investigar)). | Knowledge base temática generalista; codebases são apenas um dos tipos de fonte, ao lado de URLs, arquivos, vídeos e bookmarks ([features](https://github.com/compozy/kb#features)). | Transformar codebase, docs ou knowledge base em knowledge graph visual para arquitetura, domínio, onboarding, busca e impacto ([README](https://github.com/Egonex-AI/Understand-Anything#features)). |
| **Unidade de ingestão** | Um pacote/snapshot imutável do `csharp2md` por solução; `attach`, `refresh` e `detach` alteram uma revisão do workspace ([modelo](../scope.md#modelo-do-workspace)). | Uma fonte dentro de um tópico: URL, arquivo, vídeo, bookmarks ou um codebase/repositório; fontes são normalizadas para Markdown em `raw/` ([ingest](https://github.com/compozy/kb#kb-ingest), [pattern overview](https://github.com/compozy/kb/blob/main/skills/kb/SKILL.md#pattern-overview)). | Um diretório de projeto para `/understand`; arquivos são os lotes internos. `/understand-knowledge` recebe uma LLM Wiki já existente como diretório e a recompila integralmente em grafo ([skill principal](https://github.com/Egonex-AI/Understand-Anything/blob/main/understand-anything-plugin/skills/understand/SKILL.md#understand), [`/understand-knowledge`](https://github.com/Egonex-AI/Understand-Anything/blob/main/understand-anything-plugin/skills/understand-knowledge/SKILL.md#understand-knowledge)). |
| **Autoridade** | O pacote factual do `csharp2md` é a autoridade técnica; wiki, respostas, diagramas e Markdown de retrieval são derivados. `confirmed` só pode vir do produtor ([autoridade](../architecture.md#autoridade)). | `raw/` é o staging imutável da compilação, mas o procedimento de query trata a wiki como a fonte de verdade operacional e só retorna ao `raw/` em caso de ambiguidade ([pattern overview](https://github.com/compozy/kb/blob/main/skills/kb/SKILL.md#pattern-overview), [query](https://github.com/compozy/kb/blob/main/skills/kb/SKILL.md#procedure-3-query-the-wiki-and-file-back-the-answer)). | Código e estrutura extraída alimentam um único grafo: Tree-sitter fornece estrutura determinística e LLM acrescenta semântica. As queries checam frescor por commit e deep dives podem abrir o source, mas não há taxonomia formal de autoridade por claim ([híbrido](https://github.com/Egonex-AI/Understand-Anything#tree-sitter--llm-hybrid), [`understand-explain`](https://github.com/Egonex-AI/Understand-Anything/blob/main/understand-anything-plugin/skills/understand-explain/SKILL.md)). |
| **Autoria e manutenção da wiki** | A LLM/Investigator é autora editorial de páginas, cross-references, `index.md` e `log.md`; validadores não escrevem síntese ([wiki](../architecture.md#reverser-llm-wiki), [Investigator](../../agents/README.md#investigator)). | A LLM lê `raw/`, compila e atualiza artigos, backlinks e índices. A CLI executa scaffolding, ingestão, logs mecânicos, lint estrutural, indexação e promoção OKF ([visão geral](https://github.com/compozy/kb/blob/main/skills/kb/SKILL.md#pattern-overview), [compilação](https://github.com/compozy/kb/blob/main/skills/kb/SKILL.md#procedure-1-compile-a-wiki-article)). | O fluxo de codebase não mantém uma LLM Wiki: agentes geram e atualizam o grafo JSON. O modo knowledge analisa uma wiki existente e extrai artigos, entidades, claims e relações para um grafo; isso não equivale a possuir editorialmente a wiki de origem ([knowledge base feature](https://github.com/Egonex-AI/Understand-Anything#analyze-knowledge-bases)). |
| **Consulta** | Wiki-first: `index.md`, páginas e citações; fatos sensíveis, conteúdo stale e gaps voltam à revisão, pacote e código. Respostas registram budget e limites ([escada](../architecture.md#escada-de-consulta-e-verificação)). | Concept Index primeiro; QMD/grep em escala maior; segue um hop de wikilinks e opcionalmente verifica `raw/`. Toda resposta é salva em `outputs/queries/` e síntese durável pode ser promovida à wiki ([query e file-back](https://github.com/compozy/kb/blob/main/skills/kb/SKILL.md#procedure-3-query-the-wiki-and-file-back-the-answer)). | `/understand-chat` pesquisa seletivamente `knowledge-graph.json`; `/understand-explain` combina vizinhança do grafo com leitura do source; dashboard fornece exploração visual e search ([chat](https://github.com/Egonex-AI/Understand-Anything/blob/main/understand-anything-plugin/skills/understand-chat/SKILL.md), [explain](https://github.com/Egonex-AI/Understand-Anything/blob/main/understand-anything-plugin/skills/understand-explain/SKILL.md)). |
| **Artefato central** | Workspace revision + ledger de snapshots + mapa-base + LLM Wiki Markdown versionada; evidence bundles e reports sustentam respostas e publicação ([artefatos](../architecture.md#artefatos)). | Vault Markdown por tópico: `raw/`, `wiki/`, `outputs/`, `bases/`, `CLAUDE.md`, `AGENTS.md` e `log.md`; opcionalmente coleção QMD e bundle OKF portátil ([layout](https://github.com/compozy/kb#what-gets-generated), [OKF](https://github.com/compozy/kb/blob/main/skills/kb/SKILL.md#okf-dual-mode)). | `.ua/knowledge-graph.json`, `meta.json`, fingerprints e artefatos intermediários; opcional `domain-graph.json`; dashboard é a superfície principal ([grafo](https://github.com/Egonex-AI/Understand-Anything/blob/main/understand-anything-plugin/agents/knowledge-graph-guide.md#graph-structure), [save](https://github.com/Egonex-AI/Understand-Anything/blob/main/understand-anything-plugin/skills/understand/SKILL.md#phase-7--save)). |
| **Análise de código** | Deliberadamente delegada ao `csharp2md`; o Reverser não reimplementa C#/.NET, taxonomia, resolução de símbolos ou prova de relações ([fora do escopo](../scope.md#fora-do-escopo-inicial)). | Implementada na própria CLI para TypeScript/TSX, JavaScript/JSX e Go; produz notas por arquivo/símbolo, relações e métricas como complexidade, coupling, blast radius e dead code ([codebase analysis](https://github.com/compozy/kb#codebase-analysis), [linguagens](https://github.com/compozy/kb#supported-languages-codebase-analysis)). | Implementada no próprio produto com Tree-sitter e LLM, produzindo nós de arquivo/função/classe, relações, layers e tours para múltiplas linguagens/frameworks ([híbrido](https://github.com/Egonex-AI/Understand-Anything#tree-sitter--llm-hybrid), [graph structure](https://github.com/Egonex-AI/Understand-Anything/blob/main/understand-anything-plugin/agents/knowledge-graph-guide.md#graph-structure)). |
| **Arquitetura de agentes** | Três papéis planejados: Orchestrator, Investigator e Reviewer. O Investigator responde e mantém a wiki; módulos determinísticos cuidam de ingestão, seleção, validação e persistência ([agentes](../../agents/README.md)). | Não prescreve uma equipe fixa. A CLI cobre o workflow não-LLM e a skill orienta o agente autor, podendo coordenar skills companheiras para Markdown/Obsidian ([README](https://github.com/compozy/kb#kb), [related skills](https://github.com/compozy/kb/blob/main/skills/kb/SKILL.md#related-skills)). | Pipeline multiagente especializado com scanner, file analyzers paralelos, análises/review de assembly e arquitetura, tour builder e reviewer opcional. O modo knowledge usa parser determinístico e `article-analyzer` em lotes ([pipeline](https://github.com/Egonex-AI/Understand-Anything#multi-agent-pipeline), [`/understand`](https://github.com/Egonex-AI/Understand-Anything/blob/main/understand-anything-plugin/skills/understand/SKILL.md#phase-2--analyze), [`/understand-knowledge`](https://github.com/Egonex-AI/Understand-Anything/blob/main/understand-anything-plugin/skills/understand-knowledge/SKILL.md#phase-3-analyze)). |
| **Incrementalidade** | Planejada em dois níveis: troca somente o snapshot da solução alterada e reconcilia somente conhecimento dependente; rebuild completo será oráculo de equivalência ([refresh](../product-flow.md#10-atualizar-ou-remover-conhecimento), [fase 4](../roadmap.md#fase-4--composição-oficial-e-manutenção)). | Fontes ingeridas acumulam; snapshots de codebase e áreas gerenciadas da wiki são refrescados, enquanto adições e `outputs/` do usuário são preservados. A documentação pública não promete equivalência transacional entre atualização seletiva e rebuild ([layout](https://github.com/compozy/kb#what-gets-generated)). | No modo código, detecta mudanças desde o commit anterior, reanalisa arquivos alterados, remove nós/edges antigos e os funde com o grafo preservado; arquitetura é recalculada sobre o conjunto completo e fingerprints sustentam updates futuros. O modo knowledge atual recompila a wiki inteira ([incremental path](https://github.com/Egonex-AI/Understand-Anything/blob/main/understand-anything-plugin/skills/understand/SKILL.md#incremental-update-path), [`/understand-knowledge`](https://github.com/Egonex-AI/Understand-Anything/blob/main/understand-anything-plugin/skills/understand-knowledge/SKILL.md)). |
| **Revisão, validação e publicação** | Reviewer semântico, gates determinísticos, staging e troca atômica; qualquer falha preserva a última revisão válida. Ainda é contrato, não implementação ([verificação](../architecture.md#verificação-e-publicação)). | `kb lint` valida frontmatter, links, órfãos, fontes e staleness; a skill acrescenta lint/heal semântico e pede diff antes de corrigir. OKF possui conformance check. Não há contrato publicado de revisão imutável ou troca atômica do vault inteiro ([lint](https://github.com/compozy/kb#kb-lint), [lint e heal](https://github.com/compozy/kb/blob/main/skills/kb/SKILL.md#procedure-4-lint-and-heal)). | Merge/normalização determinística, assemble-reviewer e validação estrutural inline por default; `--review` ativa reviewer LLM completo. Issues remanescentes podem salvar o grafo com warnings e impedir o dashboard, portanto o gate é mais permissivo que o contrato planejado do Reverser ([review](https://github.com/Egonex-AI/Understand-Anything/blob/main/understand-anything-plugin/skills/understand/SKILL.md#phase-6--review)). |
| **Proveniência** | Claim rastreável identifica workspace/revisão, soluções, snapshots, proof state, evidências e locators; execução registra evidence bundle, modelo, prompt e parâmetros ([contrato da resposta](../product-flow.md#7-contrato-da-resposta), [determinismo](../architecture.md#determinismo)). | Artigos possuem `sources` e wikilinks; Source Index fornece índice reverso e query output usa `informed_by`. É boa proveniência documental, mas não oferece proof states ou exigência equivalente de evidência canônica por relação ([índices](https://github.com/compozy/kb/blob/main/skills/kb/SKILL.md#procedure-2-maintain-topic-indexes), [file-back](https://github.com/compozy/kb/blob/main/skills/kb/SKILL.md#procedure-3-query-the-wiki-and-file-back-the-answer)). | Nós carregam IDs, `filePath` e, quando aplicável, `lineRange`; o grafo registra `gitCommitHash` e edges possuem peso. Isso localiza código e detecta staleness, mas estrutura determinística e semântica LLM coexistem sem os cinco estados epistemológicos do Reverser ([graph reference](https://github.com/Egonex-AI/Understand-Anything/blob/main/understand-anything-plugin/skills/understand-chat/SKILL.md#graph-structure-reference), [grafo de exemplo](https://github.com/Egonex-AI/Understand-Anything/blob/main/understand-anything-plugin/packages/dashboard/public/knowledge-graph.json)). |
| **Multi-solução / multi-repositório** | Requisito central: ledger com um snapshot ativo por solução, correlações entre pacotes e revisão que compõe `ApiUsuarios`, `ApiCompras`, `ApiProdutos` etc. sem reanalisar snapshots preservados ([workspace](../architecture.md#knowledge-workspace)). | Um vault pode conter vários tópicos, queries podem cruzá-los e um codebase pode ser uma fonte de um tópico. O README posiciona a análise de código como entendimento profundo de um único repositório; não existe composição formal de identidades ou relações entre snapshots independentes ([multi-topic](https://github.com/compozy/kb/blob/main/skills/kb/SKILL.md#pattern-overview), [why kb](https://github.com/compozy/kb#why-kb)). | A unidade normal é um projeto/diretório. A skill pode fundir subdomain graphs e analisar subdiretórios, o que ajuda em monorepos, mas não define ledger, revisão ou proof states para compor soluções/repositórios independentes. Essa última comparação é uma inferência a partir do workflow publicado ([subdomain merge](https://github.com/Egonex-AI/Understand-Anything/blob/main/understand-anything-plugin/skills/understand/SKILL.md#phase-0--pre-flight)). |

## Onde o Reverser é mais forte por desenho

### 1. Fronteira clara entre análise e síntese

O `kb` e o Understand Anything analisam source code dentro do próprio produto. O Reverser estabelece uma divisão mais profunda: `csharp2md` extrai e prova; Reverser recupera, interpreta, compõe e mantém memória. Isso reduz duplicação de análise C# e permite evoluir a experiência sem redefinir o que conta como fato.

O custo é dependência forte do contrato e roadmap do produtor. Até o pacote oficial estabilizar, [`reverser-fixture/v0`](../contracts/temporary-input-v0.md) só consegue provar o workflow downstream, não a capacidade real do `csharp2md`.

### 2. Disciplina epistemológica

Nem `kb` nem Understand Anything expõem uma separação equivalente entre `confirmed`, `correlated`, `interpretation`, `unresolved` e `curated`. O Reverser também distingue `budget-exhausted` de `no-match` e exige evidência nos dois pacotes para uma correlação.

Essa disciplina é valiosa em engenharia reversa, sobretudo quando uma explicação de negócio combina fatos mecânicos e leitura semântica. Em contrapartida, aumenta o custo de schema, review, UX e testes: o produto precisa preservar essas diferenças sem tornar cada resposta ilegível.

### 3. Composição transacional multi-solução

O diferencial mais nítido do Reverser é a unidade `SolutionSnapshot` dentro de uma `WorkspaceRevision`. `kb` oferece multi-topic knowledge management; Understand Anything oferece análise de um projeto e merge de subdomínios. Nenhum dos dois descreve o mesmo lifecycle de anexar, substituir e remover soluções independentes, recompor relações e manter a revisão anterior quando a mudança falha.

### 4. Publicação defensiva

Staging, validação global e preservação da última revisão válida são mais rigorosos que os fluxos publicados das referências. Isso é apropriado se a wiki for memória compartilhada e acumulativa: uma atualização ruim de `ApiProdutos` não deve inutilizar conhecimento válido de `ApiUsuarios`.

O risco é transformar cada ingestão em uma transação cara, com mais chamadas LLM e maior latência. O primeiro slice deve medir esse custo antes de ampliar papéis, banco, embeddings ou UI.

## Onde as referências estão à frente

### `kb`

- ingestion adapters reais para documentos, web, mídia, bookmarks e codebases;
- uma CLI operacional, layout de vault, schemas editoriais, logs e lint;
- query file-back e manutenção de índices já codificados como procedimento;
- busca híbrida opcional via QMD e exportação portátil via OKF;
- separação prática entre conteúdo gerenciado e `outputs/` preservados.

O Reverser já adotou conceitualmente parte desse lifecycle, mas ainda precisa materializar seus artefatos equivalentes. A principal diferença a preservar é que a wiki do Reverser não pode se tornar a autoridade factual somente porque a consulta começa por ela.

### Understand Anything

- pipeline híbrido real de Tree-sitter, scripts determinísticos e agentes especializados;
- fan-out paralelo por lotes e update incremental por arquivo;
- grafo navegável, dashboard, tours, busca, diff impact e domain view;
- merge/normalização e validação concreta de IDs, edges, layers e referências;
- suporte mais amplo a linguagens e tipos de projeto.

O Reverser não deve copiar o analisador, pois isso violaria sua fronteira com `csharp2md`. As ideias reutilizáveis são operacionais: fingerprints, invalidação seletiva, merge determinístico, artefatos intermediários pequenos e uma superfície visual futura sobre o conhecimento já comprovado.

## Gaps prioritários do Reverser

1. **Não existe vertical slice executável.** O próximo avanço precisa produzir fixture, schema editorial mínimo, reader, uma query wiki-first, Reviewer e publicação recuperável; sem isso, as garantias continuam não testadas.
2. **A granularidade de dependência da wiki ainda precisa de contrato.** Para reconciliar somente páginas afetadas, cada página/insight deve declarar dependências por solução, snapshot, IDs e evidence bundle.
3. **O modelo de update da LLM precisa de prova de equivalência.** A documentação exige atualização seletiva semanticamente equivalente ao rebuild, mas ainda não define comparação, tolerâncias narrativas e diagnóstico de drift.
4. **O schema de proveniência precisa ser utilizável por humanos.** O Reverser promete mais detalhe que as referências; falta demonstrar como a UI/Markdown mantém evidência acessível sem poluir a leitura principal.
5. **Busca ainda não foi validada em escala.** Adiar embeddings é razoável; o primeiro slice deve medir quando `index.md`, links e catálogos deixam de bastar antes de adotar QMD ou banco vetorial.
6. **Não há superfície de exploração pronta.** Chat é suficiente para o MVP, mas o valor visual de graph/tour/diff do Understand Anything é um gap de produto relevante depois que o contrato factual estiver funcionando.
7. **O escopo de fontes é estreito de propósito.** Diferente do `kb`, documentos externos, tickets e mídia não entram no MVP. A escolha protege a autoridade técnica, mas limita a reconstrução de contexto organizacional.

## Recomendações

### Adotar agora

- do `kb`: separação explícita entre fonte, wiki regenerável e outputs preservados; `index.md` primeiro; Source Index reverso; file-back; lint estrutural + semântico;
- do Understand Anything: pipeline híbrido, IDs normalizados, merge determinístico, detecção de dangling references, update por delta e artefatos intermediários serializáveis;
- do próprio Reverser: manter `csharp2md` como autoridade, os cinco estados de conhecimento e a composição por snapshots — são os elementos que justificam um produto separado.

### Adiar

- dashboard, tours, graph explorer, embeddings/QMD, banco e adapters multi-source;
- novas personas de agentes antes de o trio atual provar handoffs distintos;
- análise direta de C# no Reverser;
- uma página por símbolo ou materialização do grafo de alta cardinalidade em Markdown.

## Veredito

O Reverser deve ser entendido como **uma especialização de LLM Wiki para engenharia reversa multi-solução com autoridade externa e auditoria forte**, não como clone de qualquer uma das referências.

- Se o objetivo fosse apenas criar uma wiki Markdown sobre fontes variadas, `kb` já está mais maduro e amplo.
- Se o objetivo fosse visualizar rapidamente a estrutura de um repositório, Understand Anything já oferece a experiência mais completa.
- O espaço próprio do Reverser é responder “como esse sistema distribuído realmente funciona e em qual evidência cada conclusão se apoia?”, acumulando solução por solução sem apagar diferenças entre prova, correlação e interpretação.

Essa vantagem só se tornará real quando o primeiro vertical slice demonstrar, com artefatos e testes, o percurso completo `snapshot validado -> ingestão LLM -> query wiki-first -> evidência -> review -> publicação recuperável`.
