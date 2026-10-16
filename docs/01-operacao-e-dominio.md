# 01 — Operação e domínio

## Escopo

SAFE Picking apoia o operador que consulta disponibilidades e decide manualmente o que separar. Não otimiza nem escolhe origens automaticamente. O MVP proposto começa por pedidos de venda elegíveis e itens por lote, em depósitos sem bins; essa escolha depende de confirmação do cliente. Se a operação real exige bins, o suporte correspondente vira pré-requisito, não uma validação a remover.

Outros documentos (fatura de reserva, componentes de ordem de produção, solicitação de transferência) fazem parte da descoberta, mas só são habilitados individualmente após homologação. A documentação funcional SAP contempla essas origens [S3/S4](07-fontes-e-evidencias.md); isso não comprova suporte idêntico de cada contrato Service Layer no ambiente.

## Identidade e regras obrigatórias

| ID | Regra |
|---|---|
| R01 | Identificar demanda por empresa + tipo de objeto + DocEntry + linha-base; DocNum é rótulo de exibição. ItemCode não identifica sozinho uma linha. |
| R02 | Identificar oferta por empresa + item + depósito + identidade de lote SAP; conservar número visível do lote separadamente. Incluir bin quando exigido. |
| R03 | Quantidade positiva, decimal exata, precisão e conversão de unidade homologadas. Não usar ponto flutuante binário para decisão de saldo. |
| R04 | Soma das origens de uma linha não pode ultrapassar demanda elegível restante nem capacidade liberável/alocável validada. Não reutilizar compromisso já contabilizado. |
| R05 | Não mudar implicitamente depósito da linha-base. Outro depósito só entra após fluxo SAP permitido e autorizado; pode exigir ajuste do documento ou transferência fora do MVP. |
| R06 | Lote deve pertencer ao item e ter oferta elegível no depósito; verificar status, restrições de uso, validade e políticas do cliente. Não inventar bloqueio universal por validade nem ignorar bloqueio configurado. |
| R07 | Revalidar documento aberto/aprovado/elegível, linha, unidade, estoque e alocações antes de toda escrita. Mudança externa invalida aprovação baseada em snapshot antigo. |
| R08 | Nenhuma etapa local concede reserva SAP. Exibir estado local e estado SAP separadamente. |
| R09 | Divergência física exige registro e decisão explícita. Nunca substituir origem ou completar quantidade silenciosamente. |
| R10 | Não criar entrega, saída, transferência, faturamento ou encerramento automático ao confirmar picking. |

## Disponibilidade e compromissos

Exibir medidas com unidade, granularidade, fonte e instante da consulta. Nunca chamar todos os números simplesmente de “disponível”.

| Medida | Interpretação operacional |
|---|---|
| Estoque físico por item/depósito | Existência física informada pelo SAP; não implica liberdade para usar. |
| Comprometido por item/depósito | Demanda comprometida segundo SAP; não presume vinculação exata a lote. |
| Encomendado/entrada prevista | Oferta futura; não substitui estoque físico para separar agora. |
| Disponível para liberar | Capacidade do processo de Pick and Pack, sensível a liberações e separações existentes. |
| Existente/alocado/disponível do lote | Medidas na combinação lote/depósito; precisam de fonte homologada. |
| Planejado localmente | Intenção SAFE; deve ser identificada como não reservada no SAP. |

O ATP considera estoque, compromissos e entradas previstas [S7]. Já o cálculo de liberação documentado considera estoque e quantidades liberadas, separadas e propostas para liberar [S3]. Esses indicadores respondem a perguntas diferentes. É proibido promover `OnHand - Committed` ou `OnHand + Ordered - Committed` a fórmula universal de picking ou subtrair novamente alocações já refletidas no valor consultado.

O cadastro de lotes não basta como consulta de saldo. A descoberta deve mapear cada medida à fonte SAP suportada, regras de agregação e comparação com a tela SAP. Se não houver fonte confiável para lote/depósito, apresentar “não verificado” e impedir liberação, em vez de distribuir saldo agregado entre lotes.

