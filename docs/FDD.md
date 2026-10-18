# FDD — Implementação de webhooks de pedidos

Autor: Adriano Lima com assistência do Codex. Data: 2026-10-14.
Status: especificação para revisão, com bloqueios identificados no [RFC](RFC.md).
Referências: [PRD](PRD.md), [ADRs](adrs/README.md), [Tracker](TRACKER.md).

## Contexto e motivação técnica

**FDD-CTX-01.** O OMS não possui mecanismo de notificação externa. changeStatus já
controla pedido, histórico e estoque numa transação; o novo evento deve participar
desse commit e ser enviado depois, sem HTTP dentro da transação.

## Objetivos técnicos

**FDD-OBJ-01.** Tornar o registro do evento atômico, preservar snapshot, isolar o envio
num processo separado e permitir diagnóstico e recuperação de falhas. A meta de
produto é menos de 10s; polling de 2s e timeout de 10s não provam essa meta sob carga.

## Escopo e exclusões

**FDD-SCOPE-01.** Inclui configuração, filtro, outbox, worker, assinatura, retry,
DLQ, histórico, rotação e replay ADMIN. Exclui email, dashboard, rate limiting,
arquivamento e múltiplos workers. Webhooks são somente outbound. A criação inicial
de pedido não foi definida como evento; o ponto acordado é changeStatus.

### Como interpretar os detalhes

As decisões confirmadas estão nos ADRs. Itens marcados **proposta derivada** detalham
como implementar a intenção da fonte sem alegar aprovação na reunião. Nomes de
campos internos, novos códigos específicos e rotas não ditadas literalmente são
propostas para revisão. Questões abertas não se tornam requisitos aprovados.

## Modelagem proposta

**FDD-DATA-01 — proposta derivada.** Adicionar modelos em futura migration Prisma,
sem executar a alteração nesta entrega. IDs de entidades seguem UUID. Timestamps
são instantes UTC; a representação externa usa ISO 8601.

| Estrutura futura | Campos e finalidade |
|---|---|
| webhook_endpoints | id, customer_id, url HTTPS, secret atual, secret anterior e expiração, active, lista de status, created_at, updated_at. Gerar a secret na plataforma; modo seguro de armazenamento depende de Sofia. |
| webhook_outbox | id da entrega, event_id, webhook_id, order_id, corpo renderizado, status, contador de tentativas, next_attempt_at, created_at e último erro. Corpo imutável em retentativas. |
| webhook_deliveries | id da tentativa, outbox_id, event_id, webhook_id, instante, resultado, HTTP status se houver, resposta, duração e erro. Uma linha por tentativa concluída; crash pode impedir o registro da tentativa. |
| webhook_dead_letter | id, referência da entrega, event_id, webhook_id, payload, motivo final, timestamp e referência ao replay quando houver. |

A tabela outbox terá índices de status e created_at conforme reunião. Índice composto
por status/next_attempt_at/created_at é proposta para revisar o plano de consulta.
Representação da lista de status, tamanho do batch, armazenamento de resposta e
restrições de integridade devem ser validados antes de fechar a migration.

**FDD-DATA-02 — proposta derivada.** Separar identidade lógica do evento e da entrega:
uma mudança gera um event_id; cada endpoint interessado tem sua linha de outbox com
id próprio e mesmo snapshot/event_id. Deduplicação do receptor usa event_id no contexto
do webhook_id. Não deduplicar só event_id entre assinaturas distintas. A call não
fecha a cardinalidade; validar essa proposta com Bruno/Diego. Replay conserva event_id
e corpo para permitir deduplicação, sem fingir que é nova mudança de status.

## Fluxos detalhados

### Criação transacional

**FDD-FLOW-01.** Dentro da transação existente de changeStatus:

