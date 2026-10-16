# C04 — Referência de picking SAP

Estado: material de apoio à descoberta. Não substitui o gate de contrato de [02](../02-service-layer.md) nem o `$metadata` real. Fontes em [C05](C05-fontes-e-evidencias.md).

## Papéis e processo nativo (Referência)

O Pick and Pack do SAP B1 cobre a criação da lista a partir de pedidos de venda e reservas de A/R, a separação e a criação de entrega e fatura (C5). Papéis descritos: despachante (cria e prioriza listas, compara quantidade liberada com disponível, atribui separador e acompanha no Pick and Pack Manager), separador (retira itens e marca quantidades) e quem monta os volumes e emite entrega e packing slip (C5). Equivalem aos papéis propostos em [04](../04-seguranca-e-concorrencia.md): Liberador, Separador, Confirmador.

O Pick and Pack Manager exibe pedidos de venda, reservas de A/R, ordens de produção, solicitações de transferência e listas, por status (Open, Released, Picked), em visão detalhada ou resumida (C6, C7). Quantidades da lista são salvas na unidade de medida de inventário (C7). A alocação automática de bin segue um critério configurável (ordem do código do bin, ordem do código alternativo, quantidade decrescente ou crescente) (C7).

## Modelo de dados (Referência — SDK Help 10.0)

Cabeçalho (tabela `OPKL`, C1) e linhas (`PKL1`, espelhada pelo histórico `AKL1`, C2). A coluna de Service Layer é o nome do campo observado em `02`; **validar no `$metadata`**.

| Campo de banco | Service Layer | Observação |
|---|---|---|
| AbsEntry | `Absoluteentry` (cabeçalho), `AbsoluteEntry` (linhas) | Número da lista; preservar capitalização (`02`) |
| Name | `Name` | Nome do responsável pela separação |
| OwnerCode / OwnerName | `OwnerCode` / `OwnerName` | OwnerCode relaciona com usuários |
| PickDate | `PickDate` | Data do picking |
| Remarks | `Remarks` | Observações |
| Status | `Status` | Ver abaixo |
| ObjType | `ObjectType` | `156` |
| PickEntry | `LineNumber` | Número da linha na lista |
| OrderEntry | `OrderEntry` | DocEntry do documento-base |
| OrderLine | `OrderRowID` | LineNum no documento-base |
| PickQtty | `PickedQuantity` | Quantidade separada |
| RelQtty | `ReleasedQuantity` | Quantidade liberada |
| PrevReleas | `PreviouslyReleasedQuantity` | Liberada em liberações anteriores |
| BaseObject | `BaseObjectType` | Observados `17` (pedido) e `13` (reserva de A/R) em C2 |
| (coleções) | `BatchNumbers`, `SerialNumbers`, `DocumentLinesBinAllocations` | Alocações por lote, série e bin |

Não consumir as tabelas de histórico (`APKL`, `AKL1`) na aplicação. Não escrever em tabelas SAP (AGENTS).

## Estados (Referência de banco; enums de Service Layer a validar)

| Código no banco | Significado em C1 | Nome no Service Layer |
|---|---|---|
| R | Released (liberada) | `ps_Released` observado em relato (C8) |
| Y | Picked (separada) | A validar |
| P | Partially Picked | A validar |
| D | Partially Delivered | A validar |
| C | Closed | A validar |

O SDK indica que o enum `BoPickStatus` possui cinco valores (C16). Os nomes `ps_*` além de `ps_Released` não foram verificados. Mapear estados em **um único módulo** do adaptador e validar contra o `$metadata`.

Estes estados SAP são distintos dos estados locais de [01](../01-operacao-e-dominio.md); não escrever estado SAP para forçar transição local.

## Conferência com `docs/02`

| Tema | Complemento diz | `docs/02` prevalece em |
|---|---|---|
| Fechamento | Fechar não é desfazer; lista fechada pode bloquear nova liberação em casos relatados (C14) | `POST /PickListsService_Close`; T25 |
| Atualização de alocação e quantidade | Relatos mostram `PickedQuantity` dentro de `UpdateReleasedAllocation` (C8, C9). Isso não prova a semântica de confirmação | Confirmação por contrato homologado (candidato PUT de linhas); D09, T12 |
| `GetReleasedAllocation` | Relato de uso como POST (C9) | Confirmado como POST em `02` |
| Cabeçalho `PATCH` | Não depender de PATCH em linhas | `PATCH` só para atualizações limitadas; `PUT` para linhas |
| Transporte | Sessão de 30 minutos por padrão, paginação de 20 registros e `Prefer: odata.maxpagesize` descritos em guia de terceiros (C15) | Política real de sessão e limites a confirmar no ambiente; `02` sobre OData V4 |

## Relatos de risco (usar para alimentar casos de teste)

Nenhum relato é regra. Cada um deve virar caso de teste na homologação.

| Relato | Fonte | Relaciona-se com |
|---|---|---|
| Atualização de alocação com bin retornou erro de alocação incompleta nos bins do depósito (código -5002). Resposta da comunidade: informar o número da linha-base em cada alocação de bin e converter a quantidade para a unidade de inventário quando a unidade de venda for maior que 1 | C9 | T14, T13 |
| Omitir a coleção de alocação de bin permitiu atualizar, mas perdeu a localização | C9 | T13 |
| Lote/série em lista baseada em pedido: erro de seleção incompleta (código -4014) quando o documento-base não tinha o lote atribuído. Relatado na DI API; o comportamento no Service Layer não foi confirmado | C10, C17 | T14 |
| Uma linha de pedido dividida em duas listas e entregue junta grava apenas uma lista na linha de entrega | C11 | T08, Etapa 6 |
| `PickRemark` não gravado na criação da entrega via Service Layer | C12 | Etapa 6 |
| Documento-base encerrado após a primeira entrega impediu a entrega do saldo em pedidos originados em cotação | C13 | Etapa 6 |
| Lista fechada antes da separação impediu nova lista para a mesma ordem de produção em um patch específico | C14 | T25 |
