# Plano de trabalho

- [x] 1. Ler o enunciado e identificar critérios obrigatórios.
- [x] 2. Ler integralmente a transcrição e explorar código, autenticação, transação, erros e testes.
- [x] 3. Separar decisões, requisitos, exclusões e ambiguidades com evidências.
- [x] 4. Formalizar as decisões em ADRs.
- [x] 5. Consolidar a proposta no RFC.
- [x] 6. Detalhar implementação e contratos no FDD.
- [x] 7. Consolidar produto e critérios no PRD.
- [x] 8. Conferir rastreabilidade, timestamps, links e caminhos reais.
- [x] 9. Registrar prompts, iterações e navegação no README.
- [x] 10. Revisar checklist do enunciado e garantir que código e transcrição não mudaram.
- [ ] 11. Publicar a documentação no fork público (etapa de entrega no GitHub).

## Método

Cada item substantivo recebe ID e fonte no TRACKER. Detalhamentos derivados das fontes
são identificados como propostas para revisão, nunca como decisões da reunião.
A implementação da feature não faz parte desta entrega.

## Lacunas encontradas

1. Cinco tentativas versus cinco intervalos de retry: reunião não esclarece contagem.
2. Ordem por pedido durante retries não é resolvida apenas com single-worker.
3. Rotação com duas secrets válidas não define protocolo de assinatura na janela.
4. Segurança por customer não existe no JWT; não inventar isolamento multi-tenant.
5. Sem definição de batch, recuperação após crash, encoding de assinatura e política de respostas HTTP.

## Resultado da revisão local

Sete ADRs e todos os documentos concluídos. Tracker: 124 IDs substantivos, 131 linhas,
81,7% com fonte TRANSCRICAO e 24 fontes CODIGO. Links, exemplos JSON, falantes/timestamps
e caminhos foram conferidos. Ver [REVISAO](REVISAO.md) para os limites e pendências técnicas.
Publicação no GitHub permanece como etapa posterior à revisão local.