1. Consultar pedido e validar a transição com as regras existentes.
2. Debitar/repor estoque conforme a transição, atualizar pedido e criar histórico.
3. Chamar publishWebhookEvent(tx, order, fromStatus, toStatus), usando o mesmo tx.
4. Selecionar endpoints ativos do customer cujo filtro inclui toStatus. Sem resultado, retornar sem outbox.
5. Gerar identidade UUID, montar snapshot com o novo status e dados daquela transação; medir o corpo antes de inserir.
6. Inserir as entregas pendentes; qualquer erro propaga para rollback de toda a transação.
7. Retornar o pedido como hoje. Nenhuma chamada de rede externa acontece aqui.

Passos de serialização e identidade seguem FDD-DATA-02 e FDD-PAYLOAD-02 como propostas.
Não ler o pedido novamente no worker para reconstruir o evento.

### Processamento pelo worker

**FDD-FLOW-02.** Processo separado, mesma DATABASE_URL, instância própria do Prisma,
loop de polling a cada 2s e batch pequeno dos pendentes mais antigos. Não abrir um
segundo worker em paralelo nesta fase. A lógica fica no módulo futuro e a entrada
em src/worker.ts; script npm run worker é uma alteração futura prevista na reunião.

**FDD-FLOW-03 — proposta derivada.** Buscar somente entregas vencidas
(next_attempt_at <= agora), marcar processando em transação curta e enviar fora dela.
Para cada tentativa, carregar configuração necessária, assinar os bytes persistidos,
registrar instante de envio e iniciar timeout total de 10s. Persistir resultado e
histórico na mesma transação de conclusão. Não manter lock SQL durante o HTTP.
Até validar batch e ordenação, não definir concorrência de envio arbitrária.

**FDD-FLOW-04 — proposta derivada.** Em HTTP 2xx, marcar entregue. Em resposta não-2xx,
erro de rede/TLS/DNS ou timeout, registrar falha e programar retry; não seguir redirects
silenciosamente para outro destino. Tratamento diferenciado para 4xx/429/Retry-After
não foi discutido: revisar a política única com o time. Essa política não foi decisão
expressa da call e não deve ser apresentada ao cliente como contrato aprovado.

### Retry e DLQ

**FDD-RETRY-01.** Progressão declarada: 1m, 5m, 30m, 2h e 12h. A soma é 14h36 de espera,
sem tempos de execução. Também foi decidido “5 tentativas”. RFC-OPEN-01 bloqueia o
contador final. Não remover um intervalo nem acrescentar chamada como se fosse decisão.

| Interpretação a revisar | Chamadas | Esperas |
|---|---|---|
| Cinco chamadas totais | 1 inicial + 4 retries | Só admite quatro esperas; reunião não escolheu qual remover. |
| Cinco retries | 1 inicial + 5 retries | Usa as cinco esperas; total de seis chamadas, diferente da fala literal. |

Após a falha, persistir o próximo vencimento; não dormir horas dentro do processamento.
Ao atingir o teto que o time confirmar, transferir para webhook_dead_letter com corpo,
motivo e timestamp. **Proposta derivada:** inserir DLQ e retirar a entrega da seleção
ativa atomicamente; falha de persistência não deve apagar a única cópia recuperável.

**FDD-RETRY-02 — proposta derivada.** Retry e replay reutilizam o corpo/event_id;
o timestamp de envio e a assinatura podem mudar a cada chamada. Replay cria nova
execução pendente com contador reiniciado e vínculo à DLQ, registrando userId do ADMIN,
horário e IDs. Consumir o replay em transação para evitar duas reinserções concorrentes;
segunda solicitação retorna conflito. Não há descarte automático de evidência definido.

### Ordenação e recuperação

**FDD-ORDER-01.** Intenção da reunião: ordem por order_id enquanto single-worker,
sem ordem global. Ordenar por created_at não basta quando o evento anterior está em
backoff ou quando dois registros têm o mesmo instante. **Proposta derivada, pendente:**
manter sucessores do mesmo pedido/endpoint bloqueados enquanto houver predecessor
pendente/processando, inclusive em retry, e definir sequência de desempate. Outros
pedidos podem prosseguir. Decidir também se DLQ libera sucessores e se replay pode
chegar depois de eventos novos; não prometer ordenação de replay sem decisão.