Ao editar uma alocação existente, distinguir a parcela própria da parcela de terceiros. O limite adicional pode depender do que já está alocado à própria lista. Não liberar primeiro e reservar depois como estratégia automática: isso expõe a oferta à concorrência e pode perder a posição original.

## Fluxo manual e telas

1. **Selecionar demanda:** filtrar documentos elegíveis, selecionar linhas e mostrar quantidade original, atendida, pendente, liberada/separada quando disponível, unidade e depósito da linha. Identificar restrições e origem dos dados.
2. **Consultar origens:** apresentar apenas depósitos elegíveis; listar lotes com medidas distintas, validade/status e instante da consulta. Mostrar motivos para opções indisponíveis.
3. **Planejar:** o operador informa quantidade por lote. Permitir vários lotes na mesma linha quando autorizado. Agrupar duplicatas da mesma identidade sem perder rastreabilidade; nunca confundir linhas do mesmo item.
4. **Revisar:** mostrar demanda, origens e totais, pendências e diferença desde a consulta anterior. Botão “Salvar planejamento” não chama liberação SAP.
5. **Liberar:** usuário autorizado confirma intenção; servidor revalida, cria/atualiza Pick List e alocação pelo contrato homologado. Exibir número SAP somente quando conhecido. Sucesso exige releitura coerente.
6. **Separar:** operador trabalha sobre alocação liberada verificada, registra quantidade realmente separada e ocorrências. Leitura de código de barras apenas identifica candidato, sem confirmar por si só.
7. **Confirmar:** revisar quantidade real por lote/bin, revalidar e transmitir pelo contrato de confirmação homologado. Mostrar confirmação SAP após releitura. Parcial permanece parcial; excedente exige revisão anterior da liberação.
8. **Consultar histórico:** distinguir ações, tentativas, resultados, pendências e documento SAP vinculado. Documento logístico posterior é outro processo.

Interface responsiva para desktop/tablet/coletor, com foco visível, teclado, rótulos acessíveis, estado não dependente apenas de cor e confirmação clara por ação. Falha de rede preserva rascunho, mas impede informar sucesso SAP. Não prometer operação offline com reserva.

## Estados locais propostos

| Estado | Transição e condição |
|---|---|
| Rascunho | Editável; salvar apenas intenção local. |
| Pronto para liberar | Validações locais completas; ainda exige revalidação SAP ao executar. |
| Em liberação | Comando persistido e em execução; bloquear envio duplicado. |
| Liberação incompleta | Lista criada mas alocação falhou; reconciliar a mesma lista, sem recriar. |
| Liberado verificado | Lista e alocações relidas e compatíveis com intenção aprovada. |
| Em separação | Captura física local; SAP ainda pode refletir estado anterior. |
| Em confirmação | Comando persistido; aguardar retorno/reconciliação. |
| Parcial confirmado / Confirmado | Quantidades reais confirmadas por releitura; não equivalem a entregue. |
| Resultado desconhecido | Falha ambígua; impedir repetição até reconciliar. |
| Divergente | Estado remoto diferente; exigir atualização e decisão explícita. |

Descartar rascunho é local. Desfazer liberação, cancelar ou fechar lista exige regra SAP específica, permissão e homologação; não usar DELETE, fechamento ou quantidade zero como cancelamento genérico. Estados SAP não devem ser escritos para forçar uma transição local.

## Exemplo fictício

Pedido `DocEntry=100`, linha `0`, item `ITEM-DEMO`, depósito elegível `D01`, demanda 10 UN. Operador escolhe 6 UN do lote `L-A` e 4 UN do lote `L-B`. Um saldo em `D02` não autoriza deslocar a origem. Outro usuário consome parte de `L-B` antes da liberação: SAFE interrompe, apresenta a nova situação e solicita revisão; não escolhe `L-C` automaticamente. Se a separação física resultar em 6 + 3, registrar parcial e destino da pendência conforme regra homologada, sem marcar 10 como separado.
