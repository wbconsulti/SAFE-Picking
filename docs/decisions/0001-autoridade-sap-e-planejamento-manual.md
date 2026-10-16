# ADR-001 — Autoridade SAP e planejamento manual

## Estado

Diretriz de produto solicitada pelo responsável, registrada em 2026-10-16; implementação e contratos SAP pendentes.

## Contexto

O cliente seleciona manualmente demanda, depósito, lote e quantidade. Uma interface que aloca automaticamente ou interpreta saldo agregado como lote livre altera o processo e pode violar regras SAP.

## Decisão

Manter o SAP como autoridade sobre documentos, estoque, alocações e estados. Separar planejamento local, liberação/alocação SAP e confirmação física. Guardar somente intenção, evidências e vínculos locais. Toda origem deve ser elegível para a linha-base e validada novamente no comando.

## Alternativas consideradas

- Reserva local como verdade: rejeitada, pois outros clientes SAP não a respeitam.
- Escolha automática por algoritmo: fora do requisito manual; eventual sugestão futura exige aprovação explícita.
- Confirmação que gera entrega: rejeitada no primeiro ciclo por misturar efeitos distintos.

## Consequências

Exige lidar com dados obsoletos, conflitos e falhas entre chamadas. A UI precisa explicar estado local versus SAP. Não há promessa de reserva offline. Contratos homologados e reconciliação são parte da funcionalidade, não melhorias opcionais.
