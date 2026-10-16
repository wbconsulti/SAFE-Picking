# Prompt inicial — SAFE Picking

Você trabalhará no repositório `wbconsulti/SAFE-Picking`. Comece confirmando diretório, branch, estado Git e instruções AGENTS.md. Leia docs/README.md, docs/01 a docs/07, ADRs e harness/README.md. Não trate a documentação como prova de software existente ou de integração homologada.

O cliente realiza picking manual: escolhe documento/linha, depósito elegível, lote e quantidade, observando disponibilidades. Preserve essa decisão. Não implemente seleção automática, estoque paralelo, alteração implícita de depósito, SQL de escrita no SAP ou entrega automática.

Separe planejamento local, liberação/alocação SAP e confirmação da quantidade fisicamente separada. SAP B1 é a autoridade. Identifique demanda pela empresa/tipo/DocEntry/linha e oferta por empresa/item/depósito/lote, incluindo bin quando exigido. Respeite unidades, bloqueios, compromissos, alocações e regras do documento.

Use Service Layer OData V4 em /b1s/v2. Confira o metadata real e testes controlados antes de produzir payloads finais. Distinga POST PickLists, POST PickLists(id)/GetReleasedAllocation, POST PickListsService_UpdateReleasedAllocation e a atualização de linhas via contrato PUT homologado. Não invente PATCH de confirmação nem ação vinculada de UpdateReleasedAllocation.

Nesta primeira rodada, faça inventário e análise da etapa 0, sem modificar código funcional ou escrever no SAP. Entregue:

1. Resumo das invariantes e mapa dos arquivos existentes.
2. Matriz de capacidades conhecidas, hipóteses e pendências D01–D13.
3. Contratos e fontes que precisam de validação, com critérios de evidência.
4. Proposta de primeira fatia, arquivos, testes e critérios de aceite.
5. Decisões realmente bloqueantes e trabalho independente possível.

Não peça senhas/chaves em chat. Não instale infraestrutura, publique, faça merge ou opere produção sem autorização específica. Nas rodadas posteriores autorizadas, implemente apenas a fatia acordada, preserve conteúdo existente, teste concorrência/idempotência/falhas e reporte evidências reais usando MODELO_ENTREGA.md. Diga claramente quando algo só foi testado com mocks.
