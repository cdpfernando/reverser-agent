# Referências de arquitetura para a LLM wiki

> **Nota posterior:** a leitura da [fonte oficial de Karpathy](karpathy-llm-wiki-model.md) refinou este desenho. Na arquitetura normativa do Reverser, a LLM possui, escreve, consulta e mantém a wiki; validação, Reviewer, staging e publicação atômica são extensões próprias de segurança, não elementos prescritos pelo padrão Karpathy.

Pesquisa realizada em 2026-08-25, usando somente fontes primárias dos repositórios [Egonex-AI/Understand-Anything](https://github.com/Egonex-AI/Understand-Anything) e [compozy/kb](https://github.com/compozy/kb), mais a arquitetura normativa do `csharp2md`. Os links apontam para `main` nos projetos de referência e para o commit local observado (`62789a0`) no `csharp2md`.

## Conclusão executiva

O melhor ponto de partida não é copiar integralmente nenhum dos projetos. Para este repositório, vale combinar:

- do `Understand-Anything`, a orquestração por fases, os contratos intermediários verificáveis, o processamento em lotes e a revisão separada da geração;
- do `compozy/kb`, a separação entre automação determinística e síntese LLM, o ciclo `ingest → compile → query/file-back → lint/heal`, a proveniência por artigo e o conhecimento persistido como Markdown portável;
- do `csharp2md`, a hierarquia de autoridade: manifestos e payloads factuais são canônicos; Markdown e a wiki são projeções downstream e nunca podem criar fatos técnicos.

Essa combinação respeita a fronteira que o próprio `csharp2md` declara: ele produzirá um pacote factual, auditável e navegável, enquanto interpretação de regras de negócio, wiki, busca e LLM ficam a jusante. A primeira entrega do gerador também exclui análise incremental e snapshot diff, portanto um rebuild completo da wiki é uma decisão aceitável para o MVP. ([arquitetura-alvo](https://github.com/cdpfernando/csharp2md/blob/62789a075bd51b6f90ee2f9595528eea83307610/docs/architecture/architecture-knowledge-engine.md), [README](https://github.com/cdpfernando/csharp2md/blob/62789a075bd51b6f90ee2f9595528eea83307610/README.md))

## Restrições herdadas do csharp2md

O desenho desta wiki deve começar pelo contrato do produtor, não pelas conveniências das referências:

1. O `csharp2md` separa fonte/configuração, fatos estruturais, observações, fatos classificados, relações diretas confirmadas e projeções de retrieval. Uma camada superior não pode retroativamente inventar significado para uma inferior. ([hierarquia de autoridade](https://github.com/cdpfernando/csharp2md/blob/62789a075bd51b6f90ee2f9595528eea83307610/docs/architecture/architecture-knowledge-engine.md))
2. O pacote expõe por manifesto inventário, fatos, observações, relações, unknowns, cobertura, diagnósticos, catálogos, postings e projeções. Todos os artefatos devem ser alcançáveis por caminhos relativos a partir do manifesto. ([output e retrieval](https://github.com/cdpfernando/csharp2md/blob/62789a075bd51b6f90ee2f9595528eea83307610/docs/architecture/output-and-retrieval.md))
3. Markdown não pode introduzir fatos, relações ou classificações; links quebrados, locators obsoletos ou valores projetados divergentes devem falhar validação. ([contrato de Markdown](https://github.com/cdpfernando/csharp2md/blob/62789a075bd51b6f90ee2f9595528eea83307610/docs/architecture/output-and-retrieval.md))
4. O pacote evita arquivos monolíticos de relações/catálogos e representa identidades de alta cardinalidade em shards e postings limitados. A wiki deve preservar esse modelo de navegação e não achatar tudo em um único grafo JSON. ([restrições de escala](https://github.com/cdpfernando/csharp2md/blob/62789a075bd51b6f90ee2f9595528eea83307610/docs/architecture/output-and-retrieval.md))

## Understand-Anything

### Objetivo e fluxo principal

O projeto transforma código, documentação ou bases de conhecimento em um grafo navegável com busca, explicações e tours. Para código, o README descreve um pipeline multiagente que cria nós de arquivos, funções e classes e publica o resultado em `.ua/knowledge-graph.json`; para uma wiki no padrão Karpathy, `/understand-knowledge` combina parsing determinístico de links/categorias com agentes que inferem relações implícitas, entidades e claims. ([README: proposta](https://github.com/Egonex-AI/Understand-Anything#understand-anything), [README: knowledge bases](https://github.com/Egonex-AI/Understand-Anything#analyze-knowledge-bases), [README: quick start](https://github.com/Egonex-AI/Understand-Anything#2-analyze-your-codebase))

A skill extensa funciona como o orquestrador. O fluxo corrente faz preflight e configuração de ignore; scan; construção de lotes semânticos; análise paralela por lote; merge/normalização; revisão do grafo montado; análise de camadas; tour; validação final; e save. O `SKILL.md` trata README e manifest do projeto analisado como dados não confiáveis e instrui os agentes a ignorar comandos neles embutidos, uma proteção relevante contra prompt injection. ([skill `/understand`](https://github.com/Egonex-AI/Understand-Anything/blob/main/understand-anything-plugin/skills/understand/SKILL.md))

O fluxo especializado para wiki é menor: detectar/parsear deterministicamente, analisar lotes de 10–15 artigos com até três agentes, mesclar, validar e salvar. O parser possui a responsabilidade por frontmatter, headings, categorias e wikilinks; os agentes acrescentam somente entidades, claims e relações implícitas. Essa separação é diretamente aplicável aqui, substituindo o parser do projeto pelo contrato factual do `csharp2md`. ([skill `/understand-knowledge`](https://github.com/Egonex-AI/Understand-Anything/blob/main/understand-anything-plugin/skills/understand-knowledge/SKILL.md))

### Arquitetura, artefatos e agentes

O repositório é um monorepo TypeScript/pnpm com core, dashboard, CLI e plugin; o manifesto raiz declara build recursivo, Vitest e ESLint, e inclui parsers Tree-sitter, inclusive C#. ([package.json](https://github.com/Egonex-AI/Understand-Anything/blob/main/package.json), [workspace](https://github.com/Egonex-AI/Understand-Anything/blob/main/pnpm-workspace.yaml))

Os agentes são prompts versionados no plugin: `project-scanner`, `file-analyzer`, `assemble-reviewer`, `architecture-analyzer`, `tour-builder`, `graph-reviewer`, além de analisadores de artigo, design e domínio. Isso deixa responsabilidades e formatos de handoff inspecionáveis no Git. ([diretório de agentes](https://github.com/Egonex-AI/Understand-Anything/tree/main/understand-anything-plugin/agents))

Os artefatos ficam isolados em `.ua/` (mantendo `.understand-anything/` para projetos legados): `knowledge-graph.json`, `meta.json`, `config.json`, `.understandignore`, `intermediate/` e `tmp/`. O grafo final contém metadados do projeto, `nodes`, `edges`, `layers` e `tour`; o merge normaliza IDs e complexidade, deduplica nós/arestas e remove referências pendentes antes da revisão. ([contrato do diretório e fases](https://github.com/Egonex-AI/Understand-Anything/blob/main/understand-anything-plugin/skills/understand/SKILL.md), [merge e normalização](https://github.com/Egonex-AI/Understand-Anything/blob/main/understand-anything-plugin/skills/understand/SKILL.md#phase-2--analyze))

Na variante de wiki, o handoff também é persistido: o parser cria `scan-manifest.json`, cada lote produz `analysis-batch-{N}.json`, o merge cria `assembled-graph.json` e a publicação grava `knowledge-graph.json` e `meta.json`. O `article-analyzer` recebe os artigos explicitamente como dados não confiáveis, portanto instruções embutidas no corpus não devem alterar seu trabalho. ([skill `/understand-knowledge`](https://github.com/Egonex-AI/Understand-Anything/blob/main/understand-anything-plugin/skills/understand-knowledge/SKILL.md), [prompt do article analyzer](https://github.com/Egonex-AI/Understand-Anything/blob/main/understand-anything-plugin/agents/article-analyzer.md))

### Incrementalidade e manutenção

Após a primeira execução da análise de **código**, o modo padrão usa o commit salvo para obter arquivos alterados por `git diff`; remove nós e arestas antigos desses arquivos, analisa apenas os lotes afetados e os mescla de volta com o grafo existente. Vizinhos não alterados continuam no contexto para permitir arestas entre lotes. Camadas arquiteturais são recalculadas sobre o conjunto completo, preservando nomes/IDs anteriores quando possível. Ao salvar, fingerprints estruturais viram a baseline das atualizações futuras. ([caminho incremental](https://github.com/Egonex-AI/Understand-Anything/blob/main/understand-anything-plugin/skills/understand/SKILL.md#incremental-update-path), [arquitetura em update](https://github.com/Egonex-AI/Understand-Anything/blob/main/understand-anything-plugin/skills/understand/SKILL.md#phase-4--architecture), [save e fingerprints](https://github.com/Egonex-AI/Understand-Anything/blob/main/understand-anything-plugin/skills/understand/SKILL.md#phase-7--save))

A skill atual de wiki descreve rebuild determinístico, análise e merge, mas não define invalidação incremental por artigo. Portanto, a lição para este MVP é preservar estado/IDs e projetar handoffs reaproveitáveis, sem prometer incrementalidade fina antes da estabilização do pacote de entrada. ([skill `/understand-knowledge`](https://github.com/Egonex-AI/Understand-Anything/blob/main/understand-anything-plugin/skills/understand-knowledge/SKILL.md))

### O que adotar e evitar

Adotar no MVP:

- pipeline explícito e retomável, com arquivos intermediários por fase;
- contratos JSON/Markdown validados antes de publicar;
- geração e revisão em papéis diferentes;
- lotes limitados e IDs canônicos;
- tratamento de qualquer texto vindo do repositório analisado como dado não confiável.

Evitar no MVP:

- reler e reinterpretar o código C# diretamente, pois isso duplicaria e poderia contradizer a autoridade factual do `csharp2md`;
- o grafo monolítico `knowledge-graph.json`, incompatível com o sharding previsto pelo produtor;
- dashboard, personas, chat, tour e análise de impacto antes de estabilizar a wiki;
- inferências LLM sobre arestas técnicas como se fossem fatos confirmados. Quando forem úteis, devem aparecer explicitamente como interpretação, hipótese ou unknown, ligadas à evidência factual.

## compozy/kb

### Objetivo e fluxo principal

`kb` é um CLI Go de binário único que automatiza o trabalho não-LLM: scaffold de tópicos, ingestão, conversão, lint estrutural, análise de codebase, indexação e busca. A compilação LLM fica deliberadamente na camada de agente; a saída é Markdown local, sem SaaS. ([README](https://github.com/compozy/kb#kb))

O modelo de wiki separa quatro fases: `ingest` coloca fontes imutáveis em `raw/`; `compile` faz a LLM produzir artigos em `wiki/concepts/`; `query` responde a partir da wiki e arquiva a resposta em `outputs/queries/`, promovendo sínteses fortes; `lint` procura problemas estruturais e usa a LLM para healing semântico. Cada fase acrescenta uma entrada ao `log.md`. ([arquitetura](https://github.com/compozy/kb/blob/main/skills/kb/references/architecture.md))

### Arquitetura, artefatos e prompts

O CLI mantém comandos finos e empurra comportamento para pacotes internos separados: ingestão, converters, frontmatter, lint, scanner, adapters, grafo, métricas, vault, QMD, output e modelos. O pipeline de codebase é `scan → parse por adapter → normalizar grafo → computar métricas → renderizar → escrever`. ([AGENTS.md](https://github.com/compozy/kb/blob/main/AGENTS.md), [referência de ingestão de codebase](https://github.com/compozy/kb/blob/main/skills/kb/references/cli-ingest-codebase.md))

Um tópico wiki possui `topic.yaml`, `CLAUDE.md`/`AGENTS.md`, `log.md`, `raw/`, `wiki/`, `outputs/` e `bases/`. O modo OKF oferece uma alternativa portável e plana: conceitos tipados em Markdown, `index.md`, `log.md`, frontmatter com `description`, `timestamp`, `title` e `type`, e links Markdown relativos em vez de wikilinks de Obsidian. `promote` transforma mecanicamente um conceito wiki em OKF sem alterar a fonte, e `okf check --strict` pode funcionar como gate de CI. ([skill `kb`](https://github.com/compozy/kb/blob/main/skills/kb/SKILL.md), [README: OKF](https://github.com/compozy/kb#kb-promote))

O prompt/skill de compilação exige artigos autônomos e citados, `sources:` no frontmatter, seção de fontes, links densos e síntese em vez de resumo de uma única fonte. Para updates, manda ler a página atual e as novas fontes, propor mudanças com `Current/Proposed/Reason/Source`, varrer contradições e auditar backlinks/downstream antes de editar. ([guia de compilação](https://github.com/compozy/kb/blob/main/skills/kb/references/compilation-guide.md))

### Incrementalidade e manutenção

A manutenção é incremental no nível do conhecimento, não apenas no processamento: fontes são mantidas imutáveis, consultas retornam ao corpus, o log é append-only e updates verificam fontes mais novas que os artigos. O lint/heal procura stale content, inconsistências, cobertura ausente, violações de formato, links ruins e insights arquivados ainda não absorvidos. ([arquitetura](https://github.com/compozy/kb/blob/main/skills/kb/references/architecture.md), [lint e heal](https://github.com/compozy/kb/blob/main/skills/kb/references/lint-procedure.md))

### O que adotar e evitar

Adotar no MVP:

- separar a camada mecânica (ler/validar manifestos, resolver locators, checar links) da camada LLM (planejar e sintetizar artigos);
- manter frontmatter com identidade, revisão da fonte, tipo, status e evidências;
- publicar índice, relatório de validação e log de operações;
- fazer queries e revisões futuras retornarem ao corpus como artefatos rastreáveis;
- validar estrutura automaticamente e reservar contradição/qualidade semântica para um revisor LLM.

Evitar no MVP:

- dependência obrigatória de Obsidian, QMD, embeddings, Firecrawl ou formatos externos;
- o alvo rígido de 3.000–4.000 palavras por artigo, que é caro e não decorre do contrato do `csharp2md`;
- copiar `raw/` para dentro da wiki quando o pacote versionado do `csharp2md` já é a fonte canônica;
- aceitar wikilinks sem resolver: links relativos e locators validáveis são mais portáveis, opção que o próprio modo OKF adotou. ([OKF e links relativos](https://github.com/compozy/kb/blob/main/skills/kb/SKILL.md#okf-dual-mode))

## Escopo recomendado para o MVP

### Resultado

Uma execução recebe o caminho de um pacote certificado do `csharp2md`, valida seu manifesto, compila uma wiki Markdown pequena e auditável em staging, revisa a projeção e publica atomicamente a nova versão. A wiki deve responder inicialmente:

- quais são as soluções, componentes e deployment units;
- quais entry points e boundary operations existem;
- quais contratos e dados participam de cada operação;
- quais relações diretas foram confirmadas e com qual evidência;
- quais candidates, unknowns, limitações e gaps de cobertura permanecem.

Interpretações de regra de negócio podem existir em seção separada e rotulada como interpretação LLM, sempre apontando para fatos e source locators. Elas nunca devem ser promovidas ao mesmo status dos fatos do gerador.

### Pipeline mínimo

1. **Preflight** — resolver pacote e manifesto; rejeitar pacote incompleto, incompatível ou não certificado.
2. **Plan** — ler catálogos/postings e produzir um plano determinístico de páginas, IDs e fontes.
3. **Compile** — gerar ou atualizar artigos em staging, mantendo evidências e locators.
4. **Review** — checar schema, links, identidades, cobertura e fidelidade factual; depois fazer uma revisão semântica separada.
5. **Publish** — trocar staging pela wiki validada, atualizar `index.md`, estado da execução e log.
6. **Maintain** — na primeira versão, repetir o pipeline completo para um novo manifest. Incrementalidade por diff entra somente quando o produtor expuser revisões/contratos estáveis suficientes.

### Papéis de agente

- `wiki-orchestrator`: controla fases, budgets, retries e handoffs; não escreve conteúdo final.
- `wiki-compiler`: transforma um plano e evidências delimitadas em artigos; não descobre fatos fora do pacote.
- `wiki-reviewer`: verifica grounding, contradições, cobertura e conformidade; não corrige silenciosamente o artigo que acabou de avaliar.

Scanners, analisadores de código, agentes de UI, busca, onboarding e impacto ficam fora do MVP.

### Estrutura mínima sugerida

```text
reverser-agent/
├── AGENTS.md
├── README.md
├── agents/
│   ├── wiki-orchestrator.md
│   ├── wiki-compiler.md
│   └── wiki-reviewer.md
├── prompts/
│   ├── compile-article.md
│   └── review-article.md
├── schemas/
│   ├── article.schema.json
│   ├── plan.schema.json
│   └── run-state.schema.json
├── config/
│   └── wiki.example.yaml
├── scripts/
│   ├── validate-input.*
│   └── validate-wiki.*
├── docs/
│   └── research/
└── wiki/                       # artefato publicado, se versionado neste repo
    ├── index.md
    ├── concepts/
    ├── reports/
    └── .state/
```

O repositório estava vazio no início desta pesquisa, portanto não havia convenção local de notas a preservar. `docs/research/` foi escolhido para separar material exploratório dos futuros contratos normativos.

## Decisões que podem esperar

- formato definitivo do frontmatter e taxonomia de tipos de artigo;
- wiki no mesmo repositório ou em diretório/repositório de saída configurável;
- Obsidian/OKF, busca lexical/vetorial e MCP;
- incrementalidade fina e invalidação transitiva;
- dashboard, chat, tours e diff impact;
- execução em CI, agendamento e publicação.

Antes dessas decisões, o contrato mínimo mais importante é: **cada afirmação técnica da wiki aponta para identidades e evidências do pacote `csharp2md`, e a validação consegue demonstrar que a projeção não inventou nem alterou fatos.**
