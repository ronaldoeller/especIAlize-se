# especIAlize-se
Agente que pesquisa os 10 cursos mais bem avaliados na Alura e na Udemy sobre um determinado assunto.

## Visão Geral
Sistema multi-agente que, dado um assunto qualquer, busca os 10 cursos mais bem avaliados na Alura e na Udemy, que têm relação com o assunto.

## Pré-requisitos
- Acesso à CLI do Firecrawl e ferramentas web.
- Interpretador ou interface baseada em modelos Mixture of Experts (MoE) que suporte as ferramentas `spawn_agent` e `run_in_terminal`.

## Como Iniciar
1. Leia as instruções em `AGENT.md` para inicializar.
2. O Maestro (Orquestrador) será ativado e solicitará o assunto a ser pesquisado.

## Estrutura de Diretórios
- `AGENT.md`: Instruções de inicialização para o agente.
- `personas/`: Agentes do sistema (Maestro, Alura, Udemy).
- `skills/`: Habilidades modulares (Firecrawl, Dispatch, Análise de cursos).
- `dados/`: Resultados salvos das recomendações de cursos (arquivos markdown).
