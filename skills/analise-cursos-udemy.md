# Habilidade: Análise de Cursos - Udemy

## Estratégia de Busca
- **Comando:** `firecrawl search "udemy [assunto]" --json` executado em `run_in_terminal`.
- **Fallback:** `firecrawl search "site:udemy.com.br [assunto]" --json`
- Priorize descobrir os dados pelos resultados do buscador apontando para o domínio `udemy.com.br`.

## Extração Necessária
Identifique as seguintes propriedades:
1. `nome_curso`
2. `duracao`
3. `nivel`
4. `preco`
5. `link`

## Lógica de Classificação de Nível
- **iniciante:** O título ou descrição do resultado contém as palavras "Introdução", "Primeiros Passos", "Fundamentos", "Básico" ou "Para Iniciantes".
- **intermediario:** Contém "Intermediário" ou implica experiência/conhecimento prévio no título/descrição.
- **avancado:** Contém "Avançado", "Profundo", "Expert", "Arquitetura" ou "Especialista".
*(Ordene sempre pelos mais bem avaliados usando a posição de proxy de relevância no buscador e eventuais dados de estrelas informados)*