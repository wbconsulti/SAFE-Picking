# 05 — Testes e critérios de aceite

Estado desta entrega: plano de testes, nenhum teste funcional executado. Sem aplicação ou ambiente SAP, revisão documental não comprova funcionamento.

## Camadas

1. Unidade: identidades, totais decimais, unidades, estados locais, permissões e invariantes; sem dependência SAP.
2. Contrato: fixtures sanitizadas vinculadas à versão/hash do metadata; verificar mapeamento, campos, índices, coleções e erro desconhecido.
3. Integração controlada: empresa SAP de homologação com dados autorizados; confrontar resultado pela leitura Service Layer e pelo cliente SAP.
4. Ponta a ponta: operador planeja, libera, separa e confirma; incluir teclado/tablet, erro/reconexão, dupla submissão e auditoria.
5. Resiliência/segurança: concorrência entre instâncias e cliente SAP, falha após commit remoto, reinício, autorização cruzada e redação de segredos.

## Matriz mínima

| Caso | Regra/área | Cenário e resultado exigido |
|---|---|---|
| T01 | R01/R02 | Mesmo item em duas linhas e mesmo número de lote em contextos distintos: identidades não se misturam. |
| T02 | R03/R04 | Zero, negativo, precisão inválida e soma excedente: rejeitar antes do SAP; conversão válida conserva totais. |
| T03 | R05 | Selecionar depósito incompatível, ainda que com saldo: bloquear sem alterar documento-base. |
| T04 | R06 | Lote inexistente, de outro item/depósito, bloqueado ou sujeito a política de validade: aplicar regra comprovada. |
| T05 | R07 | Documento fechado/cancelado/não elegível ou quantidade alterada entre consulta e envio: impedir escrita obsoleta. |
| T06 | R08 | Salvar plano: nenhuma criação/liberação SAP; interface mostra intenção não reservada. |
| T07 | Disponibilidade | Compromisso agregado, alocação própria/de terceiros e entrada futura: sem dupla subtração e sem oferta fictícia. |
| T08 | Lotes | Uma linha dividida em dois lotes: quantidade e vínculos corretos na releitura SAP. |
| T09 | Liberação | Criar lista e alocar com sucesso: só então liberar execução; falha na segunda chamada mantém ID e estado incompleto. |
| T10 | Confirmação/R09 | Parcial e total, falta física e excesso: sem inflar quantidade nem substituir lote; parcial permanece pendente. |
| T11 | R10 | Confirmar picking: nenhum documento de entrega/saída/fatura/transferência criado automaticamente. |
| T12 | Contrato | GetReleasedAllocation via POST e UpdateReleasedAllocation via serviço: contrato real confere com metadata e teste. |
| T13 | Coleções | PUT preserva linhas/alocações não alteradas; omissão/array vazio e referências de lote/bin testados. |
| T14 | Bins/seriais | Configuração não suportada bloqueia; quando habilitada, validar todos os vínculos e cardinalidade. |
| T15 | Concorrência | Dois operadores disputam mesma oferta; sem sucesso falso e sem sobrescrever alteração externa conhecida. |
| T16 | Idempotência | Duplo clique e repetição da mesma chave não duplicam comando; chave com outro conteúdo gera conflito. |
| T17 | Ambiguidade | SAP conclui e conexão cai: reconciliação encontra efeito, não recria lista nem soma quantidade novamente. |
| T18 | Reinício | Processo cai após enviar/antes de gravar retorno: recupera pendência como desconhecida e reconcilia. |
| T19 | ETag | Se suportado, versão antiga é rejeitada; se não, capacidade desabilitada e risco residual documentado. |
| T20 | Segurança | Usuário muda empresa/depósito/ID no request: acesso negado, inclusive leitura de comando e auditoria. |
| T21 | Segredos | Logs, respostas, fixtures e frontend não contêm credenciais/cookies SAP. |
| T22 | Transporte | Expiração de sessão, paginação, timeout e rate limit: sem truncar oferta nem repetir escrita ambígua. |
| T23 | Auditoria | Reconstruir plano → comando → tentativa → SAP → confirmação, incluindo rejeição e motivo. |
| T24 | Alteração | Realoque apenas quando SAP permitir; não liberar reserva própria antes de garantir fluxo homologado. |
| T25 | Fechamento | Cancelar/fechar fora de contrato é bloqueado; não interpretar Close como desfazer universal. |
| T26 | UX | Navegação por teclado, foco/labels, números locais e rede caída não induzem confirmação incorreta. |
| T27 | Capacidade | Metadata/versão muda: operação afetada fica bloqueada até revalidação. |

## Evidência por execução

Guardar: caso/requisito, commit, data, executor, ambiente sanitizado, versão SAP, hash metadata, precondições, entradas fictícias ou autorizadas, resultado esperado/observado, IDs protegidos de correlação, consultas de verificação e limitações. Usar [modelo](../harness/MODELO_ENTREGA.md).

Não executar escrita em produção para “testar”. Prever limpeza autorizada de dados de homologação; não presumir que registros SAP podem ser excluídos. Falhas devem gerar correção e repetição dos casos afetados, mantendo evidência anterior.

## Gates de aceite

- Nenhuma escrita sem matriz de capacidades e contrato aprovado para a combinação real.
- Nenhuma liberação sem T03–T09, T12–T18 e T20–T23 aplicáveis aprovados.
- Nenhuma confirmação sem T10–T14 e regressão de concorrência/ambiguidade.
- Nenhuma produção sem backup/restauração, reconciliação operacional, monitoramento, revisão de acessos e aceite do responsável.
- Metas de desempenho, volume e retenção devem ser definidas na descoberta; medir com volume representativo e orçamento aprovado, sem inventar SLA já acordado.
