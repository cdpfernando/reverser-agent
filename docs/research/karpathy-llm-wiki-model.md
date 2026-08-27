# Modelo LLM Wiki de Karpathy

Pesquisa realizada em 2026-08-25 contra a fonte primária de Andrej Karpathy, [`llm-wiki.md`](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f). O gist possui [uma única revisão, a criação original](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f/revisions). Comentários de terceiros não foram usados como autoridade.

## Conclusão executiva

O modelo é mais direto do que o desenho atual do Reverser:

- a wiki é uma base de conhecimento Markdown persistente, interligada e cumulativa, posicionada entre a pessoa e as fontes brutas;
- a LLM lê as fontes, cria e edita as páginas, mantém referências, índice e consistência, e também consulta a wiki para responder;
- o humano seleciona fontes, dirige a exploração, formula perguntas e pode acompanhar ou revisar as alterações;
- fontes brutas imutáveis são a fonte de verdade; a wiki é síntese derivada;
- ingestões e boas respostas enriquecem a mesma wiki; lint semântico também é trabalho atribuído à LLM;
- `index.md`, `log.md`, instruções de schema e Git dão orientação e histórico, mas o gist não define staging, revisão obrigatória, merge, publicação atômica nem um publisher determinístico.

A afirmação anterior — **“agentes são autores e mantenedores; ferramentas determinísticas são guardiãs e publicadoras”** — é apenas parcialmente sustentada. A primeira oração é fiel se “agentes” for entendida como a LLM ou o LLM agent. A segunda não vem do gist. Validação e publicação determinísticas podem ser uma especialização do Reverser, mas devem ser apresentadas como decisão própria, não como parte do padrão Karpathy.

## Natureza e limite da fonte

