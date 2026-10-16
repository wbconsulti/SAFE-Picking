# ADR-002 — Monólito modular e adaptador SAP

## Estado

Proposto em 2026-10-16. Linguagem, frameworks, banco e hospedagem não escolhidos.

## Contexto

Repositório vazio, operação integrada com estado remoto e necessidade de evolução incremental. Regras de domínio, sessões SAP e recuperação não devem ficar espalhadas na interface.

## Decisão proposta

Uma aplicação web com backend intermediário e módulos explícitos de demanda, disponibilidade, planejamento, liberação, execução, integração e auditoria. Domínio independente de infraestrutura. Persistência local para planos, revisão, comandos e eventos; executor/reconciliador recuperável. Adaptador Service Layer encapsula contratos por capacidade.

## Alternativas consideradas

- Browser direto no SAP: rejeitado por exposição de sessão e dificuldade de coordenar autorização/comandos.
- Microserviços por fase: adiado por custo operacional e transações distribuídas sem necessidade comprovada.
- Controlador monolítico sem fronteiras: rejeitado por acoplamento e dificuldade de testar invariantes.

## Consequências

Permite testar domínio e trocar detalhes de transporte sem reescrever telas. Requer contratos internos, controle de dependências e persistência de comandos. A tecnologia deve ser escolhida em novo ADR antes da etapa 1; não há instalação ou scaffold implícito nesta entrega.
