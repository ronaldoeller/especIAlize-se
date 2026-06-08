# Protocolo de Despacho (Dispatch & Handoff)

## Ordem de Invocação
1. Agente Alura
2. Agente Udemy

## Envelope de Despacho (Maestro -> Agente)
O Maestro usará a ferramenta `spawn_agent` fornecendo o seguinte modelo exato no prompt do agente-alvo:

```text
## DESPACHO: [NOME_DO_AGENTE]
### referencia_persona
[INSERIR O CONTEÚDO INTEGRAL DA PERSONA: personas/alura.md ou personas/udemy.md]

### tarefa
Busque e retorne os 10 cursos mais bem avaliados sobre o assunto informado, seguindo sua persona e skill associada.

### contexto
Assunto: [Assunto informado pelo usuário]

### saida_esperada
Envelope de Resposta.
```
*Se o assunto não estiver no contexto, o agente acionado deverá falhar imediatamente retornando erro no respectivo envelope.*

## Envelope de Resposta (Agente -> Maestro)
Todos os sub-agentes devem retornar EXATAMENTE nesta estrutura:

```text
## RESPOSTA: [NOME_DO_AGENTE]
### estado
[sucesso | erro]

### resumo
[Resumo em 2-3 frases voltado para o usuário]

### dados
[Lista numerada em chave-valor. OBRIGATÓRIO não usar tabelas markdown]

### erros
[Apenas se estado for erro: descrição detalhada da falha]
```