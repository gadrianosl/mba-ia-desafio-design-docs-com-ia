# PRD — Sistema de Webhooks de Notificação de Pedidos

Status: versão documental para revisão. Autor: Adriano Lima com assistência do Codex.
Data: 2026-10-14. Referências: [RFC](RFC.md), [FDD](FDD.md), [ADRs](adrs/README.md), [Tracker](TRACKER.md).
Os IDs identificam unidades verificáveis e permitem rastrear requisitos sem duplicar a especificação técnica.

## Resumo e contexto da feature

**PRD-CTX-01.** Oferecer notificações outbound de mudanças de status de pedidos para
integrações B2B. O OMS já gerencia pedidos, estoque e histórico; a nova capacidade
substitui a necessidade de consultas recorrentes para descobrir alterações.

## Problema e motivação

**PRD-PROB-01.** Atlas Comercial, MaxDistribuição e Nova Cargo pediram notificações.
Hoje consultam GET /orders repetidamente, com lentidão e custo. A Atlas sinalizou
risco de migração. A reunião cita fim do trimestre e fim de novembro; confirmar
prazo comercial com Marcos, sem deduzir data calendário.

## Público-alvo e cenários de uso

**PRD-USER-01.** Clientes B2B integram seus sistemas para reagir à alteração de pedidos;
um receptor pode querer somente SHIPPED e DELIVERED. Usuários autenticados gerenciam
configurações pela API; administradores recuperam falhas definitivas por replay.
O JWT representa usuário, sem customer_id, e não fornece isolamento por cliente.

## Objetivos e métricas de sucesso

| ID | Objetivo | Métrica e meta | Validação |
|---|---|---|---|
| PRD-MET-01 | Tornar a mudança conhecida pelo receptor rapidamente | Tempo entre mudança confirmada e recebimento: abaixo de 10s, expectativa declarada pelos clientes | Medir fim a fim com receptor disponível; validar condições de carga com o time. Percentil e SLA não foram definidos. Retries por indisponibilidade não podem cumprir esse limite. |

Não foram definidas metas numéricas de redução de polling, receita ou disponibilidade;
não as converter em compromissos. Estimativa discutida: três sprints com revisão de
segurança incluída; prazo comercial depende de confirmação.

## Escopo

**PRD-SCOPE-01.** Entrega inclui configuração via API, filtros por status, envio
assíncrono confiável, segurança de assinatura, histórico e recuperação administrativa.
Detalhes dos contratos ficam no FDD; justificativas arquiteturais ficam nos ADRs.

### Fora de escopo

| ID | Item | Situação na reunião |
|---|---|---|
| PRD-OUT-01 | Email de alerta/fallback após falhas | Adiado para fase futura. |
| PRD-OUT-02 | Dashboard visual | Projeto separado do frontend; fase atual só tem endpoints. |
| PRD-OUT-03 | Rate limiting de saída | Observar e decidir depois. |
| PRD-OUT-04 | Escala com múltiplos workers e particionamento | Evolução futura; fase inicial single-worker. |
| PRD-OUT-05 | Arquivamento dos eventos entregues | Fora da feature; “30 dias ou assim” não é política fechada. |
| PRD-OUT-06 | Webhooks inbound | Clientes recebem notificações; não enviam eventos para o OMS. |

**Esclarecimento de PRD-OUT-01 após revisão do aluno.** O email de alerta foi
explicitamente adiado por Larissa em [09:37], após Marcos solicitar um aviso quando
houvesse falhas consecutivas. A exclusão reflete o escopo decidido na reunião;
a transcrição não detalha uma justificativa técnica para o adiamento. Nesta fase,
o histórico permite consultar resultados das entregas, e o desenho técnico descreve
logs, métricas e DLQ para acompanhar e investigar falhas (ver [FDD](FDD.md),
seções de observabilidade e retry/DLQ). Esses mecanismos não equivalem a um alerta
proativo por email. A sugestão do aluno de antecipar esse alerta está registrada no
[README](../README.md) como contribuição de revisão, sem alterar o escopo aprovado.

## Requisitos funcionais

| ID | Requisito |
|---|---|
| PRD-FR-01 | Cadastrar endpoint por customer, com URL e lista de status; plataforma gera secret e devolve na criação. |
| PRD-FR-02 | Editar configuração do webhook por PATCH; campos editáveis e efeitos sobre entregas já pendentes são detalhados como proposta no FDD e exigem revisão. |
| PRD-FR-03 | Remover configuração por DELETE, retirando esse cadastro da seleção para novos eventos; o tratamento das entregas já pendentes permanece em aberto (RFC-OPEN-08). |
| PRD-FR-04 | Listar webhooks de um customer por GET. |
| PRD-FR-05 | Filtrar endpoints interessados no novo status na inserção; sem interessados, não inserir evento. |
| PRD-FR-06 | Registrar evento atomicamente com changeStatus, revertendo a transação se a inserção falhar. |
| PRD-FR-07 | Processar eventos assincronamente por worker separado, com polling de 2s. |
| PRD-FR-08 | Retentar falhas com backoff e enviar esgotados para DLQ persistida; contagem exata pendente em RFC-OPEN-01. |
| PRD-FR-09 | Permitir replay manual de DLQ por ADMIN, com auditoria do responsável. |
| PRD-FR-10 | Consultar histórico com sucesso/falha, payload, resposta e tempo de resposta. |
| PRD-FR-11 | Rotacionar secret por endpoint, com validade paralela da antiga por 24h. |
| PRD-FR-12 | Enviar identificação do evento para deduplicação pelo receptor, aceitando duplicatas. |
| PRD-FR-13 | Autenticar gerenciamento por JWT, permitindo ADMIN e OPERATOR; informar customer na requisição, sem extraí-lo do JWT. |

