# SAFE Picking — instruções para agentes

## Objetivo e ordem de leitura

Construir, quando autorizado, uma aplicação web de picking manual SAP B1: documento/linha → depósito elegível → lote → quantidade. Leia [docs/README.md](docs/README.md), os documentos 01–07 e [harness/README.md](harness/README.md) antes de propor implementação. Instruções explícitas do responsável prevalecem; registre conflitos e não altere a semântica SAP para acomodar a interface.

## Invariantes

- SAP B1 é a fonte da verdade. Não escrever diretamente em tabelas SAP, criar estoque paralelo ou simular sucesso de integração.
- Seleção manual é requisito. Não implementar FIFO/FEFO, substituição de lote, depósito ou quantidade sem escolha explícita do usuário e regra homologada.
- Planejamento local não é reserva SAP. Liberação, alocação liberada, confirmação física e documento logístico são etapas distintas.
- Depósito deve ser compatível com a linha-base e autorizações. Um saldo em outro depósito não autoriza usá-lo. Não reescrever a linha-base nem criar transferência implicitamente.
- Lotes não são globais: vincular empresa, item, depósito, identidade SAP e linha. Respeitar bloqueios, validade, unidades e alocações existentes. Bins obrigatórios não podem ser ignorados.
- Valores indisponíveis/desconhecidos não equivalem a zero nem autorizam escrita. Divergência impede o comando afetado.
- Não adivinhar endpoints, enums, campos, índices, códigos de objeto, suporte a ETag ou atomicidade. Usar fontes oficiais, `$metadata` do ambiente e teste controlado.
- Credenciais e cookies SAP ficam no servidor. Não incluir segredos/dados reais em logs, fixtures, commits, prompts ou capturas.
- Timeout de escrita pode significar sucesso remoto: reconciliar antes de repetir. Nenhuma repetição cega de POST/PUT.

## Método de trabalho

1. Confirmar repositório, branch, estado Git e instruções locais. Preservar trabalho existente e referências em `sources/`, se presentes.
2. Entregar uma fatia do roadmap por vez, com requisito, critérios de aceite e evidências. Não criar aplicação inteira em um único passo.
3. Manter domínio independente de HTTP, banco e SAP; contrato de integração em adaptador; interface sem regra de estoque duplicada.
4. Escolher stack e bibliotecas em ADR proposto, com justificativa; não tratar arquitetura proposta como código existente.
5. Executar testes relevantes, registrar comando/resultado e limitações. Mocks aprovados não comprovam homologação SAP.
6. Atualizar documentação e rastreabilidade ao mudar regra ou contrato; não reduzir critérios para fazer testes passarem.
7. Fazer commits pequenos e PR revisável. Merge, implantação e operações no SAP produtivo exigem autorização explícita específica.

## Condições de bloqueio

Bloquear escrita para combinação não homologada de empresa, versão, documento, gestão do item e depósito. Não contornar rejeição SAP com SQL ou payload inventado. Registrar evidência sanitizada e decisão necessária. Continuar somente trabalho independente seguro.

## Definição de pronto

Critérios da fatia satisfeitos; testes aplicáveis executados; revisão de permissões, concorrência e falhas parciais; contratos e documentação atualizados; PR com arquivos, validações e limitações. Se não houve SAP real, declarar isso expressamente.
