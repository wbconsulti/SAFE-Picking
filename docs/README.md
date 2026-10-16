# Diretrizes SAFE Picking

Versão documental inicial: 2026-10-16. Estado: especificação para revisão; integração não homologada.

## Autoridade

As premissas do usuário são normativas para o produto: picking manual, escolha por documento/linha, depósito, lote e quantidade, uso do DeepSeek Harness. O comportamento permitido pelo SAP prevalece sobre sugestões de UX ou fórmulas locais. Lacunas não autorizam flexibilizar regras.

Os documentos abaixo compõem uma única diretriz. Em conflito, interromper a decisão afetada e registrar resolução; não selecionar silenciosamente a regra mais permissiva.

| Documento | Conteúdo |
|---|---|
| [01 — Operação](01-operacao-e-dominio.md) | Fluxos, estados locais, seleção e quantidades |
| [02 — Service Layer](02-service-layer.md) | Endpoints, metadata e homologação |
| [03 — Arquitetura](03-arquitetura.md) | Módulos, dados e contratos internos propostos |
| [04 — Segurança](04-seguranca-e-concorrencia.md) | Autorizações, concorrência, auditoria e recuperação |
| [05 — Testes](05-testes.md) | Matriz de requisitos e evidências exigidas |
| [06 — Roadmap](06-roadmap.md) | Fatias, gates e decisões pendentes |
| [07 — Fontes](07-fontes-e-evidencias.md) | Referências oficiais e limites de verificação |
| [ADR-001](decisions/0001-autoridade-sap-e-planejamento-manual.md) | Autoridade SAP e fases separadas |
| [ADR-002](decisions/0002-monolito-modular-e-adaptador-sap.md) | Arquitetura proposta |
| [Harness](../harness/README.md) | Ordem de execução e handoff |

## Convenção de evidência

- **Requisito:** decisão de produto a implementar, não afirmação de capacidade SAP.
- **Referência:** capacidade descrita pela SAP; consultar fontes 07.
- **Hipótese:** desenho proposto, sujeito a ADR e teste.
- **Homologado:** somente com versão, empresa de teste, contrato, caso e resultado registrados. Nenhuma capacidade recebeu este estado nesta entrega.

Inspeção: `gh repo view`, clone, `git status` e `git ls-remote` confirmaram remoto vazio. Não havia AGENTS.md, arquitetura ou convenção de ADR no repositório. O contexto anterior recuperado confirmou a seleção manual por depósito/lote; parte das respostas anteriores veio apenas como referências opacas, portanto não é tratada como especificação completa disponível.
