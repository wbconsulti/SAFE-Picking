# 07 — Fontes e limites de evidência

Consulta documental: 2026-10-16. Fontes oficiais SAP; não houve acesso ao SAP do cliente. URLs de documentação podem evoluir sem corresponder ao patch instalado.

| ID | Fonte | Aplicação e limite |
|---|---|---|
| S1 | [Service Layer API Reference](https://help.sap.com/doc/056f69366b5345a386bb8149f1700c19/10.0/en-US/Service%20Layer%20API%20Reference.html) | Seções PickLists e PickListsService: métodos e ressalva sobre PUT. Não certifica payload do cliente. |
| S2 | [About SAP Business One Service Layer](https://help.sap.com/docs/SAP_BUSINESS_ONE/f110a154dd0f4c20bf7f3ebca9eeb794/60c7a0b745bd486589f05a1da77041f3.html?locale=en-US&state=PRODUCTION&version=10.0) | OData V4 corresponde a `/b1s/v2/$metadata`. |
| S3 | [Creating a Pick List](https://help.sap.com/docs/SAP_BUSINESS_ONE/68a2e87fb29941b5bf959a184d9c6727/cc3c8dadd5544f2c9aaae86a48fe8019.html) | Processo de liberação, documentos e cálculo específico de disponível para liberar. |
| S4 | [Assigning Batches or Serials in Pick and Pack Process](https://help.sap.com/docs/SAP_BUSINESS_ONE/68a2e87fb29941b5bf959a184d9c6727/c7b899a02d364f95859e8856bdac20f8.html?locale=en-US) | Alocação de lotes/seriais ao longo do processo. |
| S5 | [Batch Quantities Window](https://help.sap.com/docs/sap_business_one/68a2e87fb29941b5bf959a184d9c6727/4522b59673c80108e10000000a114a6b.html) | Distingue existente, alocado e disponível por lote/depósito. Não fornece por si só endpoint de consulta. |
| S6 | [Allocating Serial or Batch Items from Bin Locations](https://help.sap.com/docs/SAP_BUSINESS_ONE/68a2e87fb29941b5bf959a184d9c6727/0dce95a06aca403aa75d2f8b40455a0f.html?locale=en-us) | Lote pode ocupar vários bins e exigir seleção adicional de localização. |
| S7 | [Available-to-Promise Report](https://help.sap.com/docs/SAP_BUSINESS_ONE_WEB_CLIENT/2554bf7e9aa347729b0547a737e123ac/4659f0938d1a4eb592e2634a93b77d94.html) | Indicador de disponibilidade considera estoque, comprometido e encomendado; não substitui elegibilidade para picking. |
| S8 | [Working with SAP Business One Service Layer](https://help.sap.com/doc/fc2f5477516c404c8bf9ad1315a17238/10.0/en-US/Working_with_SAP_Business_One_Service_Layer.pdf) | Seções Service Layer ETag e ETag Metadata: suporte por entidade; não presumir PickLists/ações. |
| S9 | [Glossary for SAP Business One 10.0](https://help.sap.com/doc/8e194caacc284f8b943a388f05446d2f/10.0/en-US/B1Glossary.pdf) | Referência terminológica fornecida na conversa original; não contrato de integração. |
| S10 | [Página SAP fornecida originalmente](https://help.sap.com/docs/SAP_BUSINESS_ONE/68a2e87fb29941b5bf959a184d9c6727/4523653f9e152b31e10000000a1553f7.html) | Mantida para proveniência; não retornou texto verificável nesta consulta, portanto não sustenta novas afirmações. |

## Diferenças e cautelas

A descrição genérica de PickLists em S1 menciona pedidos de venda aprovados, enquanto a documentação funcional S3/S4 inclui outras origens. Essa diferença exige homologação por documento; não autoriza afirmar suporte irrestrito na Service Layer.

Os estados locais, papéis, arquitetura, idempotência, roadmap e gates deste pacote são requisitos/propostas SAFE, não funcionalidades garantidas pelo SAP. `UpdateReleasedAllocation` e `GetReleasedAllocation` não devem ser confundidos com confirmação física, e nenhuma regra de banco do cliente foi inferida de exemplos públicos.

## Registro de homologação a preencher

Para cada capacidade: versão/empresa sanitizada, hash metadata, documento/item/depósito suportados, contrato, teste, data, responsável, resultado e limitações. Estado inicial de todas: **não homologada**. Não substituir esse registro por uma captura de HTTP 200.
