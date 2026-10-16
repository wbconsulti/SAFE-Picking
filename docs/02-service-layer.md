# 02 — Service Layer e contratos

## Referência oficial, ainda não homologada no cliente

Usar OData V4 em `/b1s/v2`; validar em `GET /b1s/v2/$metadata` [S2]. A referência [S1](07-fontes-e-evidencias.md) apresenta:

| Método/caminho relativo a `/b1s/v2` | Finalidade |
|---|---|
| `GET /PickLists` | Consultar coleção |
| `GET /PickLists({id})` | Consultar lista |
| `POST /PickLists` | Criar lista |
| `PATCH /PickLists({id})` | Atualizações limitadas, como observações |
| `PUT /PickLists({id})` | Atualização de linhas; a referência alerta contra pressupor PATCH para `PickListsLines` |
| `POST /PickLists({id})/GetReleasedAllocation` | Obter alocação liberada; apesar da leitura, o método é POST |
| `POST /PickListsService_UpdateReleasedAllocation` | Atualizar alocação liberada; parâmetro externo `PickList` |
| `POST /PickListsService_Close` | Fechamento; não é confirmação física nem cancelamento universal |

Não inventar `/PickLists({id})/UpdateReleasedAllocation`. Campos observados incluem `Absoluteentry` no cabeçalho e `AbsoluteEntry` nas linhas; `LineNumber`, `BaseObjectType`, `OrderEntry`, `OrderRowID`, `ReleasedQuantity`, `PickedQuantity`, `PreviouslyReleasedQuantity`, `BatchNumbers`, `SerialNumbers`, `DocumentLinesBinAllocations` e `UseBaseUnits`. Preservar capitalização e tipos. Exemplos públicos não são payloads universais.

## Gate de descoberta de contrato

Antes de habilitar qualquer escrita, produzir um relatório sanitizado com:

1. Versão/FP/patch SAP B1 e Service Layer, plataforma de banco, localização, empresa de homologação e configurações relevantes. Nunca copiar senha/cookie.
2. `$metadata` autorizado com hash e data, armazenado em local controlado; extrato mínimo no repositório apenas após remover informações específicas do cliente.
3. EntitySet, chave e tipos de PickLists; definição de ações, binding/ActionImport, parâmetros, retorno e nulabilidade. Registrar diferenças em relação à referência pública.
4. Mapeamento de cada tipo de documento elegível; validar código de objeto e semântica da linha. Não assumir que índice visual, posição no array, `LineNumber` e `OrderRowID` são intercambiáveis.
5. Fontes suportadas para documento, unidade, depósito, lote, saldo, compromisso e alocação; filtros, paginação, limites, frescor e autorizações. Consultas somente de leitura também precisam ser aprovadas e parametrizadas; não inventar entidade de saldo por lote.
6. Contrato por operação: campos graváveis, enumerações, coleções preservadas, retorno esperado, efeitos e releituras necessárias. Metadata descreve forma, não comprova todas as regras de negócio.
7. Evidência de teste real controlado para criação, alocação, confirmação parcial/total, alteração e recuperação. Registrar requests/responses sanitizados, sem divulgar informações comerciais.
8. Matriz de capacidades por empresa/versão/documento/gestão de item/bin. Capacidade desconhecida ou divergente permanece desabilitada.

## Mapeamento e invariantes do adaptador

- Manter identidade do documento-base separada da identidade da Pick List e de sua linha. Mapear explicitamente de ida e volta.
- Reconstruir comando a partir de estado remoto atual e mudança autorizada. Não reutilizar um JSON antigo completo como PUT: pode sobrescrever alterações legítimas.
- Preservar linhas e coleções não alteradas conforme comportamento homologado. Testar omissão, array vazio e substituição; nunca ativar cabeçalho de substituição de coleções por conveniência.
- Mapear lotes e bins por referências corretas, não por coincidência de posição. Validar significado de `BaseLineNumber` e `SerialAndBatchNumbersBaseLine` no objeto/operação utilizados.
- Confirmar unidade efetiva e comportamento de `UseBaseUnits`; não assumir que quantidade da tela está na unidade SAP do payload.
- Definir em contrato se quantidades são valores totais ou incrementais. Entrada local “mais 2” não autoriza enviar 2 como total nem somar novamente numa repetição.
- Não inferir limite por fórmula fixa entre `ReleasedQuantity`, `PreviouslyReleasedQuantity` e `PickedQuantity`; validar comportamento ao longo de liberações e confirmações parciais.
- Não habilitar seriais como se fossem lotes; possuem identidade e cardinalidade próprias.
- Bins obrigatórios exigem alocação coerente de item/lote/depósito/bin. Se não suportados, bloquear a operação com motivo claro [S6].

## Sequência de comandos proposta

**Planejar:** somente consultas SAP e persistência local do plano. Nenhum POST de criação de lista.

**Liberar:** registrar intenção → validar revisão local → reler SAP → verificar elegibilidade → criar Pick List, se ainda não houver → guardar identidade retornada → consultar alocação liberada → atualizar alocação conforme contrato → reler lista/alocação e conferir totais/origens → marcar liberado verificado.

Criação e alocação podem ser chamadas separadas, sem atomicidade conjunta. Lista criada com alocação pendente não é sucesso completo. Recuperar a mesma operação/lista; não criar outra. Não prometer que `$batch` elimina essa janela: suporte e transação das operações precisam de evidência específica.

**Confirmar:** registrar intenção de quantidade física → reler lista/alocação/documento → conferir origens e limites → aplicar atualização de picking pelo contrato homologado (candidato: PUT de linhas) → reler e comparar. `UpdateReleasedAllocation` não substitui confirmação da quantidade separada. O payload final só deve ser escrito após testes no ambiente.

**Alterar alocação:** exigir motivo, autoridade e plano revisado; consultar alocação atual; aplicar mudança permitida; reler. Se já houver separação, não presumir que pode ser realocada livremente.

## Transporte e erros

Sessão Login/Logout e cookies gerenciados exclusivamente no backend; confirmar política de sessão e balanceamento no ambiente. Nunca retransmitir cookie SAP ao navegador. Validar TLS, timeouts e limites; restringir host de destino e seguir paginação apenas dentro do serviço autorizado.

Encapsular filtros OData com allowlist de campos/operadores, escaping de literais e codificação de URL; não concatenar expressões fornecidas pelo usuário. Evitar `$select` que omita dados indispensáveis à validação.

Mapear erros por categoria (autenticação, autorização, regra SAP, conflito, indisponibilidade, resultado desconhecido), preservando código SAP e correlation ID em registro seguro. Não classificar toda falha como saldo insuficiente. O status HTTP isolado não comprova efeito de negócio; aplicar [reconciliação](04-seguranca-e-concorrencia.md).
