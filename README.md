# SAFE Picking

Diretrizes para uma aplicação web de picking manual integrada ao SAP Business One.

Repositório inicializado para receber a especificação e o material de orientação do DeepSeek Harness via pull request. Ainda não há aplicação executável.

## Diretriz de produto

O operador escolhe documento/linha, depósito elegível, lote e quantidade. O SAP B1 é a autoridade sobre estoque, documentos, alocações e estados. Planejar não reserva no SAP; liberar não confirma separação; confirmar picking não gera entrega automaticamente.

## Leitura e implementação

1. [AGENTS.md](AGENTS.md): regras para agentes e colaboradores.
2. [Índice e autoridade documental](docs/README.md).
3. [Operação e regras de domínio](docs/01-operacao-e-dominio.md).
4. [Service Layer e contratos](docs/02-service-layer.md).
5. [Arquitetura e modularidade](docs/03-arquitetura.md).
6. [Segurança, concorrência e rastreabilidade](docs/04-seguranca-e-concorrencia.md).
7. [Testes e critérios de aceite](docs/05-testes.md).
8. [Roadmap e decisões pendentes](docs/06-roadmap.md).
9. [Material de execução do DeepSeek Harness](harness/README.md).
10. [Fontes oficiais e limites de evidência](docs/07-fontes-e-evidencias.md).

## Estado do repositório

Inspeção em 2026-10-16: remoto sem commits, branches ou arquivos; clone vazio. Esta entrega cria apenas documentação. Não há stack, dependências, comandos de execução, testes automatizados ou integração SAP implementados. Os testes descritos são requisitos para a implementação futura, não resultados executados.

A branch `chore/repository-baseline` contém a base mínima para permitir revisão via PR; `docs/safe-picking-deepseek-guidelines` contém as diretrizes. Não há merge automático nem publicação em produção.

Para iniciar o trabalho técnico, use o [prompt inicial](harness/PROMPT_INICIAL.md) e conclua a etapa de descoberta antes de habilitar escrita no SAP. Dados e credenciais do cliente devem permanecer fora do Git e dos prompts.