Karpathy apresenta um *idea file* para ser entregue a um agente, que instanciará os detalhes junto com o usuário. Ele declara expressamente que estrutura de diretórios, schema, formatos e ferramentas dependem do domínio e são opcionais ([introdução](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f#file-llm-wiki-md-L2-L4), [nota final](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f#file-llm-wiki-md-L53-L55)). Portanto, trata-se de um padrão conceitual primário, não de uma especificação normativa de implementação.

## Modelo canônico

### O que é a wiki

A LLM Wiki é uma coleção estruturada de arquivos Markdown interligados que acumula síntese entre o usuário e as fontes. Em vez de redescobrir conhecimento a cada pergunta, a LLM integra cada fonte ao que já existe, atualiza páginas e contradições e conserva a síntese para reutilização futura. Karpathy a chama de artefato persistente e cumulativo ([ideia central](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f#file-llm-wiki-md-L6-L12)).

Não é apenas um relatório final, cache de respostas ou projeção gerada uma vez. É o corpus de trabalho evolutivo que a LLM escreve e consulta.

### Quem cria, escreve e mantém

A atribuição é explícita: a LLM possui a camada wiki, cria páginas, atualiza-as quando chegam fontes, mantém cross-references e consistência. A formulação curta do texto é “You read it; the LLM writes it” ([três camadas](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f#file-llm-wiki-md-L19-L25)).

O humano normalmente não redige a wiki. Seu papel é:

- curar e acrescentar fontes;
- dirigir a análise e indicar ênfases;
- fazer perguntas e explorar conexões;
- ler os resultados e pensar sobre o significado;
- opcionalmente revisar updates em um fluxo human-in-the-loop.

Karpathy descreve acompanhamento próximo como preferência pessoal, mas também admite ingestão em lote com menos supervisão. Revisão humana de toda escrita não é uma regra do padrão ([ingestão](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f#file-llm-wiki-md-L26-L28), [divisão do trabalho](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f#file-llm-wiki-md-L49-L52)).

O gist fala de um LLM agent genérico. Não define uma equipe, papéis separados ou handoffs entre Orchestrator, Compiler e Reviewer. O exemplo de wiki empresarial menciona LLMs no plural e possível revisão humana, mas não prescreve uma arquitetura multiagente ([exemplos](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f#file-llm-wiki-md-L13-L18)).

### Quando a wiki é lida

Na operação `Query`, a pergunta é feita contra a wiki. A LLM procura páginas relevantes, lê o conhecimento já sintetizado e produz uma resposta com citações. Em escala moderada, ela lê primeiro `index.md` para localizar páginas e então aprofunda nelas ([query](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f#file-llm-wiki-md-L29-L30), [índice](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f#file-llm-wiki-md-L32-L36)).

As fontes não desaparecem: sustentam a ingestão e as citações. Porém, a diferença em relação a RAG puro é justamente não começar de novo nas fontes brutas em toda pergunta.

### Como os updates acontecem

O lifecycle descrito é incremental e conversacional:

```text
instanciar schema com a LLM
  -> adicionar fonte imutável
    -> pedir ingestão
      -> LLM lê e discute a fonte
        -> cria resumo e atualiza páginas relacionadas
          -> atualiza index.md e acrescenta log.md
            -> perguntas consultam a wiki
              -> boas respostas podem virar páginas
                -> lint periódico mantém a wiki saudável
                  -> repetir
```

Uma fonte pode alterar muitas páginas. Perguntas também acumulam conhecimento quando uma boa resposta, comparação ou conexão é arquivada de volta. Periodicamente, a LLM procura contradições, claims stale, páginas órfãs, cross-references ausentes e lacunas ([ingestão, query e lint](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f#file-llm-wiki-md-L26-L31)).

O gatilho mostrado é o usuário adicionar uma fonte e mandar processá-la. O texto não especifica watcher automático. Também não exige que toda resposta seja persistida: afirma que boas respostas **podem** ser arquivadas.

### Artefatos e estrutura

O núcleo possui três camadas:

1. fontes brutas, curadas e imutáveis;
2. diretório de Markdown gerado pela LLM;
3. documento de schema/instruções, como `AGENTS.md` ou `CLAUDE.md`, coevoluído por humano e LLM.

A wiki pode conter resumos, páginas de entidades e conceitos, comparações, overview e síntese. Dois arquivos têm papéis especiais:

- `index.md`: catálogo orientado ao conteúdo, atualizado em toda ingestão e lido no início de queries;
- `log.md`: histórico cronológico append-only de ingestões, queries e lint passes.

Frontmatter, assets locais, Obsidian, Dataview, Marp, busca e QMD são opcionais. A wiki pode ser simplesmente um repositório Git de Markdown, obtendo histórico, branches e colaboração ([indexação e log](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f#file-llm-wiki-md-L32-L40), [opções e Git](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f#file-llm-wiki-md-L41-L48)).

### Autoridade

As fontes brutas são explicitamente imutáveis e constituem a fonte de verdade. A LLM somente as lê. A wiki é gerada e mantida pela LLM ([arquitetura em três camadas](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f#file-llm-wiki-md-L19-L25)).

O gist não define proof states, certificação, provenance maps, regras para promover relações ou uma distinção formal entre fato e interpretação. Citações aparecem no contrato de query, e contradições devem ser registradas, mas a disciplina epistemológica detalhada precisa vir do schema específico do domínio.

No Reverser, é coerente especializar “fontes brutas” como os snapshots e payloads factuais do `csharp2md`, preservando-os como autoridade. O Markdown de retrieval produzido pelo `csharp2md` e a LLM Wiki do Reverser não são o mesmo artefato: o primeiro é uma projeção técnica do produtor usada como insumo; a segunda é a síntese cumulativa escrita pela LLM.

### O que é determinístico

O padrão não separa explicitamente uma camada LLM autora de uma camada determinística publicadora. Pelo contrário, ele atribui à LLM a edição e o lint da wiki.

Há apenas mecanismos de feição mecânica ou reprodutível:

- fontes são imutáveis;
- `log.md` é append-only e pode usar prefixos parseáveis por ferramentas Unix;
- Git fornece histórico, branch e colaboração;
- busca por CLI/MCP pode ser adicionada opcionalmente.

Nada no gist exige seleção determinística de evidências, validação estrutural, staging, troca atômica, rebuild equivalente, aprovação formal ou bytes estáveis. Essas garantias podem complementar o padrão, mas não derivam dele.

## Comparação após o alinhamento documental

### Alinhamentos

| Ponto | Karpathy | Reverser atual |
| --- | --- | --- |
| Conhecimento cumulativo | Wiki persistente enriquece com fontes e perguntas | Wiki é memória operacional cumulativa do assistente ([README](../../README.md), [escopo](../scope.md)) |
| Fonte de verdade | Fontes brutas imutáveis | Pacote factual do `csharp2md` é autoridade técnica ([arquitetura](../architecture.md#autoridade)) |
| Conteúdo derivado | Wiki pertence à LLM | Investigator possui autoria editorial de páginas, índice, log e cross-references ([agentes](../../agents/README.md#investigator)) |
| Instruções duráveis | Schema como `AGENTS.md`/`CLAUDE.md` disciplina a LLM | Cada workspace possui schema editorial versionado; `agents/README.md` define papéis até existir runner ([arquitetura](../architecture.md#reverser-llm-wiki), [agentes](../../agents/README.md)) |
| Busca simples primeiro | `index.md` é suficiente em escala moderada | Queries começam por `index.md`; embeddings e banco são adiados ([fluxo](../product-flow.md#5-investigar), [roadmap](../roadmap.md#fase-5--superfícies-opcionais)) |
| Histórico | Wiki pode ser um repositório Git | Workspace e wiki possuem revisões e histórico ([arquitetura](../architecture.md#knowledge-workspace)) |

### Extensões deliberadas do Reverser

| Tema | Modelo de Karpathy | Reverser alinhado | Consequência |
| --- | --- | --- | --- |
| Centro da consulta | `index.md` e páginas da wiki são lidos primeiro | mesma ordem, seguida de verificação contra a revisão e os payloads canônicos quando necessário ([fluxo](../product-flow.md#5-investigar), [arquitetura](../architecture.md#escada-de-consulta-e-verificação)) | acrescenta grounding auditável sem deslocar a wiki do centro |
| Momento do update | toda ingestão integra a fonte; boas queries podem retornar à wiki | ingestão autorizada atualiza a wiki; query só retorna por file-back explícito ([fluxo](../product-flow.md#8-fazer-file-back-de-uma-descoberta)) | política conservadora adicional para mutação derivada de conversa |
| Lint | health-check semântico é pedido à LLM | Investigator executa lint semântico; módulos determinísticos validam invariantes mecânicos ([agentes](../../agents/README.md#lint-da-wiki)) | divisão híbrida explícita |
| Publicação | a LLM edita; usuário observa em tempo real; Git dá histórico | staging, validação, revisão e troca atômica preservam a última revisão válida ([fluxo](../product-flow.md#9-publicar-a-wiki)) | Garantia adicional do Reverser, sem base no gist |
| Unidade de lifecycle | fonte adicionada, query arquivada e lint | snapshot, workspace revision, insight candidate e wiki revision | O Reverser adiciona transação, composição multi-solução e invalidação dependente |
| Autoridade formal | fontes são verdade; wiki é derivada, sem taxonomia de prova | cinco estados, evidence bundles, locators e `confirmed` exclusivo do `csharp2md` ([README](../../README.md#estados-do-conhecimento)) | Extensão necessária para engenharia reversa auditável |
| Curadoria humana | humano raramente escreve a wiki; dirige e pode revisar | conteúdo humano vive em overlay separado da área regenerável ([arquitetura](../architecture.md#reverser-llm-wiki), [roadmap](../roadmap.md#fase-4--composição-oficial-e-manutenção)) | política própria compatível com autoria LLM |

O desvio inicialmente encontrado — wiki como saída promovida em vez de memória operacional — foi corrigido nos documentos normativos. A wiki agora é mantida em toda ingestão, consultada primeiro e enriquecida por queries somente via file-back explícito.

## Veredito sobre autoria e publisher

Avaliação da formulação anterior:

| Trecho | Veredito | Motivo |
| --- | --- | --- |
| “agentes são autores e mantenedores” | **Sustentado com ajuste** | O gist atribui autoria e manutenção à LLM/LLM agent. Não prescreve múltiplos agentes nem papéis separados. |
| “ferramentas determinísticas são guardiãs e publicadoras” | **Não sustentado** | Não há publisher, gate determinístico ou publicação atômica na fonte; o lint descrito também é realizado pela LLM. |
| “csharp2md é a autoridade factual” | **Adaptação coerente** | Instancia a camada de fontes imutáveis/source of truth para o domínio de engenharia reversa. |

Formulação fiel para o projeto:

> No padrão Karpathy, a LLM possui, escreve, consulta e mantém a camada wiki; as fontes imutáveis permanecem a autoridade. O Reverser acrescenta proof states, revisão, validação e publicação determinísticas para atender a proveniência e atomicidade exigidas pelo domínio.

Um publisher determinístico continua compatível como guardrail se não escrever síntese nem decidir verdade semântica. Porém, sua existência é uma decisão arquitetural do Reverser. Da mesma forma, separar Orchestrator, Investigator/Compiler e Reviewer pode ser útil, mas não é requisito ou padrão descrito por Karpathy.

## Correções aplicadas aos documentos normativos

Sem alterar as invariantes herdadas do `csharp2md`, a revisão:

1. distingue nominalmente `csharp2md Markdown retrieval projection` de `Reverser LLM Wiki`;
2. coloca `index.md -> páginas relevantes` no início da escada de consulta, antes de aprofundar em fatos, postings e código para confirmação ou lacunas;
3. atribui à LLM a criação e manutenção de páginas, cross-references, `index.md`, `log.md` e lint semântico;
4. define ingestão de um snapshot como atualização da wiki existente, não apenas construção de baseline map;
5. mantém file-back explícito de respostas como política adicional de segurança;
6. documenta Reviewer, validação determinística, staging e publicação atômica como extensões do Reverser;
7. preserva pacote factual, proof states, provenance, budgets e última revisão válida, pois são requisitos do domínio ausentes do gist, não incompatibilidades com ele.

A mudança não criou um quarto agente nem um `WikiCompiler` determinístico. O Investigator recebeu ownership editorial completo e explícito da wiki, enquanto ferramentas mecânicas verificam e persistem o que a LLM escreveu.