**Esclarecimento de PRD-FR-02/03/05 após revisão do aluno.** Se todos os webhooks
de um customer forem removidos, nenhuma nova notificação será gerada para ele,
pois o filtro na inserção só considera endpoints interessados ([09:33] Marcos;
[09:34] Bruno). Isso não impede mudanças de status dos pedidos: sem interessados,
a operação segue sem inserir evento de webhook. Um cadastro posterior não implica
reenvio automático das mudanças ocorridas sem assinatura, porque esses eventos não
foram registrados na outbox. A reunião não define o que fazer com entregas que já
estavam pendentes quando o endpoint foi editado ou removido. Essa regra deve ser
fechada antes da implementação; remoção lógica e preservação de histórico são
propostas no [FDD](FDD.md), e não decisões confirmadas na reunião.

## Requisitos não funcionais

| ID | Requisito |
|---|---|
| PRD-NFR-01 | Exigir HTTPS e HMAC-SHA256 sobre o corpo, com secret única por endpoint. |
| PRD-NFR-02 | Recusar payload acima de 64KB, sem truncar. Unidade e tratamento propostos no FDD precisam de revisão. |
| PRD-NFR-03 | Timeout de envio HTTP de 10s; expiração conta como falha para retry. |
| PRD-NFR-04 | Preservar snapshot do evento; evitar inclusão de items. |
| PRD-NFR-05 | Manter intenção de ordenação por pedido em single-worker, sem garantia global; resolver retry em RFC-OPEN-02. |
| PRD-NFR-06 | Reutilizar MySQL, stack e padrões do projeto; não adicionar Redis nesta fase. |

## Decisões e trade-offs principais

**PRD-DEC-01.** As decisões estão nos [ADRs](adrs/README.md): troca-se envio imediato
na transação por isolamento do receptor; aceita-se duplicação para reduzir coordenação;
single-worker e MySQL reduzem infraestrutura, mas limitam vazão. Não há garantia de
sucesso se o receptor permanece indisponível até o esgotamento dos retries.

## Dependências

**PRD-DEP-01.** Dependemos do fluxo de mudança de status do OMS, MySQL, JWT existente e
receptor HTTPS preparado para verificar assinatura e deduplicar. Marcos orientará
clientes no portal de desenvolvedor. Sofia precisa de pelo menos dois dias úteis
para revisão de segurança antes do deploy, dentro da estimativa de três sprints.

## Riscos e mitigação

Probabilidades e impactos abaixo são avaliações qualitativas propostas nesta análise,
derivadas das falas, e não estimativas aprovadas na reunião.

| ID | Risco | Probabilidade proposta | Impacto proposto | Mitigação |
|---|---|---|---|---|
| PRD-RISK-01 | Receptor indisponível por horas | Média | Alto: notificações atrasadas ou em DLQ | Backoff, evidência persistida e replay administrativo. |
| PRD-RISK-02 | Vazamento de secret | Média | Alto: autenticidade comprometida no endpoint | Secret individual, rotação e revisão de segurança. |
| PRD-RISK-03 | Backlog impedir meta de latência | Média | Alto: integração recebe tarde | Medir fila e latência, validar batch e capacidade; não prometer escala não testada. |
| PRD-RISK-04 | Duplicata causar efeito repetido no receptor | Média | Alto: processamento duplicado | Comunicar at-least-once e deduplicação por X-Event-Id. |

## Critérios de aceitação

| ID | Cenário verificável |
|---|---|
| PRD-AC-01 | Usuário autenticado cadastra, edita, lista e remove webhook; criação devolve secret. Requisição sem JWT é recusada. |
| PRD-AC-02 | Transição válida cria snapshot apenas para endpoints interessados; erro na outbox reverte pedido, histórico e estoque. |
| PRD-AC-03 | Receptor disponível recebe evento assinado; medir latência contra PRD-MET-01 no cenário acordado. |
| PRD-AC-04 | Timeout gera retry; esgotamento gera DLQ; OPERATOR não faz replay e ADMIN deixa auditoria. Contagem depende de RFC-OPEN-01. |
| PRD-AC-05 | Rotação permite migração com 24h de sobreposição; validar protocolo com Sofia antes da implementação. |
| PRD-AC-06 | Histórico permite inspecionar payload, resposta, duração e resultado; duplicatas carregam a mesma identidade do evento. |
| PRD-AC-07 | URLs sem HTTPS e payload acima do limite são recusados, sem truncamento. |

## Estratégia de testes e validação

**PRD-TEST-01.** Plano derivado dos requisitos: testes de API para CRUD/JWT/ADMIN;
integração MySQL para commit e rollback; relógio controlado para retry e rotação;
receptor HTTP de teste para assinatura, timeout, duplicata e latência; regressão da
máquina de estados e estoque. Conferir implementação com os testes atuais de pedidos.
Esta entrega especifica os testes, sem implementar ou afirmar que a feature já foi testada.
