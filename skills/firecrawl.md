# Habilidade: Firecrawl CLI

Esta skill define a principal interface de busca e raspagem web do agente.

## Comandos 
- **Busca Primária:** `firecrawl search "<consulta>" --json`
- **Raspagem (Scrape):** `firecrawl scrape "<url>" --json` (Se for necessário raspar diretamente algo)

## Regras e Tratamento de Erros
- Se o `firecrawl search` falhar e estourar a quantidade de tentativas, use a estratégia de fallback de domínio do agente (`site:dominio.com assunto`).
- Se o `firecrawl scrape` falhar em uma URL específica, não aborte. Use o título/descrição do resultado da busca anterior, anote que a extração aprofundada falhou e continue a raspagem dos demais itens.