**FDD-RECOVERY-01 — proposta derivada.** Como o processo pode cair após o receptor
receber, aceitar reenvio do mesmo event_id. No startup, após garantir operacionalmente
que a instância antiga terminou, devolver processando a pendente. Falha do banco
interrompe conclusão; não presumir sucesso ou criar novo event_id. A estratégia de
exclusão mútua e recuperação deve ser aprovada em RFC-OPEN-04; não usar esse procedimento
com múltiplos workers. Em shutdown, parar polling, concluir/preservar trabalho em voo
e desconectar Prisma, seguindo o estilo do server existente.

## Contratos públicos

### Convenções

**FDD-API-00 — proposta derivada de padrões existentes.** Base `/api/v1` já usada pela
aplicação. Gerenciamento aceita `Authorization: Bearer <JWT>` e JSON; ADMIN ou OPERATOR,
exceto replay. `customerId` é explícito no cadastro e na consulta. Não deriva do JWT.
Parâmetros de entidade são UUID. As rotas de CRUD e rotação abaixo detalham verbos e
intenções da reunião, mas seus paths exatos precisam de revisão (deliveries e replay
foram ditados na call). Status HTTP seguem os controllers atuais.

Exemplos usam os aliases **W** = `11111111-1111-4111-8111-111111111111`, **C** =
`22222222-2222-4222-8222-222222222222` e **D** =
`33333333-3333-4333-8333-333333333333` nos paths. Nos JSONs, IDs estão expandidos.
Secrets e assinaturas são ilustrativas, sem valor operacional.

### Cadastro — FDD-API-01

`POST /api/v1/webhooks` → **201**. Secret é gerada pelo servidor; não é entrada obrigatória.

Request:
```json
{"customerId":"22222222-2222-4222-8222-222222222222","url":"https://cliente.example/webhooks/orders","statuses":["SHIPPED","DELIVERED"]}
```

Response (proposta: iniciar active=true):
```json
{"id":"11111111-1111-4111-8111-111111111111","customerId":"22222222-2222-4222-8222-222222222222","url":"https://cliente.example/webhooks/orders","statuses":["SHIPPED","DELIVERED"],"active":true,"secret":"<secret-gerada>"}
```

Falhas: 400 para URL/lista inválida, 401 sem autenticação, 404 para customer inexistente.
Validar customer existente antes de persistir. Valores permitidos para status reutilizam
OrderStatus: PENDING, PAID, PROCESSING, SHIPPED, DELIVERED, CANCELLED. PENDING pode fazer
parte do enum, mas não há transição atual de retorno a PENDING nem evento de criação aprovado.

### Edição — FDD-API-02

`PATCH /api/v1/webhooks/W` → **200**. Alterar apenas campos presentes; customerId
permanece associado ao cadastro. Secret muda pela operação específica de rotação.

Request:
```json
{"url":"https://cliente.example/hooks/orders","statuses":["DELIVERED"],"active":true}
```

Response:
```json
{"id":"11111111-1111-4111-8111-111111111111","customerId":"22222222-2222-4222-8222-222222222222","url":"https://cliente.example/hooks/orders","statuses":["DELIVERED"],"active":true}
```

Falhas: 400, 401, 404. **Proposta:** edição do filtro afeta inserções futuras e não
reescreve snapshots já pendentes. Alteração de destino ou desativação com entregas
pendentes precisa de regra aprovada; não enviar a endpoint removido por suposição.

### Listagem por customer — FDD-API-03

`GET /api/v1/webhooks?customerId=C&page=1&pageSize=20` → **200**.
Request sem corpo; customerId deve ser UUID. Paginação proposta conforme helper atual:
page >= 1; pageSize entre 1 e 100, default 20.

Response:
```json
{"data":[{"id":"11111111-1111-4111-8111-111111111111","customerId":"22222222-2222-4222-8222-222222222222","url":"https://cliente.example/webhooks/orders","statuses":["SHIPPED","DELIVERED"],"active":true}],"pagination":{"page":1,"pageSize":20,"total":1,"totalPages":1}}
```

