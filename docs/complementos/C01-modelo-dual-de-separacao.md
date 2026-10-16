# C01 — Modelo dual de separação

Estado: proposta. Subordinado a `AGENTS.md`. O modo manual é o produto; assistência e periféricos são camadas opcionais.

## Contexto do cliente-base

Hoje o picking é manual: o operador consulta a disponibilidade na origem e seleciona as quantidades (**Requisito**, [01](../01-operacao-e-dominio.md)). O modelo abaixo preserva isso como caso principal e organiza, sem mudar a semântica SAP, duas escolhas independentes.

## Eixo 1 — Modo de decisão (quem escolhe a origem)

| Modo | Comportamento | Disponibilidade |
|---|---|---|
| **M — Manual** | O operador escolhe documento/linha, depósito elegível, lote e quantidade. Nada vem pré-selecionado. | Único modo habilitado nas Etapas 0 a 5 |
| **S — Sugestão assistida** | O sistema preenche uma **proposta** no rascunho. O operador revisa, edita, rejeita ou aceita cada linha. Ver [C02](C02-disponibilidade-e-sugestao-assistida.md). | Etapa 6, atrás de gates; desabilitado por padrão |

Não existe modo que grave sem revisão do operador. Uma escolha automática silenciosa viola AGENTS e ADR-001.

## Eixo 2 — Canal de confirmação (como o operador informa)

| Canal | Comportamento |
|---|---|
| Toque/teclado | O operador digita ou toca na quantidade |
| Leitura | Leitor ou câmera **identificam** um candidato (item, bin, lote). Não confirmam nada por si só (`01`, passo 6) |
| Híbrido | Leitura quando possível; digitação com justificativa quando não |

Política de leitura por contexto: `scan = off | optional | required`.

- `optional`: a leitura preenche o candidato; o operador confirma a quantidade.
- `required`: a confirmação só habilita se o candidato lido coincidir com a origem planejada. Divergência vira ocorrência (R09); o operador registra a decisão com justificativa e a autoridade necessária. A leitura continua sem confirmar sozinha.

Os eixos são independentes: qualquer modo funciona com qualquer canal.

## Onde cada modo atua

No fluxo de `01`, a escolha de depósito/lote/quantidade ocorre no **planejamento** (passo 3). Portanto:

| Etapa do fluxo | Modo M | Modo S |
|---|---|---|
| Planejar (passo 3) | O operador informa quantidade por lote | O sistema propõe; o operador decide linha a linha |
| Liberar (passo 5) | Revalidação e comando conforme `02` | Idêntico: a proposta já virou plano manual |
| Separar e confirmar (passos 6–7) | O operador registra a quantidade real | Idêntico; nada é automático |

O Modo S nunca atua depois do planejamento. A separação física e a confirmação são sempre decisões do operador.

## Política de separação

Configuração **da aplicação**, não do SAP (**Hipótese**).

| Campo | Valores | Padrão |
|---|---|---|
| `modoDecisao` | `M`, `S` | `M` |
| `scan` | `off`, `optional`, `required` | `optional` |
| `criterioSugestao` | referência a critério homologado (DC02) | vazio |
| `exigirRevisaoEmFalta` | sim/não | sim |

Resolução por precedência `classe de item > depósito > perfil > padrão` (**Hipótese**, depende de DC04). Trocar `modoDecisao` para `S` exige capacidade habilitada, papel Administrador e gates de C02. Mudanças de política são auditadas. O Administrador não ignora regra SAP (`04`).

## Auditoria de modo

Complementa a [trilha de `04`](../04-seguranca-e-concorrencia.md). Por linha, registrar em eventos append-only:

- modo usado e política vigente;
- proposta (identidade e conteúdo mínimo), quando houver;
- valor final escolhido e se divergiu da proposta;
- canal e tipo de dispositivo;
- ator humano, motivo de divergência e instante UTC;
- correlação com plano, comando e objeto SAP.

Serve para medir quanto a sugestão é aceita e para sustentar decisões de produto. Copiar um resumo para um UDF no SAP é **Hipótese** e depende de DC07 e D08.

## Telas (complementam `01`)

**Consulta de origens (modo M)**
- Lista de origens por depósito elegível e lote, com medidas **separadas** (existente, alocado, disponível para liberar), unidade, fonte e instante da consulta.
- Valor sem fonte confiável aparece como "não verificado" e impede liberação. Nunca como zero.
- Motivos de indisponibilidade visíveis. Ordenação e filtros são só de apresentação e não pré-selecionam origem.
- Quantidade informada por origem; total planejado contra pendente da linha, atualizado em tempo real.

**Revisão**
- Diferença desde a consulta anterior. Mudança externa invalida a aprovação baseada em snapshot antigo (R07).
- Botão "Salvar planejamento" não chama liberação SAP. A tela mostra "planejado localmente, não reservado no SAP" (R08).

**Operação em coletor e tablet**
- Foco automático no campo de leitura, feedback visual e sonoro de sucesso/erro, alvos de toque de pelo menos 44 px.
- Todos os fluxos utilizáveis sem periférico, exceto onde a política disser `scan = required`.
- Rede caída preserva o rascunho e nunca exibe sucesso SAP (`01`, T26).

## Princípios de compatibilidade

- A disponibilidade é revalidada no comando de liberação (R07). Se divergir, mostrar a diferença e pedir nova decisão. Não recalcular proposta em silêncio.
- Falha do motor, da leitura ou de um periférico não impede o fluxo manual.
- Uma proposta não reserva nada no SAP (R08). Um aviso local de concorrência (ex.: outro operador está planejando a mesma origem) é **Hipótese**, apenas informativo, e nunca entra no cálculo de disponibilidade.
- Desligar modo S e periféricos deve manter T06 a T11 aprovados.
