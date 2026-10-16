# 06 — Roadmap e decisões pendentes

Cada etapa é uma fatia revisável. Não avançar escrita SAP apenas porque a UI ou mocks estão prontos.

| Etapa | Entrega | Saída verificável |
|---|---|---|
| 0 — Descoberta | Inventário do ambiente, metadata, fontes de saldo, políticas e matriz de capacidades | Contratos mínimos de leitura/escrita e decisões bloqueantes resolvidos com evidências |
| 1 — Fundação | Stack em ADR, monólito modular, identidade e adaptador somente leitura | Isolamento por empresa/permissões e testes de contrato de consulta |
| 2 — Planejamento manual | Demanda, origens e rascunho por depósito/lote/quantidade | Fluxo navegável com dados fictícios; zero escrita SAP |
| 3 — Liberação | Comando persistido, criação/alocação, auditoria e reconciliação | Lista/alocação verificadas; falha parcial e concorrência testadas |
| 4 — Confirmação | Captura física e atualização parcial/total | Releitura comprova quantidade real, sem documento logístico automático |
| 5 — Piloto | Observabilidade, operação, treinamento, backup e restauração | Aceite do cliente em escopo controlado, sem pendências críticas |
| 6 — Expansões | Bins, seriais, outros documentos, packing/entrega, sugestões | Um contrato e conjunto de testes por capacidade; aprovação de novo escopo |

## Registro inicial de decisões

| ID | Pergunta | Responsável esperado | Bloqueia |
|---|---|---|---|
| D01 | Versão/FP/patch, banco, localização e Service Layer disponíveis? | Administrador SAP | Integração |
| D02 | Qual documento inicia o piloto e quais linhas são elegíveis? | Dono do processo + consultor SAP | Liberação |
| D03 | Depósitos têm bins? Itens usam lote/serial? Qual unidade/precisão? | Estoque + consultor SAP | Escopo MVP |
| D04 | Fonte suportada para saldo e alocação por lote/depósito, incluindo parcela própria? | Consultor SAP + integração | Decisão de disponibilidade |
| D05 | Quais políticas para validade, bloqueios, saldo negativo e exceções? | Dono do processo | Liberação/confirmação |
| D06 | Pode mudar depósito-base? Qual fluxo autorizado fora do picking? | Consultor SAP | Seleção alternativa |
| D07 | Papéis, segregação, conta SAP e isolamento por empresa? | Segurança + cliente | Escrita |
| D08 | Identificador remoto confiável para reconciliar criação incerta? UDF é permitido? | Consultor SAP | Criação segura |
| D09 | PUT, coleções, unidades, quantidades cumulativas e concorrência reais? | Integração + QA | Confirmação |
| D10 | Política de parcial, realocação, cancelamento e fechamento? | Dono do processo + SAP | Fluxos correspondentes |
| D11 | Stack, hospedagem, identidade, banco, volume e SLA? | Responsável técnico | Fundação/produção |
| D12 | Retenção, backup, auditoria e responsável por pendências? | Operação + segurança | Produção |
| D13 | Versão instalada do DeepSeek Harness e modelo/quota disponíveis? | Responsável pelo Harness | Execução assistida |

Todas estão pendentes nesta entrega; não exigem parar documentação ou protótipo fictício, mas bloqueiam as operações dependentes. Registrar resolução, data, autor e evidência. Não pedir credenciais em respostas a essas perguntas.

## Fora do primeiro ciclo

Escolha automática de lotes/FEFO/FIFO, estoque paralelo, substituição silenciosa, baixa direta em SQL, entrega/faturamento automático, execução offline com promessa de reserva, suporte universal a todos os documentos e migração de processo sem aceite. Propostas futuras devem preservar as invariantes.
