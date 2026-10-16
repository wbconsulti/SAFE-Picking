# 04 — Segurança, concorrência e rastreabilidade

## Permissões propostas

| Papel | Ações |
|---|---|
| Consulta | Ver documentos/origens no próprio escopo |
| Planejador | Criar e revisar planos autorizados |
| Liberador | Liberar e alterar alocação dentro das regras homologadas |
| Separador | Capturar quantidades físicas das listas atribuídas |
| Confirmador | Confirmar separação e registrar divergência |
| Auditor | Consultar histórico autorizado, sem alterar evidências |
| Administrador | Configurar acesso e capacidades; não ignora regra SAP |

Papéis podem ser combinados por política aprovada; separação de função é decisão do cliente. Validar autorização por operação, empresa, depósito, documento e atribuição no servidor. Permissão local não amplia a permissão da conta SAP. Se houver conta técnica compartilhada, registrar o ator humano e mapear precisamente o escopo permitido; evitar conta administrativa genérica.

## Controles

- Credenciais em cofre/configuração protegida fora do Git e dos prompts; rotação, acesso mínimo e segregação de homologação/produção. Fixtures sempre fictícias.
- TLS validado; SAP acessível somente pelo backend autorizado. Não desabilitar certificados para “resolver” conexão.
- Sessão web protegida, cookies HttpOnly/Secure e política SameSite adequada; proteção CSRF para autenticação por cookie; CORS restrito. Não guardar credenciais SAP em localStorage.
- Validar payload, tamanhos, quantidade de linhas, números decimais, IDs e filtros. Aplicar limites de taxa, tempo e concorrência.
- Verificar autorização no acesso direto por ID, não apenas na listagem; impedir troca de empresa por parâmetro e acesso a comandos de outro usuário/escopo.
- Redigir segredos, cookies, dados pessoais e detalhes comerciais em logs, exceções, telemetria e evidências. Definir retenção, acesso e descarte com responsável antes de produção.
- Backup/restauração do banco local deve preservar vínculos e comandos pendentes; após restauração reconciliar antes de voltar a escrever no SAP.

## Concorrência: duas autoridades diferentes

O controle de revisão local evita dois operadores sobrescreverem o mesmo plano. Um lock distribuído por lista/linha ou recurso pode coordenar instâncias SAFE, mas não bloqueia o cliente SAP nem outras integrações. Mesmo com serialização local, reler e aceitar que o SAP ainda pode rejeitar a operação.

ETag/If-Match só pode ser usado se a entidade e operação efetivamente suportarem e tiverem teste comprovando rejeição de versão antiga. O guia SAP descreve suporte seletivo [S8](07-fontes-e-evidencias.md); não presumir suporte em PickLists ou nas ações. Se inexistente, documentar risco residual, reduzir janela de leitura/escrita, serializar comandos SAFE e verificar resultado. Não afirmar prevenção total de corrida por releitura.

## Protocolo de comando e idempotência

1. Autorizar e validar revisão esperada; conflito local não escreve no SAP.
2. Persistir chave de idempotência + empresa + tipo de operação + hash canônico do conteúdo e intenção. Mesma chave/conteúdo retorna resultado existente; mesma chave/conteúdo diferente é conflito.
3. Adquirir coordenação local/distribuída necessária; reler SAP e validar novamente.
4. Marcar tentativa em andamento antes de enviar. Persistir identidade SAP retornada assim que disponível.
5. Após resposta, reler o objeto e comparar efeito pretendido; registrar resultado e liberar coordenação.
6. Timeout, conexão perdida, falha de persistência após retorno ou reinício durante envio tornam o resultado desconhecido. Não assumir rollback.

A chave local não torna o endpoint SAP idempotente. Para criação, a descoberta deve definir vínculo remoto pesquisável e confiável (por exemplo, UDF aprovado e suportado, se disponível) ou procedimento de reconciliação assistida. Sem identificação única, não reenviar criação incerta. Observação livre ou combinação de data/item/quantidade não prova unicidade. Múltiplos candidatos exigem intervenção.

## Falhas e recuperação

| Situação | Conduta |
|---|---|
| Validação/autorização falha antes de enviar | Registrar rejeição; permitir correção sem efeito remoto |
| Sessão expirada em consulta | Renovação controlada e repetição limitada de leitura |
| Timeout/5xx durante escrita | Resultado desconhecido; reconciliar antes de decidir nova tentativa |
| Pick List criada, alocação rejeitada | Manter ID e estado de liberação incompleta; corrigir mesma lista com autorização |
| SAP muda após planejamento | Invalidar revisão operacional e exibir diferenças |
| Resposta recebida, releitura falha | Não declarar confirmado; continuar reconciliação |
| Efeito remoto diferente do esperado | Estado divergente; preservar evidências e solicitar decisão |
| Confirmação parcial | Registrar apenas efeito comprovado; pendência permanece visível |

Retries automáticos somente para leituras seguras, com limite/backoff. Ações POST de leitura precisam ser classificadas explicitamente pelo contrato. Não realizar compensação destrutiva (fechar, cancelar, zerar, excluir) sem regra e autorização próprias. Escalonar comandos sem resolução após prazo operacional acordado; não mantê-los invisíveis em loop.

## Trilha de auditoria

Registrar autor humano, conta de integração identificável sem segredo, empresa, documento/linha, lista/linha, depósito/lote/bin, unidade, quantidades planejadas/liberadas/reais, revisão, motivo, horário UTC, correlation ID, comando, tentativa, resposta sanitizada e releitura. Registrar também rejeições e mudanças de permissão/capacidade.

Eventos append-only na aplicação, acesso restrito e mecanismo de integridade/backup definido antes de produção. Não substituir eventos antigos pelo último status. Cadeia: intenção → tentativa → objeto SAP → efeito observado → decisão de reconciliação. Acesso do auditor também é auditável.