Falhas: 400, 401. Proposta: customer inexistente retorna 404; existente sem cadastro,
coleção vazia. A listagem não expõe secrets.

### Remoção — FDD-API-04

`DELETE /api/v1/webhooks/W` → **204**, sem corpo de request e sem corpo de response.
Falhas: 400 para UUID inválido, 401, 404. **Proposta pendente:** remoção lógica impede
novos envios e preserva histórico; destino de linhas pendentes precisa de aprovação,
assim como a distinção entre cancelamento e falha em DLQ. Não apagar evidência em cascade.

### Histórico — FDD-API-05

`GET /api/v1/webhooks/W/deliveries?page=1&pageSize=100` → **200**.
Request sem corpo. Proposta de paginação igual à listagem, com ordenação de tentativas
mais recentes primeiro. O exemplo mostra tentativa bem-sucedida:

```json
{"data":[{"id":"33333333-3333-4333-8333-333333333333","eventId":"44444444-4444-4444-8444-444444444444","success":true,"payload":{"event_id":"44444444-4444-4444-8444-444444444444","event_type":"order.status_changed","timestamp":"2026-10-14T18:00:00.000Z","order_id":"55555555-5555-4555-8555-555555555555","order_number":"ORD-000001","from_status":"PROCESSING","to_status":"SHIPPED","customer_id":"22222222-2222-4222-8222-222222222222","total_cents":12500},"responseStatus":200,"response":"ok","durationMs":85,"attemptedAt":"2026-10-14T18:00:02.000Z"}],"pagination":{"page":1,"pageSize":100,"total":1,"totalPages":1}}
```

Falhas: 400, 401, 404. Sem resposta HTTP (timeout/rede), responseStatus/response são
null, success=false e errorCode informa a falha. Tamanho máximo e redação de respostas
armazenadas ainda precisam de definição; não incluir credentials ou secrets no histórico.

### Rotação — FDD-API-06

`POST /api/v1/webhooks/W/rotate-secret` → **200**. Request sem corpo.

```json
{"id":"11111111-1111-4111-8111-111111111111","secret":"<nova-secret-gerada>","previousSecretValidUntil":"2026-10-15T18:00:00.000Z"}
```

Falhas: 400, 401, 404. Expiração exemplifica rotação em 2026-10-14T18:00Z. Geração é
server-side. **Proposta:** se já houver janela aberta, devolver 409 até ela expirar,
para não invalidar uma chave antes das 24h; validar essa regra, não discutida na call.
Protocolo de assinatura na janela depende de FDD-SEC-02.

### Replay administrativo — FDD-API-07

`POST /api/v1/admin/webhooks/dead-letter/D/replay` → **202**, aceitação assíncrona proposta.
Request sem corpo. Não significa que o receptor já recebeu.

```json
{"deadLetterId":"33333333-3333-4333-8333-333333333333","outboxId":"66666666-6666-4666-8666-666666666666","eventId":"44444444-4444-4444-8444-444444444444","status":"pending"}
```

Falhas: 401 sem token, 403 para OPERATOR, 404 para DLQ ausente, 409 se já recolocada
nessa execução. Reutilizar requireRole('ADMIN') e registrar userId do solicitante.
Replays de uma nova falha após tentativa anterior exigem uma nova entrada DLQ vinculada;
política exata de manutenção dessa cadeia deve ser revisada com Diego.

### Notificação outbound — FDD-PAYLOAD-01

`POST <url-cadastrada>` com o corpo snapshot ilustrado no histórico. Não incluir items;
o cliente pode consultar `/api/v1/orders/:id` para detalhes. Campo timestamp é o instante
do evento, enquanto X-Timestamp é o instante de cada envio.

```http
Content-Type: application/json
X-Event-Id: 44444444-4444-4444-8444-444444444444
X-Webhook-Id: 11111111-1111-4111-8111-111111111111
X-Timestamp: 2026-10-14T18:00:02.000Z
X-Signature: <HMAC-SHA256-do-corpo>
```

