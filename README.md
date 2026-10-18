# Da reunião ao documento: webhooks de pedidos

## Sobre o desafio

Neste desafio do MBA, produzi com apoio de IA um pacote de documentação para uma
funcionalidade de webhooks de mudanças de status de pedidos. As fontes foram a
transcrição de uma reunião técnica e o código existente de um OMS em Node.js e
TypeScript. A entrega reúne PRD, RFC, FDD, sete ADRs e um Tracker de rastreabilidade.

Meu papel foi solicitar a organização do trabalho, ler os documentos gerados,
questionar pontos que não ficaram claros e pedir ajustes. O Codex fez a exploração
das fontes, a redação e as verificações técnicas. A entrega é documental: a aplicação
não foi modificada. O [enunciado local](Orientacoes) e o
[repositório base](https://github.com/devfullcycle/mba-ia-desafio-design-docs-com-ia)
podem ser consultados para conferir os requisitos.

## Ferramentas de IA utilizadas

- **Codex:** usei para interpretar o enunciado, analisar a transcrição e o código,
  organizar o TODO, produzir os documentos e discutir minhas dúvidas durante a revisão.
- **Git e Python no terminal, operados pelo Codex:** ferramentas auxiliares para
  conferir alterações, links, exemplos JSON, fontes e publicar a documentação no GitHub.
  Não são ferramentas de IA adicionais.

## Workflow adotado

1. Pedi ao Codex que lesse as orientações e explicasse o que precisava ser entregue.
2. Solicitei um TODO e a execução por etapas, com explicações durante o trabalho.
3. O assistente leu a transcrição e explorou os padrões do código. Produziu primeiro
   sete ADRs, depois o RFC, uma primeira versão do PRD e o FDD, completando o Tracker.
4. O assistente fez conferências de fontes, contratos, links e preservação do código.
   As ambiguidades encontradas ficaram visíveis, sem transformá-las em decisões aprovadas.
5. Li os arquivos e, ao revisar o conteúdo, questionei o adiamento do email, as
   consequências de remover configurações e a expressão sobre perder a Atlas.
6. Pedi o registro das minhas revisões e os ajustes nos documentos. Também esclareci
   com a IA que o README deveria relatar meu processo completo, incluindo ferramentas,
   prompts e revisões, em vez de conter apenas uma lista de correções.
7. Solicitei a conferência final e a publicação da versão revisada no fork público.

## Prompts customizados

Abaixo estão pedidos que escrevi durante a conversa, com ajustes de ortografia e
formatação. Eles mostram a organização inicial e minhas revisões dirigidas ao conteúdo.
Os direcionamentos técnicos internos do assistente não são apresentados como prompts meus.

### Organização da execução

```text
Monte um TODO e vá fazendo e me explicando passo a passo.
```

### Revisão do escopo de alertas

```text
No PRD, não entendi por que o envio de email ficou para fases futuras.
Acredito que seja importante implementar agora para facilitar a identificação
de problemas na funcionalidade.
```

### Revisão das consequências de remoção

```text
No PRD, fiquei em dúvida sobre estes dois requisitos:
PRD-FR-02 — Editar configuração do webhook por PATCH.
PRD-FR-03 — Remover configuração por DELETE.
Se eu fizer um DELETE de todas as configurações, o que acontece?
```

### Revisão da clareza do risco comercial

```text
No RFC, não entendi o que quer dizer: há risco de perda da Atlas.
Acho que isso deveria ficar mais claro no RFC.
```

O último bloco reúne minha pergunta e meu pedido de ajuste, enviados em duas mensagens.

## Iterações e ajustes

Após a produção inicial e as verificações automáticas do assistente, conduzi
**três rodadas de revisão do conteúdo**, descritas abaixo. São episódios reais da
conversa; não representam aprovações técnicas dos participantes da reunião.

### 1. Alertas por email: conferir o escopo antes de alterá-lo

Ao ler o PRD, questionei o adiamento do email por considerar importante um aviso
proativo de falhas. A IA conferiu a fonte e mostrou que Larissa havia excluído
explicitamente esse recurso em [09:37]. Pedi que minha revisão fosse registrada.

O PRD foi ajustado para explicar a origem da exclusão e distinguir o acompanhamento
por histórico, logs, métricas e DLQ de um alerta proativo por email. Minha sugestão
permaneceu registrada como melhoria a considerar em uma evolução, sem contrariar o
escopo decidido. A transcrição não informa uma justificativa técnica detalhada para
o adiamento, e o texto não inventa uma.

### 2. Edição e DELETE: explicitar as consequências

Os requisitos “editar por PATCH” e “remover por DELETE” estavam curtos demais para
eu entender o comportamento. Perguntei o que aconteceria se todos os cadastros de
webhook de um cliente fossem removidos e pedi o registro da revisão.

O PRD passou a esclarecer que, sem endpoints interessados, novas mudanças de status
não geram eventos na outbox para esse cliente ([09:33] Marcos; [09:34] Bruno), enquanto
os pedidos continuam funcionando. Um cadastro posterior não recupera automaticamente
eventos que não foram registrados. A discussão também evidenciou que o destino de
entregas já pendentes não foi definido na reunião: a lacuna ficou explícita, com
referência ao FDD e a RFC-OPEN-08. Nenhuma política de cancelamento foi inventada.

### 3. Atlas: esclarecer o significado de um risco

No RFC, não entendi a expressão “há risco de perda da Atlas”. Pedi esclarecimento
e solicitei que o texto fosse mais claro. A fonte era a fala de Marcos em [09:00]:
a Atlas Comercial poderia migrar para um concorrente caso a funcionalidade não fosse
entregue até o fim do trimestre.

O RFC foi corrigido para explicitar que se trata do risco comercial de perder a
Atlas como cliente. O documento mantém a necessidade de confirmar o prazo comercial,
pois a reunião também menciona fim de novembro. Este README registra como minha
pergunta resultou na melhoria da redação.

### Verificações de apoio feitas pelo assistente

Além das minhas revisões, o Codex identificou ambiguidades de contagem de retries,
ordenação durante falhas e protocolo de rotação de secrets; conferiu que o JWT não
contém customerId e que os erros Zod atuais são genéricos; e validou caminhos, fontes,
links e JSONs. Essas verificações foram feitas pelo assistente, não são revisões
minhas. As questões técnicas não resolvidas permanecem sinalizadas no RFC para a
equipe avaliar antes da implementação.

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

A documentação foi publicada no [fork público](https://github.com/gadrianosl/mba-ia-desafio-design-docs-com-ia)
em 2026-10-17, após leitura dos arquivos pelo aluno. Isso não representa aprovação
técnica pelos participantes da reunião. O link do repositório pode ser usado na entrega do MBA.
