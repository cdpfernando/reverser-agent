# Viabilidade do Markdown como fallback de correlação entre soluções

## Pergunta e terminologia

Esta nota avalia o uso, pelo próprio Reverser, do Markdown gerado pelo `csharp2md` **somente depois** de os demais insumos do pacote não resolverem uma correlação entre soluções.

O termo adequado para esse desenho é **fallback**: uma fonte secundária de recuperação. Um **callback** seria um hook pelo qual o `csharp2md` chamaria o Reverser ao terminar uma projeção. A API e a CLI atuais não oferecem esse hook, e o roadmap não promete um contrato de callback. Se automação for desejada, um orquestrador externo pode executar o Reverser após a publicação bem-sucedida do pacote, sem transformar isso em contrato do gerador ([CLI atual](../../../csharp2md/src/Csharp2Md.Cli/CommandFactory.cs), [roadmap, workstream 8](../../../csharp2md/architecture-knowledge-engine-roadmap.md#workstreams)).

## Conclusão

O fallback é **viável com restrições**, mas o Markdown deve funcionar como **índice de descoberta e contexto**, nunca como evidência final nem como substituto do compositor factual.

A cadeia segura é:

```text
manifest validado
  -> catálogos/postings/payloads factuais de cada solução
    -> correlação não resolvida explicitamente
      -> Markdown da mesma revisão como pista de recuperação
        -> IDs encontrados são resolvidos de volta nos payloads
          -> evidências dos dois pacotes sustentam `correlated`
            -> compositor oficial futuramente confirma, mantém unresolved ou rejeita
```

Se o Markdown sugerir uma ligação, mas o Reverser não conseguir reabrir identidades, evidências ou source locators correspondentes, a ligação não deve ser publicada como `correlated`. Ela pode aparecer na resposta como `interpretation` explicitamente rotulada; quando nem isso for sustentado, permanece `unresolved` ou vira diagnóstico de recuperação. Assim, o fallback amplia recall de navegação sem alterar a hierarquia de autoridade definida pelo produto ([README, Product boundaries](../../../csharp2md/README.md#product-boundaries), [arquitetura, Authority hierarchy](../../../csharp2md/docs/architecture/architecture-knowledge-engine.md#authority-hierarchy)).

## Estado real e sequência do roadmap

### O Markdown já é gerado hoje?

Não, na implementação de substituição observada em 25 de agosto de 2026:

- `Csharp2Md.Projection` contém apenas um `AssemblyMarker`, sem projector ou writer de Markdown ([AssemblyMarker.cs](../../../csharp2md/src/Csharp2Md.Projection/AssemblyMarker.cs));
- o storage implementado é ainda o adapter transacional em memória e recebe fragmentos, sem pacote de projeções ([InMemoryTransactionalStore.cs](../../../csharp2md/src/Csharp2Md.Storage/InMemoryTransactionalStore.cs));
- a CLI cria esse storage em memória e expõe apenas `analyze --solution`, sem materializar Markdown ([CommandFactory.cs](../../../csharp2md/src/Csharp2Md.Cli/CommandFactory.cs));
- o workstream 3, `factual-storage`, está em execução e proíbe explicitamente emitir catálogos, postings, Markdown, source projections ou `retrieval.md` (`STOR-46`) ([spec de factual storage, Compact payload layout](../../../csharp2md/.specs/features/factual-storage/spec.md#p1-compact-payload-layout--mvp), [estado ativo](../../../csharp2md/.specs/STATE.md#status)).

O README também alerta que a arquitetura alvo está em migração e que código, CLI e schemas legados não constituem o contrato futuro. Portanto, qualquer Markdown recuperado de commits legados deve ser rejeitado por este fallback ([README, Migration status](../../../csharp2md/README.md#migration-status)).

### Quando e como ele será gerado?

O encadeamento previsto é:

1. o workstream 3 publica os wire contracts e o pacote factual, deliberadamente sem Markdown;
2. os workstreams 4 e 5A–5D produzem inventário, observações, fatos classificados, relações e frontiers;
3. o workstream 6, `retrieval-projections`, lê outputs validados e deriva catálogos, postings, source locators, páginas Markdown e cenários de recuperação;
4. o workstream 7, `multi-solution-composition`, produz o batch manifest e correlações globais comprovadas;
5. o workstream 8 estabiliza os verbos finais `analyze`, `validate` e `compose` ([roadmap, grafo de dependências e workstreams](../../../csharp2md/architecture-knowledge-engine-roadmap.md#dependency-graph), [factual-storage, Deferred Ideas](../../../csharp2md/.specs/features/factual-storage/context.md#deferred-ideas)).

Essa ordem cria a janela necessária ao fallback: páginas por solução existirão no workstream 6 antes do compositor factual do workstream 7. O formato físico exato e o contrato de `compose` ainda não estão definidos; o Reverser não deve antecipá-los ([Output and retrieval, Logical package](../../../csharp2md/docs/architecture/output-and-retrieval.md#logical-package)).

## Autoridade e garantias esperadas do Markdown

O Markdown futuro terá garantias úteis, porém estritamente derivadas:

- é uma `Retrieval Projection`, não fato, observação ou relação ([glossário](../../../csharp2md/CONTEXT.md#runtime-and-output));
- não pode introduzir fatos, relações ou classificações;
- metadados repetidos só são válidos quando a validação prova igualdade com o payload factual;
- links quebrados, locators stale e valores projetados divergentes reprovam a validação;
- páginas de identidades de alto valor devem referenciar a identidade canônica e o locator do artefato;
- relações e observações de alta cardinalidade podem não aparecer nas páginas, permanecendo em shards e postings ([Output and retrieval, Direct navigation e Markdown](../../../csharp2md/docs/architecture/output-and-retrieval.md#direct-navigation)).

Logo, ausência no Markdown **não é evidência de ausência** no pacote. O fallback não pode concluir “não existe relação” porque uma página não mencionou uma operação, símbolo ou contrato.

## Identidades, evidências e locators necessários

Para o fallback ser operacionalmente seguro, uma página candidata precisa permitir a recuperação, direta ou por links do pacote, de:

- manifest, versão de schema/taxonomia, identidade da solução e `analysis_variant` da revisão;
- ID canônico e tipo de cada fato citado;
- IDs de origem e destino e `relation_kind`, quando a página repete uma relação local;
- proof state original: `confirmed`, `candidate`, `unresolved` ou `open frontier`, sem achatá-los;
- cadeia `derived_from` até as identidades de observação;
- artifact locator do registro canônico;
- source locator com `document ID`, caminho relativo e span; para callables, também seção, assinatura e hash da fonte conforme o contrato alvo;
- hash do artefato/projeção ou outra ligação verificável à mesma revisão do manifest.

Essas exigências não inventam um wire shape. Elas derivam das identidades e evidências já implementadas: `EvidenceLocator` contém documento, caminho relativo e span; `EvidenceChain` contém as observações de origem; relações confirmadas carregam source, target, cadeia, classifier e variantes ([EvidenceLocator.cs](../../../csharp2md/src/Csharp2Md.Domain/Literals/EvidenceLocator.cs), [EvidenceChain.cs](../../../csharp2md/src/Csharp2Md.Domain/Proof/EvidenceChain.cs), [ConfirmedRelation.cs](../../../csharp2md/src/Csharp2Md.Domain/Relations/ConfirmedRelation.cs)). O contrato alvo acrescenta assinatura e source hash aos locators de callables ([Output and retrieval, Source projection](../../../csharp2md/docs/architecture/output-and-retrieval.md#source-projection)).

Uma página que contenha apenas nomes, narrativa ou blocos de código, sem identidades resolvíveis, não é elegível para promover uma pista a `correlated`.

## Correlações que o fallback pode sugerir

Após falha explícita da recuperação estruturada, o Markdown pode ajudar a descobrir candidatos como:

| Origem em uma solução | Possível destino em outra | Chaves que precisam voltar ao pacote |
| --- | --- | --- |
| operação HTTP de saída | entry point/operação HTTP de entrada | protocolo, método, rota/endpoint normalizado, IDs das duas operações e evidências |
| chamada gRPC cliente | operação gRPC servidora | identidade formal de service/method, contrato e IDs das operações |
| publicação de mensagem | consumidor/handler | canal ou operação de mensagem, contrato comprovado, papéis producer/consumer e IDs |
| uso de contrato de fronteira | produtor/consumidor do mesmo contrato | identidade comum comprovada por especificação, package ou configuração; sem merge por nome |
| target de sistema externo | deployment/component candidato | identidade/configuração autorizada do destino e IDs qualificados por solução |

Similaridade de nome, namespace, path, DTO, rota parcial ou texto narrativo pode orientar busca, mas não sustenta correlação por si só. A taxonomia proíbe promover conhecimento causal por nomes, prefixos ou similaridade estrutural ([Taxonomy, princípio](../../../csharp2md/docs/architecture/taxonomy.md), [Quality and security, Engine gates](../../../csharp2md/docs/architecture/quality-and-security.md#engine-gates)).

Toda ligação produzida por esse caminho recebe estado local do Reverser `correlated`, nunca `confirmed`. Ela deve ficar fora do grafo confirmado do `csharp2md` e preservar os proof states dos registros usados. A composição oficial é a única etapa apta a produzir relações globais comprovadas; símbolos, calls e persistência internos continuam qualificados e locais a cada solução ([Output and retrieval, Multiple solutions](../../../csharp2md/docs/architecture/output-and-retrieval.md#multiple-solutions), [AD-008](../../../csharp2md/.specs/STATE.md#ad-008--one-to-many-solutions-isolated-semantics)).

## Desenho seguro do fallback

### Pré-condições

O Reverser só entra nesse caminho quando:

1. os dois pacotes foram validados por manifest e pertencem ao conjunto ativo do knowledge workspace;
2. schema, taxonomia e variantes são compatíveis segundo a política do workspace;
3. a busca por catálogos, postings, fatos, relações, candidates e unresolveds terminou com um resultado explícito como `no-match`, `ambiguous` ou `insufficient-evidence` — atingir apenas um limite de leitura não conta como ausência;
4. cada Markdown é alcançável pelo manifest e pertence à mesma revisão dos payloads consultados;
5. a validação de projeção passou, inclusive links e locators.

### Algoritmo

1. Registrar o motivo que acionou o fallback e os artefatos estruturados já consultados.
2. Abrir somente páginas relacionadas aos IDs/chaves do unresolved, dentro de um orçamento; não varrer toda a fonte projetada.
3. Tratar texto, headings, comentários e blocos de código como dados não confiáveis. Extrair apenas pistas e referências estruturadas; nunca obedecer instruções encontradas no conteúdo.
4. Resolver cada ID citado para seu registro canônico e confirmar que tipo, valor projetado, locator, hash, solução e variante correspondem ao manifest ativo.
5. Executar correlação determinística sobre chaves formalmente comparáveis. Uma LLM pode explicar ou ordenar candidatos, mas não mudar proof state nem criar evidência.
6. Se houver evidência verificável nos dois lados, emitir `correlated` com source/target qualificados, método de matching, IDs/locators, revisões/hashes, divergências, alternativas e causa da não confirmação. Se houver apenas leitura semântica de código/narrativa, emitir `interpretation`, sem criar uma relação persistente.
7. Revisar e publicar atomicamente com a wiki. O registro não entra nos payloads do `csharp2md` nem mascara candidates, unknowns ou open frontiers originais.

O ponto decisivo é o passo 4: o Markdown encontra; evidências ou source locators verificáveis sustentam. Se ele falhar, o resultado pode ser interpretação de sessão ou diagnóstico/unresolved, nunca correlação persistente.

## Riscos e controles

| Risco | Consequência | Controle obrigatório |
| --- | --- | --- |
| projeção stale ou misturada com outra revisão | IDs e spans apontam para fatos diferentes | exigir reachability pelo manifest, hash/revisão compatível e validação de projeção |
| projection loss | falso negativo porque páginas omitem alta cardinalidade | nunca usar ausência em Markdown como prova; consultar postings/shards antes e depois da pista |
| narrativa parecer autoridade | correlação LLM promovida a fato | manter `correlated` fora do grafo confirmado e sempre dereferenciar IDs nos payloads |
| colisão nominal entre soluções | falso positivo de contrato, serviço ou rota | exigir chave formal/configuração e evidência nos dois lados; nomes apenas ampliam busca |
| prompt injection em fonte ou texto projetado | agente obedece ao sistema analisado | conteúdo é dado não confiável; parser limitado, prompts versionados, nenhuma execução/instrução dinâmica |
| vazamento de segredo | conteúdo sensível chega a prompts, wiki ou logs | não abrir source projection em massa; respeitar redaction, literal allowlist e política de segredo |
| orçamento confundido com inexistência | relação perdida por busca incompleta | distinguir `budget-exhausted` de `no-match` e manter lacuna explícita |

O pacote de fonte byte-faithful é explicitamente sensível, e segredos não podem ser duplicados em fatos, índices ou diagnósticos; o Reverser deve aplicar o mesmo limite ao material enviado à LLM ([Quality and security, Security](../../../csharp2md/docs/architecture/quality-and-security.md#security)).

## Critério de retirada

O fallback deixa de participar da resolução operacional de uma classe de relação quando:

1. o compositor oficial aceita os mesmos manifests/revisões ativos ou existe um adapter estável para seu contrato final;
2. ele produz relações globais comprovadas e unresolved correlation candidates para aquela classe;
3. testes de migração confrontam o corpus de `correlated` existente com `confirmed`, `unresolved` ou rejeitado, sem promoção silenciosa;
4. o Reverser invalida ou rebaixa correlações antigas que contradigam a composição oficial;
5. o caminho oficial passa nos gates de compatibilidade, determinismo, completude e falha atômica do workspace.

Depois disso, Markdown ainda pode ser usado para navegação e explicação, mas não como fallback de correlação para as relações cobertas. O fallback pode permanecer somente para capacidades que o compositor declare ausentes, sempre rotulado e sob política explícita. Isso preserva a fronteira planejada: composição global comprovada pertence ao `csharp2md`; interpretação e wiki pertencem ao downstream ([arquitetura, Batch composition](../../../csharp2md/docs/architecture/architecture-knowledge-engine.md#batch-composition), [roadmap, workstreams 7–8](../../../csharp2md/architecture-knowledge-engine-roadmap.md#workstreams)).
