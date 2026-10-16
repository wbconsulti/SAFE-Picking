# Complemento — modelo dual de separação, periféricos e referência de picking

Versão documental: 2026-10-16. Estado: proposta para revisão. Nenhuma capacidade descrita aqui está homologada.

Este diretório **complementa** as diretrizes existentes. Não altera `AGENTS.md`, `docs/01`–`07`, os ADRs nem o material de `harness/`.

## Precedência

1. `AGENTS.md` e suas invariantes prevalecem sobre qualquer parte deste complemento. Nada aqui relaxa uma invariante, um requisito (R01–R10), um gate de aceite ou uma decisão de ADR.
2. Em conflito, interromper a decisão afetada, registrar o conflito e pedir resolução ao responsável. Não escolher silenciosamente a regra mais permissiva.
3. O complemento só entra em execução após autorização explícita do responsável (ver [prompt complementar](../../harness/complementos/PROMPT_COMPLEMENTAR.md)). Etapas 0 a 5 do [roadmap](../06-roadmap.md) seguem exclusivamente o fluxo **manual**.

## Convenção de evidência

Mesma de [docs/README.md](../README.md): **Requisito**, **Referência**, **Hipótese**, **Homologado**. Acrescenta-se:

- **Relato:** caso público da comunidade SAP ou guia de terceiros. Serve como alerta de risco e como origem de casos de teste; não é contrato, não é fonte oficial e nunca vale como homologação.

## Conteúdo

| Documento | Conteúdo |
|---|---|
| [C01 — Modelo dual de separação](C01-modelo-dual-de-separacao.md) | Modo de decisão (manual × sugestão assistida) e canal de confirmação (toque, leitura, híbrido); política, auditoria e telas |
| [C02 — Disponibilidade e sugestão assistida](C02-disponibilidade-e-sugestao-assistida.md) | Contrato de consulta de origens, motor de sugestão, gates e testes |
| [C03 — Periféricos e embalagem](C03-perifericos-e-embalagem.md) | Leitor, câmera, impressora, balança; embalagem como expansão da Etapa 6 |
| [C04 — Referência de picking SAP](C04-referencia-picking-sap.md) | Modelo de dados, estados, papéis e relatos de risco, conferidos com `docs/02` |
| [C05 — Fontes e evidências](C05-fontes-e-evidencias.md) | Referências C1–C17 e limites de verificação |
| [Prompt complementar](../../harness/complementos/PROMPT_COMPLEMENTAR.md) | Rodadas G e H do Harness, após autorização |

## Reconciliação com as diretrizes existentes

O material que originou este complemento foi ajustado para não contradizer `AGENTS.md` e `docs/`. O que mudou:

| Ideia original | Tratamento neste complemento | Base nas diretrizes |
|---|---|---|
| Modo "automática direta" (grava sem revisão) | **Removido.** Não deve ser proposto. | AGENTS (invariantes), ADR-001, `06` "Fora do primeiro ciclo" |
| Alocação automática com revisão | Vira **sugestão assistida**: opt-in, Etapa 6, atrás de gates; o operador aceita explicitamente cada linha | ADR-001 (sugestão futura exige aprovação), `06` Etapa 6 |
| FEFO/FIFO como estratégia | **Sem estratégia padrão.** Só critério homologado pelo cliente (DC02) | AGENTS: não implementar FIFO/FEFO sem escolha explícita e regra homologada |
| "Disponível = estoque − comprometido + pedido" | **Removido.** Medidas distintas, nuláveis, com fonte e instante; desconhecido ≠ 0 | `01` Disponibilidade; AGENTS |
| Ponto de alocação no separador | Alocação por lote é decidida no **planejamento**; a separação física só registra quantidade real | `01` Fluxo manual, passos 3 e 6 |
| Leitura de código "verifica e confirma" | Leitura **só identifica candidato**; `scan = required` apenas bloqueia | `01` passo 6, T26 |
| Stack TypeScript/Node/React prescrita | **Removida.** Stack só por ADR | AGENTS (método 4), ADR-002, D11 |
| Geração de entrega no fluxo | **Fora.** Confirmar picking não gera entrega; entrega é ação separada na Etapa 6 | R10, ADR-001, T11 |
| Registrar separação por `UpdateReleasedAllocation` (`PickedQuantity`) | Tratado como **relato**. Contrato de confirmação segue D09/T12 | `02` Sequência de comandos |
| `POST /PickLists({id})/Close` | Usar o contrato de `02` (`PickListsService_Close`); fechar não é desfazer | `02`, T25 |
| `v1`/`v2` configurável | Usar OData V4 em `/b1s/v2` | `02`, S2 |
| Flag `mock/readonly/live` | Escrita **desabilitada por padrão por capacidade**, até homologação | `02` gate, `05` gates |

## Decisões complementares pendentes

Não alteram o registro D01–D13. Todas pendentes. Nenhuma resposta deve conter credenciais.

| ID | Pergunta | Responsável esperado | Bloqueia |
|---|---|---|---|
| DC01 | O cliente quer sugestão assistida? Quem aprova o escopo e em que ponto (planejamento)? | Responsável pelo produto | Todo o C02 |
| DC02 | Qual critério de origem o cliente homologa (por classe de item/depósito)? | Dono do processo | Motor de sugestão |
| DC03 | Como o operador consulta hoje a disponibilidade na origem (tela, relatório, consulta salva)? | Dono do processo + consultor SAP | Linha de base do modo manual |
| DC04 | Política de leitura por contexto: `off`, `optional` ou `required`? Quem pode justificar divergência? | Dono do processo | Canal de leitura |
| DC05 | Quais periféricos existem e qual a prioridade? | Operação | Rodada H |
| DC06 | Onde estão os códigos de barras de item, bin e lote? Há GS1-128? | Estoque + consultor SAP | Identificação por leitura |
| DC07 | Auditoria de modo só na aplicação ou também no SAP (UDF)? Retenção? | Segurança + consultor SAP | Trilha de modo |
| DC08 | Embalagem: basta o nativo ou precisa de volumes/etiquetas (UDO/UDT/UDF)? | Dono do processo + consultor SAP | Etapa 6 (packing) |

Registrar resolução, data, autor e evidência, como em `06`.