**FDD-PAYLOAD-02 — proposta derivada.** Renderizar JSON uma vez em UTF-8 e persistir
exatamente esse corpo; assinar e enviar os mesmos bytes, sem reserializar. Interpretar
64KB como 65.536 bytes e aceitar <= limite, rejeitando acima, sem truncamento; Sofia
deve confirmar KB/KiB antes de fechar testes de fronteira. Payload excessivo gera erro
antes do commit de changeStatus, coerente com atomicidade; revisar esse impacto.

## Matriz de erros previstos

**FDD-ERR-01 — proposta derivada.** Códigos do domínio seguem WEBHOOK_, usando AppError
ou subclasses locais. Nomes além dos exemplos falados são propostas, não requisitos
novos. HTTP refere-se à API de gerenciamento ou transação, não à resposta do receptor.

| Código | HTTP / contexto | Condição e tratamento |
|---|---|---|
| WEBHOOK_NOT_FOUND | 404 | Configuração não existe. |
| WEBHOOK_INVALID_URL | 400 | URL inválida ou sem HTTPS; recusar antes de persistir. |
| WEBHOOK_INVALID_STATUSES | 400 | Lista vazia ou valor fora do enum, conforme schema proposto. |
| WEBHOOK_CUSTOMER_NOT_FOUND | 404 | Customer informado não existe. |
| WEBHOOK_PAYLOAD_TOO_LARGE | 422 em changeStatus / falha no worker se legado inválido | Corpo ultrapassa limite; não truncar nem enviar. Na publicação transacional, rollback. |
| WEBHOOK_TIMEOUT | Worker, sem HTTP próprio | Timeout de 10s; registrar e aplicar retry. |
| WEBHOOK_DELIVERY_FAILED | Worker, sem HTTP próprio | Resposta não-2xx ou falha de transporte; registrar motivo e retry proposto. |
| WEBHOOK_SECRET_REQUIRED | Worker, sem HTTP próprio | Chave necessária ao envio ausente; tratar falha sem enviar sem assinatura; investigar persistência. Não exigir secret no POST de cadastro. |
| WEBHOOK_DEAD_LETTER_NOT_FOUND | 404 | Entrada DLQ inexistente. |
| WEBHOOK_REPLAY_CONFLICT | 409 | Entrada já recolocada pela mesma operação de replay. |
| WEBHOOK_ROTATION_IN_PROGRESS | 409 | Regra proposta para sobreposição de rotações. |

**FDD-ERR-02.** Compatibilidade: authenticate e requireRole já produzem UNAUTHORIZED
(401) e FORBIDDEN (403). validate transforma Zod em VALIDATION_ERROR (400), enquanto
middleware central também trata Prisma com códigos genéricos. Esses erros transversais
continuam com seus códigos existentes. Para URL/status com erro específico do módulo,
mapear localmente o resultado de safeParse para AppError WEBHOOK_*; simplesmente passar
Zod ao validate atual não produz esses códigos. Demais erros de schema podem permanecer
VALIDATION_ERROR. O middleware central já serializa AppError como abaixo:

```json
{"error":{"code":"WEBHOOK_INVALID_URL","message":"Webhook URL must use HTTPS","details":{"field":"url"}}}
```

## Segurança

**FDD-SEC-01.** Confirmado: HTTPS, HMAC-SHA256 sobre o corpo e secret por endpoint.
**Proposta derivada:** gerar secret com fonte criptograficamente segura; evitar retornar
em GET/PATCH, logs ou histórico. Não registrar a configuração inteira em Pino: a redação
atual não contém o campo secret. Sofia deve definir armazenamento recuperável e gestão
de chaves, pois hash irreversível não basta para assinar. Revisar destinos de rede e
redirecionamento para evitar acesso indevido a recursos internos; isso é avaliação de
segurança derivada do envio externo, não uma política SSRF já definida na reunião.

