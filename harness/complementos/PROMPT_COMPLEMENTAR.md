# Prompt complementar — sugestão assistida e periféricos (SAFE Picking)

Este prompt **complementa** [PROMPT_INICIAL.md](../PROMPT_INICIAL.md); não o substitui nem altera `AGENTS.md`. Use-o somente após o responsável autorizar explicitamente a rodada correspondente. Nas Etapas 0 a 5 do roadmap, siga apenas o fluxo manual.

---

Você já concluiu a leitura do contexto base: `AGENTS.md`, `docs/README.md`, `docs/01` a `07`, os ADRs e `harness/README.md`. Agora leia também `docs/complementos/` (README, C01 a C05). Se algum desses arquivos não estiver disponível, pare e informe.

## Precedência

`AGENTS.md` e as invariantes prevalecem sobre `docs/complementos/` e sobre este prompt. Se houver conflito, interrompa a decisão afetada, registre-o e peça resolução. Nunca escolha a regra mais permissiva. Instruções explícitas do responsável no chat prevalecem, mas registre o conflito que elas criem com as invariantes.

## O que este complemento pede

O cliente-base trabalha de forma **manual**: consulta a disponibilidade na origem e seleciona as quantidades. O produto preserva isso como caso principal. Duas camadas opcionais podem ser estudadas, conforme `C01`:

- **Modo de decisão:** M manual (único habilitado nas Etapas 0 a 5) e S sugestão assistida (Etapa 6, atrás de gates).
- **Canal de confirmação:** toque/teclado, leitura (que apenas identifica candidato) e híbrido, com `scan = off | optional | required`.

Não existe modo que grave sem revisão do operador. Não proponha um.

## Rodadas (somente após autorização)

| Rodada | Pré-condição | Trabalho | Evidência de saída |
|---|---|---|---|
| G — Sugestão assistida | Gates G1 a G5 de `C02` satisfeitos e registrados | Rascunho de ADR, contrato `ConsultarOrigens` detalhado, motor puro com testes de propriedade, telas de revisão e aceitação por linha | TC01 a TC09 executados; regressão manual (T06 a T11) aprovada |
| H — Periféricos | Autorização do responsável; DC04 a DC06 respondidas | Abstração de dispositivo, leitura como identificação, `scan` por política, desambiguação de códigos | TC10 a TC14 executados; fluxo manual intacto sem periférico |

Embalagem, entrega e fatura (`C03`) ficam fora destas rodadas. Só entram com escopo próprio da Etapa 6.

## Regras para o trabalho

1. **Não relaxe invariantes.** Sem FIFO/FEFO, substituição de lote, depósito ou quantidade sem escolha explícita do usuário e regra homologada. Sem estoque paralelo, sem escrita direta em tabelas SAP, sem entrega automática (R10).
2. **Sem critério padrão.** A sugestão só usa critério que o cliente homologou (DC02). Sem critério, não há proposta.
3. **Desconhecido não é zero.** Medida sem fonte confiável é "não verificado" e impede liberação. Nunca distribua saldo agregado entre lotes. Use decimal exato (R03).
4. **Nenhuma fonte assumida.** Não invente entidade, view ou consulta de saldo por lote/bin. A fonte é decisão D04 do consultor SAP. Você consome um contrato; ele define a implementação.
5. **A proposta não reserva.** Ela vive no rascunho, é marcada "não reservada no SAP", é invalidada por mudança de snapshot e nunca avança sozinha. Aceitar é ação explícita do operador, por linha, e fica auditada.
6. **Leitura identifica, não confirma.** `scan = required` apenas bloqueia enquanto o candidato lido não coincidir. Divergência vira ocorrência (R09).
7. **O modo manual nunca depende da camada opcional.** Se o motor ou um periférico falhar ou for desligado, o fluxo manual segue funcionando. Teste isso.
8. **Escrita desabilitada por padrão.** Toda capacidade de escrita no SAP permanece desabilitada até haver contrato homologado para a combinação real (`02`, `05`). Trabalhe com dados fictícios e declare que mocks não comprovam homologação.
9. **Não adivinhe contratos SAP.** Use `docs/02` e o `$metadata` real. O que está em `C04` é referência e relato, não contrato. Em especial, não trate `UpdateReleasedAllocation` como confirmação física sem teste real (D09, T12).
10. **Segredos e dados reais fora de tudo.** Sem credenciais, cookies ou dados do cliente em prompts, logs, fixtures, commits ou capturas. Credenciais SAP e sessão ficam no servidor.
11. **Stack só por ADR.** Este complemento não prescreve linguagem ou framework. Descreva comportamento e contratos; a tecnologia segue o ADR de fundação.
12. **Idempotência e falhas parciais valem aqui também.** Resultado desconhecido exige reconciliação antes de repetir (`04`).

## Perguntas ao responsável

Faça numa única mensagem curta, com a opção padrão de cada uma. Não peça credenciais.

- **DC01:** há interesse e aprovação para a sugestão assistida? Em qual ponto? (padrão: não; só modo manual)
- **DC02:** qual critério de origem o cliente homologa, por classe de item/depósito? (padrão: nenhum)
- **DC03:** como o operador consulta hoje a disponibilidade na origem? (padrão: a descobrir na Etapa 0)
- **DC04:** política de leitura por contexto e quem pode justificar divergência? (padrão: `optional`)
- **DC05/DC06:** quais periféricos existem e onde estão os códigos de barras de item, bin e lote? (padrão: leitor e câmera; códigos a levantar)
- **DC07:** a auditoria de modo fica só na aplicação ou também no SAP? (padrão: só na aplicação)
- **DC08:** embalagem além do nativo? (padrão: nativo; Etapa 6)

Sem resposta, assuma o padrão, registre a suposição e siga com trabalho independente e seguro.

## Como relatar

Use [MODELO_ENTREGA.md](../MODELO_ENTREGA.md) e acrescente:

- modo de decisão e canal envolvidos na fatia;
- gates G1 a G5 e decisões DC01 a DC08 relevantes, com estado;
- casos TC executados, com comando e resultado;
- o que foi testado só com mocks e o que ainda depende de homologação;
- confirmação de que `AGENTS.md`, `docs/01` a `07` e os ADRs não foram alterados.

Seja direto. Não recapitule o passo a passo. Declare limitações e pendências em vez de esconder lacunas.
