# Da reunião ao documento: webhooks de pedidos

## Sobre o desafio

Este trabalho transforma a transcrição de uma reunião técnica e a leitura de um OMS
Node.js/TypeScript em documentação acionável para uma feature de notificações outbound
de mudanças de status de pedidos. A entrega reúne produto, arquitetura, decisões e
desenho de implementação, com fontes verificáveis na reunião e no código.

A IA foi usada para explorar as fontes, estruturar os documentos e revisar coerência.
O resultado é uma proposta documental para revisão da equipe, sem implementação da
feature. Código, migrations, testes, configurações e TRANSCRICAO.md foram preservados.
O enunciado original está no [repositório base](https://github.com/devfullcycle/mba-ia-desafio-design-docs-com-ia).

## Ferramentas de IA utilizadas

- **Codex:** leitura da transcrição e do código local, organização de requisitos,
  redação de ADRs/RFC/FDD/PRD, identificação de ambiguidades e revisão dos documentos.
- **Terminal com Git e Python, operado pelo Codex:** inspeção de arquivos e diferenças,
  geração dos Markdown e validação de JSONs, links, fontes, timestamps e cobertura por IDs.
  São ferramentas auxiliares; não constituem ferramentas de IA adicionais.

Não foram usados agentes paralelos, serviços de pesquisa ou plugins para produzir os documentos.

## Workflow adotado

1. Ler o enunciado e converter seus critérios em um [checklist](docs/TODO.md).
2. Ler toda a transcrição, incluindo as decisões finais de UUID e snapshot, e explorar
   transação de pedidos, modelos Prisma, JWT, erros, logging, rotas, respostas e testes.
3. Separar decisões confirmadas, exclusões, pontos adiados e lacunas identificadas pela
   análise. Criar IDs para conectar cada unidade substantiva às evidências.
4. Produzir sete ADRs e consolidar o RFC com alternativas e pedidos de revisão.
5. Consolidar o PRD e aprofundar o FDD. O PRD teve uma primeira versão antes da conclusão
   do FDD; ambos foram revisados em conjunto para manter escopo e critérios consistentes.
6. Montar o Tracker e conferir fontes, caminhos reais, exemplos e limites das propostas.
7. Substituir este README pelo relato do processo e registrar a revisão final.

Os documentos tratam de perspectivas diferentes: PRD explica necessidade e aceite;
RFC propõe arquitetura; ADRs justificam decisões isoladas; FDD especifica implementação;
Tracker permite auditar as origens. Propostas derivadas ficam identificadas e não são
apresentadas como falas ou decisões da reunião.

## Prompts customizados

Os direcionamentos abaixo foram formulados e aplicados durante a produção e revisão
pelo assistente; são instruções de trabalho, não transcrição literal dos pedidos do aluno.

### Extração com evidências

```text
Leia integralmente TRANSCRICAO.md e o código existente. Separe requisitos confirmados,
decisões fechadas, alternativas descartadas, exclusões, pontos adiados e ambiguidades.
Para cada item, registre timestamp e falante ou arquivo real. Verifique especificamente
changeStatus, TransactionClient, JWT/roles, AppError, validação Zod, Pino e paginação.
Não derive customer_id do JWT sem comprovação e não transforme ideias adiadas em escopo.
```

### Revisão crítica do desenho

```text
Revise ADRs, RFC, FDD e PRD contra as fontes. Confronte “5 tentativas” com os cinco
intervalos de backoff; teste a afirmação de ordenação quando o primeiro evento falha;
verifique como duas secrets funcionam durante a rotação e o que o HMAC realmente assina.
Marque detalhes não decididos como propostas derivadas ou questões abertas. Não invente
SLA, infraestrutura de tracing, autorização por customer ou garantia de exactly-once.
```

### Auditoria da entrega

```text
Confira todos os critérios do enunciado. Verifique IDs no Tracker, percentual de fontes
TRANSCRICAO, timestamps com falantes existentes e arquivos CODIGO reais. Valide links
locais, JSONs, número e seções dos ADRs, endpoints, matriz WEBHOOK_ e seções obrigatórias.
Compare o diff para assegurar que somente documentação foi alterada. Diferencie teste
da documentação de teste da feature, que não foi implementada.
```

## Iterações e ajustes

Foram realizados **três ciclos principais de produção e revisão pelo assistente**.
Não representam três rodadas de aprovação humana; a revisão do aluno permanece importante.

1. **Contextualização e primeira redação:** o desenho inicial encontrou a contradição
   entre cinco tentativas e cinco intervalos. Em vez de escolher silenciosamente uma
   contagem, o RFC e o FDD registram as duas interpretações e pedem decisão. A intenção
   de ordenar por pedido também foi distinguida da garantia real durante retries.
2. **Revisão de integração e segurança:** a leitura confirmou que JWT não contém
   customerId e que o middleware Zod gera VALIDATION_ERROR, não WEBHOOK_INVALID_URL.
   Os contratos explicitam customer, os erros transversais foram preservados e o FDD
   especifica adaptação local para erros do domínio. A revisão também distinguiu o
   timestamp do corpo do header não coberto pelo HMAC e deixou o protocolo de rotação
   como pendência, evitando prometer uma transição de chaves já resolvida.
3. **Auditoria de rastreabilidade e consistência:** fontes complementares CODIGO foram
   adicionadas para status HTTP, paginação, JWT, logger e integração dos ADRs, pois os
   contratos também dependem desses padrões. Caminhos futuros ficaram separados dos
   arquivos reais. Probabilidade/impacto dos riscos, regras de replay e detalhes de
   serialização foram identificados como análises/propostas derivadas, não decisões
   inventadas da reunião. Exemplos e links foram validados e o checklist foi atualizado.

A reunião não fornece todos os detalhes necessários à implementação. As pendências
ficam visíveis no RFC: contagem de retry, ordenação com falhas, rotação, recuperação após
crash, autorização por customer e algumas semânticas operacionais. Documentar a lacuna
é mais fiel às fontes do que apresentar um detalhe arbitrário como aprovado.

## Como navegar a entrega

| Ordem | Arquivo | Finalidade |
|---|---|---|
| 1 | [PRD](docs/PRD.md) | Necessidade, escopo, requisitos e aceite. |
| 2 | [RFC](docs/RFC.md) | Proposta concisa, alternativas, riscos e questões para revisão. |
| 3 | [ADRs](docs/adrs/README.md) | Sete registros numerados de decisões arquiteturais. |
| 4 | [FDD](docs/FDD.md) | Fluxos, modelagem proposta, contratos, erros e integração. |
| 5 | [Tracker](docs/TRACKER.md) | Referências cruzadas às evidências. |
| 6 | [Revisão final](docs/REVISAO.md) | Verificações da documentação e limites. |
| 7 | [TODO](docs/TODO.md) | Checklist e andamento da entrega. |

A fonte primária é [TRANSCRICAO.md](TRANSCRICAO.md); o código existente complementa a
análise. O Tracker cobre 124 IDs substantivos, com 131 linhas, 107 fontes TRANSCRICAO
(81,7%) e 24 CODIGO. A contagem de IDs é uma medida operacional de cobertura, não
prova automática de completude semântica de cada frase.

## Validação e entrega

A revisão local verificou exemplos JSON, links relativos, nomes/seções dos sete ADRs,
existência das fontes de código, correspondência de timestamps/falantes e cobertura dos
IDs. O diff confirmou preservação da transcrição e dos arquivos da aplicação. Não foi
necessário executar a aplicação nem os seus testes: a feature não foi implementada.

A publicação destas alterações no fork público é a etapa de entrega posterior à revisão
local. Este README não afirma que houve push ou aprovação dos participantes da reunião.