**FDD-SEC-02 — proposta derivada, bloqueada para revisão.** Usar HMAC-SHA256 dos bytes
UTF-8 do corpo, codificado em hexadecimal minúsculo sem prefixo no X-Signature. O cliente
compara em tempo constante. Na rotação, proposta de enviar com a nova chave e manter
ambas aceitas no verificador do receptor por 24h; a plataforma conserva anterior até
expirar. Isso exige que o receptor prepare a nova chave antes de receber o próximo envio.
A reunião não resolve a coordenação nem define múltiplas assinaturas; Sofia deve aprovar
um protocolo antes de oferecer essa operação. Não afirmar que guardar a chave antiga
no servidor, sozinho, permite validar envios novos com ela.

**FDD-SEC-03.** X-Timestamp foi proposto para detecção opcional de replay pelo cliente.
Como a assinatura acordada cobre só o corpo, o header não está autenticado por HMAC.
Não alegar proteção completa contra replay: cliente deve deduplicar event_id e pode
avaliar o timestamp do corpo assinado. Janela de aceitação ou assinatura composta
não foram definidas e exigem revisão sem alterar silenciosamente o contrato da call.

## Estratégias de resiliência

**FDD-RES-01.** Timeout HTTP 10s, backoff persistido, teto pendente, DLQ persistida e
replay são a recuperação prevista. Não há fallback por email nesta fase. Persistir
antes de considerar entrega finalizada; após resultado incerto, reenvio pode duplicar.
Propostas de recuperação após crash e semântica HTTP estão nos fluxos acima. Backlog
ou erro do banco não justificam perder evento nem criar garantia de exactly-once.

## Observabilidade

**FDD-OBS-01 — proposta derivada.** A call exige reuso de Pino e auditoria de replay;
não escolhe stack de métricas/tracing. Instrumentação abaixo é desenho proposto com
base em polling, retries, histórico e meta de latência, sem afirmar infraestrutura instalada.

- Métricas: quantidade pendente, idade do mais antigo, latência evento→recebimento no teste e evento→confirmação em produção, duração HTTP, sucesso/falha por categoria, retries e entradas DLQ. Não definir alertas ou metas novas sem validação.
- Logs Pino: event_id, outbox_id, webhook_id, order_id, attempt, status, durationMs e errorCode; replay inclui adminUserId e deadLetterId. O logger existente registra requestId/userId na API. Não logar secret, Authorization nem corpo/resposta sensível indiscriminadamente.
- Tracing: correlacionar transação/publicação, processamento e tentativa por event_id/outbox_id; requestId pode ligar chamada original e replay. Correlação estruturada em logs é viável com Pino; spans distribuídos e exporter dependem de escolha futura, sem adicionar OpenTelemetry como requisito aprovado.

## Integração com o sistema existente

Os caminhos da tabela existem hoje. Caminhos futuros são apresentados separadamente.

| ID | Arquivo real | Integração prevista, sem alteração nesta entrega |
|---|---|---|
| FDD-INT-01 | src/modules/orders/order.service.ts | changeStatus chama publishWebhookEvent com tx dentro do $transaction antes de retornar. Preservar validação, estoque e histórico. |
| FDD-INT-02 | prisma/schema.prisma | Acrescentar modelos de configuração, outbox, tentativas e DLQ, relacionamentos e índices; MySQL e UUID já são padrões. |
| FDD-INT-03 | src/middlewares/auth.middleware.ts | Reutilizar authenticate e requireRole('ADMIN') para replay. JWT contém id/email/role, sem vínculo customer. |
| FDD-INT-04 | src/shared/errors/app-error.ts | Base das classes de erro WEBHOOK_* com statusCode/errorCode/details. |
| FDD-INT-05 | src/middlewares/error.middleware.ts | Serializa AppError sem nova stack; considerar códigos genéricos de Zod/Prisma. |
| FDD-INT-06 | src/middlewares/validate.middleware.ts | Reutilizar validação comum; mapear localmente erros específicos se precisar de WEBHOOK_. |
| FDD-INT-07 | src/shared/logger/index.ts | Reutilizar Pino; impedir exposição de secret não coberta pelas regras atuais. |
| FDD-INT-08 | src/config/database.ts | Reutilizar fábrica/configuração; cada processo cria sua instância Prisma e conecta à mesma DATABASE_URL. |
| FDD-INT-09 | src/routes/index.ts | Acrescentar router de webhooks e rota administrativa, usando padrão de montagem atual. |
| FDD-INT-10 | src/app.ts | Construir repository/service/controller em buildControllers e manter prefixo /api/v1 e middleware central. |
| FDD-INT-11 | src/shared/http/response.ts | Reutilizar paginated para listagens e histórico. |
| FDD-INT-12 | src/modules/orders/order.status.ts | Manter máquina de estados e regras de estoque; reutilizar enum, sem novas transições. |
| FDD-INT-13 | src/server.ts | Referência para lifecycle e encerramento da entrada futura do worker, sem executar worker no servidor da API. |
| FDD-INT-14 | tests/orders.test.ts | Referência para testes futuros de rollback e regressão de status/estoque; não modificar no desafio documental. |

