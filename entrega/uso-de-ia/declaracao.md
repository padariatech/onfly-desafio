# Declaração de uso de IA

## Ferramenta e modelo

- **Ferramenta:** Claude Code, no aplicativo desktop da Anthropic (aba Code).
- **Modelo:** Claude Haiku 5.5 (`claude-haiku-5-5`).

## O que a IA fez

- Leu o `pytest` e os arquivos de `src/onfly_report/` para localizar onde cada falha se origina.
- Comparou cada função com a regra correspondente do `README.md`.
- Explicou a causa de cada defeito e sugeriu a correção.
- Rodou a suíte de testes, a amostra de validação, o relatório completo e o `ruff`, e comparou os resultados com a tabela do README.
- Escreveu `README-ENTREGA.md`, `sessao.md` e esta declaração.

## O que foi feito por mim

- Realizei a correção baseado na leitura da IA de com os erros estavam acontecendo.
- Comentarios e testes iniciais
