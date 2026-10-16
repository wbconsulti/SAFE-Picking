# C05 — Fontes e evidências do complemento

Consulta documental: 2026-10-16. Não houve acesso ao SAP do cliente. Complementa [07](../07-fontes-e-evidencias.md); os IDs `S1`–`S10` pertencem àquele documento e `C1`–`C17` a este.

Tipos: **Oficial** (documentação SAP) e **Relato** (comunidade ou terceiros). Páginas de versões antigas (9.x) podem não refletir o patch instalado.

| ID | Tipo | Fonte | Aplicação e limite |
|---|---|---|---|
| C1 | Oficial | [SDK Help 10.0 — tabela OPKL](https://help.sap.com/doc/089315d8d0f8475a9fc84fb919b501a3/10.0/en-US/SDKHelp/OPKL.html) | Campos e estados do cabeçalho. Não é contrato do Service Layer |
| C2 | Oficial | [SDK Help 10.0 — tabela AKL1](https://help.sap.com/doc/089315d8d0f8475a9fc84fb919b501a3/10.0/en-US/SDKHelp/AKL1.html) | Linhas (histórico da `PKL1`): campos, estados e tipos de documento-base |
| C3 | Oficial | [SDK Help 10.0 — membros de PickLists_Lines](https://help.sap.com/doc/089315d8d0f8475a9fc84fb919b501a3/10.0/en-US/SDKHelp/SAPbobsCOM~PickLists_Lines_members.html) | Correspondência campo ↔ propriedade na DI API |
| C4 | Oficial | [SDK Help 10.0 — membros de PickLists](https://help.sap.com/doc/089315d8d0f8475a9fc84fb919b501a3/10.0/en-US/SDKHelp/SAPbobsCOM~PickLists_members.html) | Métodos e propriedades do objeto na DI API. Não prova nomes no Service Layer |
| C5 | Oficial | [Help Portal 9.0 — Pick and Pack](https://help.sap.com/doc/saphelp_sbo900/9.0/en-US/45/23653f9e152b31e10000000a1553f7/content.htm) | Processo, papéis, entrega e fatura como etapas posteriores |
| C6 | Oficial | [Help Portal 9.1 — Pick and Pack Manager, critérios](https://help.sap.com/saphelp_sbo91/helpdata/en/45/2365be9e152b31e10000000a1553f7/content.htm) | Origens e status exibidos pelo Manager. Versão antiga |
| C7 | Oficial | [Help Portal 9.2 — gaveta Released e alocação de bin](https://help.sap.com/saphelp_sbo92/helpdata/en/45/2365ff9e152b31e10000000a1553f7/content.htm) | Quantidades em UoM de inventário; critérios de alocação automática de bin. Versão antiga |
| C8 | Relato | [SAP Community — PickList via Service Layer](https://answers.sap.com/questions/578617/picklist-difference-with-b1-client-with-service-la.html) | Forma de resposta e uso de `UpdateReleasedAllocation`. 2018 |
| C9 | Relato | [SAP Community — alocação de bin em PickLists](https://answers.sap.com/questions/582121/picklists-bin-allocation-problem.html) | Erro -5002, `BaseLineNumber`, unidade de venda. 2018 |
| C10 | Relato | [SAP Community — DIAPI lote e bin em lista](https://answers.sap.com/questions/364962/diapi-allocation-of-batch-and-bin-location-in-pick.html) | Erro -4014 na DI API, não no Service Layer |
| C11 | Relato | [SAP Community — vínculo lista ↔ entrega](https://groups.community.sap.com/t5/enterprise-resource-planning-q-a/linking-pick-list-and-delivery-note/qaq-p/12590782) | Campo de vínculo guarda uma única lista. 2022 |
| C12 | Relato | [SAP Community — PickRemark na entrega](https://answers.sap.com/questions/12657179/sap-b1-93-hana-pl08-service-layer-create-delivery.html) | Campo não gravado na criação. 2019, 2022 |
| C13 | Relato | [SAP Community — documento-base encerrado](https://answers.sap.com/questions/12082537/delivery-note-error---one-of-the-base-documents-ha.html) | Entrega do saldo de pedido originado em cotação |
| C14 | Relato | [SAP Community — lista fechada antes da separação](https://answers.sap.com/questions/13456492/pick-list-closed-before-pick-cant-create-new-pick.html) | Bloqueio de nova lista em ordem de produção, B1 10.0 PL02 |
| C15 | Relato | [Apideck — guia da Service Layer](https://dev.to/apideck/sap-business-one-api-integration-guide-authentication-endpoints-and-odata-queries-ean) | Sessão, paginação e `$batch`; guia de terceiros |
| C16 | Oficial | [SDK Help 10.0 — PickStatus](https://help.sap.com/doc/089315d8d0f8475a9fc84fb919b501a3/10.0/en-US/SDKHelp/SAPbobsCOM~Document_Lines~PickStatus.html) | `BoPickStatus` com cinco valores |
| C17 | Relato | [SAP Community — lote/série e bin em lista](https://answers.sap.com/questions/665338/sap-93-pl-06---update-packing-list-with-batchseria.html) | Atribuir lote ao documento-base antes da lista. DI API |

## Limites

- S9 e S10 (material originalmente fornecido) bloqueiam leitura automatizada ou não retornaram texto verificável. O conteúdo funcional de C1 a C7 corrobora o processo, mas não os substitui. Recomenda-se guardar o glossário localmente para consulta humana.
- Relatos (C8 a C15, C17) descrevem versões e patches específicos, em geral anteriores ao OData V4. Servem para antecipar riscos e escrever casos de teste. Nenhum vale como contrato nem como homologação.
- Os nomes de campos de Service Layer nas tabelas deste complemento vêm de `02` e de C8/C9. Validar no `$metadata` real antes de qualquer uso.
- Todas as capacidades têm estado inicial **não homologada**.