**FDD-NEW-01.** Caminhos ainda inexistentes, previstos como parte da implementação
futura: `src/worker.ts` e `src/modules/webhooks/` com controller/service/repository/routes/
schemas e processor ou worker. Não confundir esses caminhos planejados com evidência
existente. Scripts e migrations também só serão alterados na fase de implementação.

## Dependências e compatibilidade

**FDD-DEP-01.** Base atual: Node >=20, TypeScript, Express 4.21.1, Prisma 5.22.0/MySQL,
Zod 3.23.8, Pino 9.5.0, JWT e uuid. Versões são as declaradas em package.json, sem
recomendar atualização. Não há pacote atual de fila, tracing ou cliente HTTP dedicado;
a escolha de mecanismo HTTP não foi fechada. Preservar contratos atuais de pedidos,
erros transversais e envelope de paginação. Revisão de segurança deve preceder deploy.

## Critérios de aceite técnicos

Plano de validação derivado das fontes; testes abaixo não foram executados porque a
feature não foi implementada nesta entrega documental.

| ID | Verificação futura |
|---|---|
| FDD-AC-01 | Falha induzida na inserção outbox desfaz status, histórico e estoque; commit cria snapshot só para endpoints ativos interessados. |
| FDD-AC-02 | Alteração posterior do pedido não muda corpo persistido; HMAC verificado sobre bytes enviados; retry conserva event_id. |
| FDD-AC-03 | Relógio controlado valida polling 2s, timeout 10s, vencimentos e DLQ conforme contagem a confirmar. |
| FDD-AC-04 | API cobre CRUD, filtro, histórico, rotação; HTTP sem TLS é rejeitado; fronteiras do limite de payload são testadas após confirmar unidade. |
| FDD-AC-05 | OPERATOR recebe 403 no replay; ADMIN recebe aceitação, cria outbox e deixa auditoria sem duplicar replay concorrente na proposta. |
| FDD-AC-06 | Crash entre recepção e commit não perde evento; duplicata é deduplicável. Ordem com retry e DLQ segue regra a aprovar. |
| FDD-AC-07 | Rotação testa antes/depois das 24h e coordenação do receptor conforme protocolo a aprovar. Logs/histórico não vazam secret. |
| FDD-AC-08 | Medir menos de 10s em cenário de carga acordado; registrar backlog e demora do receptor. Não substituir medição por cálculo de polling. |

## Riscos e mitigação

**FDD-RISK-01.** Escrita extra e falta de retenção podem pressionar MySQL; validar
consulta/indexação e observar volume. Single-worker com timeout de 10s pode produzir
head-of-line blocking; dimensionar com testes antes de aceitar a meta sob carga.
Retries, crash e replay trazem duplicação/ordenação incerta: confirmar regras abertas,
preservar identidade e orientar consumidor. Validade paralela de chaves e acesso por
customer precisam da revisão de Sofia; não alegar isolamento ou rotação sem interrupção
antes de aprovar os protocolos. Ver também riscos de produto no PRD.
