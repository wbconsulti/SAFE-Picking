# C02 — Disponibilidade e sugestão assistida

Estado: proposta. Corresponde à expansão "sugestões" da Etapa 6 de [06](../06-roadmap.md) e ao que [ADR-001](../decisions/0001-autoridade-sap-e-planejamento-manual.md) chama de "eventual sugestão futura", que exige aprovação explícita. Nenhum item abaixo deve ser implementado antes dos gates.

## Gates

O recurso permanece desabilitado até **todos** os gates estarem satisfeitos e registrados.

| Gate | Condição |
|---|---|
| G1 | Aprovação explícita do responsável e de novo escopo (DC01) |
| G2 | ADR próprio, aprovado, descrevendo o escopo da sugestão e seu efeito sobre as invariantes |
| G3 | Fonte de disponibilidade por lote/depósito **homologada** (D04), com T07 aprovado |
| G4 | Critério de sugestão definido e homologado pelo cliente, por classe de item/depósito (DC02). O sistema não tem critério padrão |
| G5 | Etapas 2 a 4 concluídas e regressão do fluxo manual aprovada |

## Consulta de origens

Detalha o contrato `ConsultarOrigens(referenciaLinha)` de [03](../03-arquitetura.md). É uma porta interna (**Hipótese**). O domínio não conhece SQL, view nem entidade SAP.

Cada origem retornada contém:

| Campo | Regra |
|---|---|
| Empresa, item, depósito | Identidade conforme R02 |
| Identidade SAP do lote e número visível | Mantidos separados (R02) |
| Bin | Quando exigido (`02`, S6) |
| Existente, alocado, disponível para liberar | Cada uma é uma **medida** distinta |
| Validade, status e bloqueios do lote | Como informados pelo SAP; sem inventar bloqueio universal (R06) |
| Motivos de indisponibilidade | Texto para o operador |

Cada **medida** carrega: valor decimal exato **ou nulo**, unidade, fonte e instante da consulta (UTC). Valor nulo significa "não verificado". Nulo não equivale a zero e impede a liberação (AGENTS; `01`).

Fonte: **nenhuma assumida**. `02` proíbe inventar entidade de saldo por lote. Possíveis caminhos, a decidir pelo consultor SAP em D04 (**Hipótese**): consulta somente de leitura, autorizada e parametrizada (por exemplo uma consulta SQL cadastrada ou uma view do cliente), ou entidade que o `$metadata` real comprove. Sem fonte homologada para lote/depósito, o sistema exibe "não verificado" e não distribui saldo agregado entre lotes (`01`).

## Motor de sugestão

Função pura, sem I/O, no domínio (**Hipótese**; linguagem por ADR).

**Entrada:** linha de demanda (R01), origens elegíveis com medidas, critério homologado.
**Saída:** proposta (origem, quantidade) por linha, `faltante`, `bloqueios` e `justificativa` legível.

Regras obrigatórias:

- Respeita R03 (decimal exato), R04 (soma não excede o pendente nem a capacidade liberável), R05 (depósito da linha-base) e R06 (lote elegível).
- Usa somente origens com todas as medidas necessárias verificadas. Medida nula exclui a origem e gera bloqueio. Nunca distribui saldo agregado.
- Se não há critério homologado, não produz proposta.
- Faltante maior que zero vira alerta e exige decisão do operador. Nunca completa quantidade com outro depósito ou lote (R05, R09).
- Determinística: mesma entrada, mesma saída. Explica o porquê de cada origem escolhida.

## Critérios candidatos (nenhum é padrão)

Só podem ser usados se o cliente homologar (DC02):

- Ordem de código do bin, ordem do código alternativo, quantidade decrescente ou crescente: são as opções que o próprio SAP B1 oferece para alocação automática de bin (**Referência**, C7). Replicá-las não é automático nem obrigatório.
- Lote por validade (FEFO) ou por entrada (FIFO): **Hipótese**, sem fonte SAP que os imponha. AGENTS veda implementá-los sem escolha explícita do usuário e regra homologada.

## Fluxo do modo S

1. O operador abre a linha. O sistema consulta origens e o motor gera a proposta no **rascunho**, marcada "Sugestão — não reservada no SAP".
2. O operador revisa por linha: edita, rejeita ou **aceita**. Aceitar é ação explícita, registrada.
3. A linha aceita passa a ser plano manual comum. Daí em diante o fluxo é idêntico ao do modo M, incluindo revalidação no comando (R07).
4. Se o snapshot mudar, a proposta é invalidada. Só nova ação do operador gera nova proposta.

A proposta nunca avança de estado sozinha, nunca libera e nunca altera depósito.

## Concorrência

Proposta não reserva (R08). Dois operadores podem receber a mesma sugestão. O SAP pode rejeitar a liberação do segundo; a tela deve explicar e pedir nova decisão. Aviso local de conflito é **Hipótese**, apenas informativo. Ver `04` sobre os limites de locks locais.

## Testes complementares

| Caso | Regra | Cenário e resultado exigido |
|---|---|---|
| TC01 | R01–R06 | Testes de propriedade: toda proposta respeita identidade, depósito da linha e elegibilidade do lote |
| TC02 | R04 | Soma proposta nunca excede o pendente da linha |
| TC03 | Medida nula | Medida desconhecida: origem excluída, "não verificado", sem proposta que dependa dela |
| TC04 | R07 | Snapshot muda entre proposta e liberação: proposta invalidada, sem recálculo silencioso |
| TC05 | AGENTS | Proposta nunca avança sozinha; aceitação é explícita e auditada |
| TC06 | Gates | Sem G1–G5 o recurso está desabilitado e inacessível |
| TC07 | Regressão | Com o modo S desligado, T06 a T11 continuam aprovados |
| TC08 | Determinismo | Mesma entrada produz a mesma proposta e a mesma justificativa |
| TC09 | T23 | A auditoria reconstrói proposta → decisão → divergência → comando |

Cada execução usa o [modelo de entrega](../../harness/MODELO_ENTREGA.md). Mocks e dados fictícios não comprovam homologação SAP (`05`).
