# C03 — Periféricos e embalagem

Estado: proposta. Periféricos são **canais** de entrada e saída do fluxo existente. Embalagem é expansão da Etapa 6. Nada daqui é pré-requisito do fluxo manual.

## Periféricos como canais

A interface fala com uma abstração de dispositivo (**Hipótese**; tecnologia por ADR). O domínio recebe apenas eventos normalizados: item lido, bin lido, lote lido, peso capturado. Nada fora dessa camada depende de um aparelho específico.

| Periférico | Abordagem | Evidência |
|---|---|---|
| Leitor de código de barras (teclado-wedge, coletor Android) | Campo de captura focado; tratar prefixo/sufixo e intervalo entre leituras | Hipótese |
| Câmera do dispositivo | Leitura por API do navegador quando disponível, com alternativa por biblioteca; o suporte varia por navegador e deve ser verificado | Hipótese |
| Códigos GS1-128 | Interpretar identificadores de aplicação (item, lote, validade, série, quantidade) **somente** se o cliente usar o padrão (DC06) | Hipótese |
| Impressora de etiquetas | O backend atua como gateway de impressão; formato a definir | Hipótese |
| Balança | Captura por gateway local ou API do dispositivo, sempre com digitação como alternativa | Hipótese |
| Coletor RF, voz, pick-to-light | Fora do escopo; a abstração deixa a porta aberta | — |

## Regras

- A leitura **identifica candidato**; não confirma quantidade, não libera e não substitui a decisão do operador (`01`, passo 6).
- `scan = required` apenas **bloqueia** a confirmação enquanto o candidato lido não coincidir com a origem planejada. Divergência vira ocorrência (R09); nunca troca de lote ou depósito em silêncio.
- Código lido que corresponda a mais de uma identidade (mesmo número de lote em contextos distintos, T01) exige desambiguação explícita.
- O mapa código → item/bin/lote é resolvido no servidor, por empresa, com cache de prazo configurável. O navegador não decide identidade.
- Entradas de dispositivo são dados não confiáveis: validar tamanho, caracteres e formato no servidor (`04`).
- Nenhum periférico ou gateway carrega credenciais SAP. Permissões do navegador (como câmera) exigem contexto seguro (HTTPS).
- Falha de leitura, impressão ou captura nunca perde o rascunho nem exibe sucesso SAP.
- Nenhuma funcionalidade pode **exigir** periférico, exceto onde a política disser `scan = required` (C01).

## Embalagem e expedição (Etapa 6)

- O processo nativo termina em entrega e packing slip (**Referência**, C5). Nesta aplicação, confirmar picking **não** gera entrega (R10, ADR-001, T11).
- Geração de entrega ou fatura é ação separada, explícita, autorizada, e só após homologação do contrato correspondente. Faz parte de "packing/entrega" da Etapa 6, com escopo aprovado à parte.
- Volumes, peso e dimensões: verificar no `$metadata` real se o documento de marketing possui coleção de pacotes. Se não houver ou não bastar, as opções são UDO/UDT/UDF do cliente (**Hipótese**, DC08 e D08). Nada é criado no SAP antes da decisão.
- **Relatos** a considerar nos testes de homologação, sem valor de regra:
  - quando uma linha do pedido é dividida em duas listas e vira uma única entrega, o campo de vínculo com a lista na linha de entrega guarda apenas uma delas (C11). O vínculo lista ↔ entrega deve ser guardado também pela aplicação;
  - o campo de observação de picking pode não ser gravado na criação da entrega via Service Layer (C12). Se persistir, a gravação em duas etapas não é transacional e exige reconciliação (`04`);
  - a entrega do saldo de pedido originado em cotação pode falhar porque o documento-base já foi encerrado (C13). Verificar o estado de cada linha-base antes de montar o documento.

## Testes complementares

| Caso | Regra | Cenário e resultado exigido |
|---|---|---|
| TC10 | `01` passo 6, T10/T11 | Leitura correta de item, bin e lote preenche o candidato e não confirma nem libera |
| TC11 | `scan = required` | Candidato divergente bloqueia a confirmação; justificativa e papel registrados; sem troca silenciosa |
| TC12 | T01 | Código ambíguo exige desambiguação |
| TC13 | T21/T26 | Falha de leitura ou periférico desconectado: rascunho preservado, fluxo manual disponível, sem credenciais em logs |
| TC14 | T20 | Mapa de código resolvido por empresa no servidor; o navegador não consegue trocar empresa ou identidade |
| TC15 | R10/T11 | Com packing habilitado, confirmar picking continua sem criar entrega; a entrega só nasce de ação própria |
