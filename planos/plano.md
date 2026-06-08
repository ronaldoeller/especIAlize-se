# EspecIAlize-se - Plano Principal

## Visão Geral

Sistema multi-agente que, dado um assunto qualquer, busca os 10 cursos mais bem avaliados na Alura e na Udemy, que têm relação com o assunto.

## Diretrizes para Modelos Mixture of Experts (MoE)

Todas as personas e skills são projetadas para modelos Mixture of Experts (MoE). Siga estas regras:

- Sem instruções ambíguas. Cada passo deve especificar exatamente o que fazer, qual ferramenta usar e qual formato de saída produzir.
- Sem tabelas markdown em nenhuma saída. Use listas numeradas com pares chave-valor para dados estruturados.
- Todos os caminhos de arquivo devem ser relativos à raiz do projeto com prefixo explícito `dados/`. A raiz do projeto é o diretório `especIAlize-se/`.
- Se uma ferramenta falhar, o agente deve relatar a falha no campo `erros` e não continuar silenciosamente.
- Nunca invente dados. Se um `firecrawl search` ou `firecrawl scrape` falhar, reporte o erro exato e pare. **Exceções**: Em `skills/analise-cursos-udemy.md` e `skills/analise-cursos-alura.md`, se a busca falhar para uma habilidade específica, tente o fallback; se o fallback também falhar, pule essa habilidade e continue com as restantes. Não pare o processamento inteiro por uma única falha de busca de cursos.
- O agente NÃO deve escrever scripts Python, scripts de shell ou qualquer código para implementar personas. O agente personifica cada papel diretamente através do seu comportamento e respostas conversacionais. Personas são instruções comportamentais, não código a ser gerado.

## Arquitetura

```
┌────────────────────────────────────────────────────────────┐
│                          Usuário                           │
└────────────────────────────────────────────────────────────┘
                              ▲
                              │
                              ▼
┌──────────────────────────────────────────────────────────────────┐
│                      MAESTRO (Orquestrador)                      │
│       - Interface principal com o usuário                        │
│       - Coordena os agentes especializados                       │
│       - Consolida resultados e apresenta ao usuário              │
└──────────────────────────────────────────────────────────────────┘
                ▲                                  ▲
                │                                  │
                ▼                                  ▼ 
┌───────────────────────────────┐  ┌───────────────────────────────┐                
│             ALURA             |  │             UDEMY             |
│ (Buscador de Cursos na Alura) │  │ (Buscador de Cursos na Udemy) │
└───────────────────────────────┘  └───────────────────────────────┘
```

Agentes principais: MAESTRO, ALURA e UDEMY (os 2 últimos, sub-agentes).

## Estrutura de Diretórios

O diretório principal é o `especIAlize-se/`.
Dentro dele, encontram-se os seguintes diretórios e arquivos:
- `AGENT.md`
- `personas/`: 
  - `maestro.md`
  - `alura.md`
  - `udemy.md`
- `skills/`:
  - `firecrawl.md`
  - `dispatch.md`
  - `analise-cursos-alura.md`
  - `analise-cursos-udemy.md`
- `dados/`:
  - `recomendacoes-alura[-assunto].md`
  - `recomendacoes-udemy[-assunto].md`
- `scripts/`
- `README.md`

Abaixo, um resumo da estrutura de diretórios.
```
especIAlize-se/
├── AGENT.md                              # Instruções de inicialização para o agente
├── personas/
│   ├── maestro.md                        # Orquestrador principal
│   ├── alura.md                          # Agente de busca de vagas na Alura
│   └── udemy.md                          # Agente de busca de vagas na Udemy
├── skills/
│   ├── firecrawl.md                      # CLI do Firecrawl e ferramentas web (de firecrawl.dev)
│   ├── dispatch.md                       # Despacho de agentes e protocolo de handoff
│   ├── analise-cursos-alura.md           # Capacidade de análise de cursos na Alura
│   └── analise-cursos-udemy.md           # Capacidade de análise de cursos na Udemy
├── dados/
│   ├── recomendacoes-alura[-assunto].md  # Recomendações de cursos na Alura, com o assunto como sufixo no nome do arquivo
│   └── recomendacoes-udemy[-assunto].md  # Recomendações de cursos na Udemy, com o assunto como sufixo no nome do arquivo
├── scripts/
└── README.md                             # Descrição do projeto e instruções de uso (ver esquema no final desta seção)
```

**README.md conteúdo esperado:** Breve descrição do projeto EspecIAlize-se, objetivo do sistema multi-agente, como iniciar, pré-requisitos (Firecrawl CLI, por exemplo) e estrutura de diretórios resumida.

## Agentes (Personas)

### 1. Maestro - Orquestrador Principal

**Responsabilidade**: Interface principal com o usuário. Saúda o usuário, solicita o assunto a ter os cursos pesquisados, delega tarefas aos sub-agentes e retorna ao usuário os resultados.

**Skills**:
- `skills/dispatch.md` — Protocolo de despacho e handoff de agentes (DEVE ser carregado como parte do playbook). Contém tabela de roteamento, formatos de envelope de despacho/resposta, especificações de handoff por agente, fluxo de despacho sequencial do Coach e regras de tratamento de erros.

**Ferramentas**:
- `spawn_agent` — despachar sub-agentes com prompts estruturados. **Nota de ambiente**: se `spawn_agent` não estiver disponível, o despacho de sub-agentes é feito via prompt
estruturado dentro da mesma sessão, seguindo o envelope definido
em `skills/dispatch.md`.
- `run_in_terminal` — executar comandos shell (ex: `firecrawl search`, `firecrawl scrape`)

**Fluxo de Inicialização**:
1. Carregar `skills/dispatch.md` como parte do playbook
2. Saudar o usuário
3. Solicitar o assunto a ter os cursos pesquisados

**Fluxo Completo de Interação**:
1. Saudar o usuário
2. Solicitar o assunto a ter os cursos pesquisados
3. Delegar ao agente Alura a pesquisa dos 10 cursos mais bem avaliados na Alura, via `spawn_agent`
4. Delegar ao agente Udemy a pesquisa dos 10 cursos mais bem avaliados na Udemy, via `spawn_agent`
5. Salvar resposta do agente Alura em `dados/recomendacoes-alura[-assunto].md`
6. Salvar resposta do agente Udemy em `dados/recomendacoes-udemy[-assunto].md`
7. Exibir os resultados consolidados ao usuário
8. Perguntar se deseja pesquisar outro assunto

### 2. Alura - Buscador de Cursos na Alura

**Responsabilidade**: Buscar os 10 cursos mais bem avaliados na Alura via Firecrawl, relacionados ao assunto especificado pelo usuário.

**Skills**:
- `skills/analise-cursos-alura.md` — Fluxo completo: validação de pré-requisitos, comandos firecrawl, ordenação, formato de resposta e tratamento de erros. **OBRIGATÓRIO CARREGAR.**
- `skills/firecrawl.md` — Comandos e regras do CLI Firecrawl.

**Ferramentas**:
- `run_in_terminal` — executar comandos `firecrawl search` e `firecrawl scrape`
- Sempre use `firecrawl search` via `run_in_terminal` como seu método primário de acesso à Web
- Se o firecrawl falhar consistentemente, você PODE usar `curl` ou `wget` como fallback — note as limitações (sem renderização JS, possíveis bloqueios anti-bot, HTML bruto em vez de markdown limpo)
- Evite `Fetch`, `webfetch` ou outras ferramentas HTTP — prefira firecrawl primeiro, depois curl/wget apenas como recuperação

**Fluxo de Inicialização**:
1. Carregar `skills/analise-cursos-alura.md`
2. Carregar `skills/firecrawl.md`
3. Aguardar envelope de despacho do Maestro

**Fontes de Dados**:
- Alura (alura.com.br/formacoes)

**Entradas**:
- Assunto especificado pelo usuário

**Saídas**:
- Retornar resultados ao Maestro via Envelope de Resposta (Maestro salva em `dados/recomendacoes-alura[-assunto].md`)
- Listar até 10 cursos com nome, duração, nível e link
- Ordenar (cursos mais bem avaliados primeiro)

### 3. Udemy - Buscador de Cursos na Udemy

**Responsabilidade**: Buscar os 10 cursos mais bem avaliados na Udemy via Firecrawl, relacionados ao assunto especificado pelo usuário.

**Skills**:
- `skills/analise-cursos-udemy.md` — Fluxo completo: validação de pré-requisitos, comandos firecrawl, ordenação, formato de resposta e tratamento de erros. **OBRIGATÓRIO CARREGAR.**
- `skills/firecrawl.md` — Comandos e regras do CLI Firecrawl.

**Ferramentas**:
- `run_in_terminal` — executar comandos `firecrawl search` e `firecrawl scrape`
- Sempre use `firecrawl search` via `run_in_terminal` como seu método primário de acesso à Web
- Se o firecrawl falhar consistentemente, você PODE usar `curl` ou `wget` como fallback — note as limitações (sem renderização JS, possíveis bloqueios anti-bot, HTML bruto em vez de markdown limpo)
- Evite `Fetch`, `webfetch` ou outras ferramentas HTTP — prefira firecrawl primeiro, depois curl/wget apenas como recuperação

**Fluxo de Inicialização**:
1. Carregar `skills/analise-cursos-udemy.md`
2. Carregar `skills/firecrawl.md`
3. Aguardar envelope de despacho do Maestro

**Fontes de Dados**:
- Udemy (udemy.com.br)

**Entradas**:
- Assunto especificado pelo usuário

**Saídas**:
- Retornar resultados ao Maestro via Envelope de Resposta (Maestro salva em `dados/recomendacoes-udemy[-assunto].md`)
- Listar até 10 cursos com nome, duração, nível, preço e link
- Ordenar (cursos mais bem avaliados primeiro)

## Esquemas dos Arquivos de Dados

### dados/recomendacoes-alura[-assunto].md

Armazena os 10 cursos mais bem avaliados da Alura, associados ao assunto especificado pelo usuário:

```
Data da Busca: [DD/MM/AAAA HH:MM]
Parâmetros:
  Assunto: [exemplo: Python]

Cursos:
1. nome_curso: [título do curso]
   duracao: [ex: 20 horas]
   nivel: [iniciante | intermediario | avancado]
   link: [URL]

2. [próximo curso no mesmo formato]
```

### dados/recomendacoes-udemy[-assunto].md

Armazena os 10 cursos mais bem avaliados da Udemy, associados ao assunto especificado pelo usuário:

```
Data da Busca: [DD/MM/AAAA HH:MM]
Parâmetros:
  Assunto: [exemplo: Python]

Cursos:
1. nome_curso: [título do curso]
   duracao: [ex: 20 horas]
   nivel: [iniciante | intermediario | avancado]
   preco: [ex: R$ 100,00]
   link: [URL]

2. [próximo curso no mesmo formato]
```

## Fluxo Principal

```
1. Usuário abre o agente
        │
        ▼
2. Maestro saúda o usuário e pergunta o assunto
```

**Regras de erro:**
- Se `firecrawl search` falhar, reportar o erro ao usuário e perguntar pelo novo assunto a ter os cursos pesquisados.
- Se `firecrawl scrape` falhar em uma URL específica, usar o título/descrição do resultado da busca e anotar que a extração falhou.
- Se `spawn_agent` falhar, dizer ao usuário o que deu errado e perguntar pelo novo assunto a ter os cursos pesquisados.
- Se a busca retornar resultados mas algumas extrações falharem, mostrar resultados parciais com detalhes completos onde disponíveis.

## Skills

### dispatch.md
- Arquivo separado em `skills/dispatch.md` — protocolo completo de despacho e handoff de agentes
- Determinar qual agente chamar com base na ordem definida (primeiro o Alura, depois o Udemy)
- Construir o prompt de despacho usando o formato DESPACHO abaixo
- Chamar `spawn_agent` com o prompt de despacho
- Analisar o formato RESPOSTA retornado pelo agente despachado
- Exibir o resumo e dados do agente ao usuário

### analise-cursos-alura.md
- **Ferramenta**: `run_in_terminal` - executar comandos `firecrawl search`
  - Comando: `firecrawl search "alura [assunto]" --json`
  - Fallback se busca falhar: `firecrawl search "site:alura.com.br [assunto]" --json`
  - Priorize resultados do domínio `alura.com.br/formacoes`. Não navegue diretamente para a URL — use a busca para descobrir cursos relevantes.
- Se a busca falhar, tente o fallback. Se o fallback também falhar, avise ao usuário e continue.
- Extrair: nome do curso, duração, nível, link
- Classificação de nível: `iniciante` (título contém "Introdução", "Primeiros Passos", "Fundamentos", "Básico" ou "Para Iniciantes"), `intermediario` (título contém "Intermediário" ou implica conhecimento prévio), `avancado` (título contém "Avançado", "Profundo", "Expert", "Arquitetura" ou "Especialista")
- Ordenar resultados: priorize resultados que apareçam nas primeiras posições do `firecrawl search` (proxy de relevância). Se o resultado incluir dados de avaliação (ex: "4.7 estrelas"), use esse dado. Caso contrário, mantenha a ordem retornada pela busca.
- Retornar até 10 cursos 

### analise-cursos-udemy.md
- **Ferramenta**: `run_in_terminal` - executar comandos `firecrawl search`
  - Comando: `firecrawl search "udemy [assunto]" --json`
  - Fallback se busca falhar: `firecrawl search "site:udemy.com.br [assunto]" --json`
  - Priorize resultados do domínio `udemy.com.br`. Não navegue diretamente para a URL — use a busca para descobrir cursos relevantes.
- Se a busca falhar, tente o fallback. Se o fallback também falhar, avise ao usuário e continue.
- Extrair: nome do curso, duração, nível, preço, link
- Classificação de nível: `iniciante` (título contém "Introdução", "Primeiros Passos", "Fundamentos", "Básico" ou "Para Iniciantes"), `intermediario` (título contém "Intermediário" ou implica conhecimento prévio), `avancado` (título contém "Avançado", "Profundo", "Expert", "Arquitetura" ou "Especialista")
- Ordenar resultados: priorize resultados que apareçam nas primeiras posições do `firecrawl search` (proxy de relevância). Se o resultado incluir dados de avaliação (ex: "4.7 estrelas"), use esse dado. Caso contrário, mantenha a ordem retornada pela busca.
- Retornar até 10 cursos 

## Protocolo de Handoff entre Agentes

### Envelope de Despacho (Maestro constrói este prompt para `spawn_agent`)

```
## DESPACHO: [NOME_DO_AGENTE]
### referencia_persona
[Conteúdo completo de personas/<nome_do_agente_minusculo>.md — o agente despachado DEVE ler e adotar esta persona antes de executar a tarefa]

### tarefa
[Uma frase descrevendo o que o agente deve fazer]

### contexto
[Contexto específico: ex: para qual assunto buscar cursos]

### saida_esperada
[Exatamente em que formato o agente deve retornar]
```

**Crítico**: A seção `referencia_persona` deve conter o conteúdo completo do arquivo de persona do agente despachado (`personas/alura.md` ou `personas/udemy.md`). Isso garante que o agente despachado obedeça a todas as regras comportamentais, proibições de ferramentas e fluxos de trabalho definidos em sua persona.

### Envelope de Resposta (agente despachado retorna isto)

```
## RESPOSTA: [NOME_DO_AGENTE]
### estado
[sucesso | erro]

### resumo
[Resumo legível de 2-3 frases para o usuário]

### dados
[Resultados como listas numeradas com pares chave-valor. Sem tabelas markdown.]

### erros
[Apenas se estado for erro: o que deu errado]
```

### Handoff do Alura

**Contexto passado:** assunto especificado pelo usuário. Se o assunto não tiver sido informado, o Alura reporta um erro e pede ao usuário para informar o assunto primeiro.

**Formato de dados da resposta:**
```
1. nome_curso: [título do curso]
   duracao: [ex: 20 horas]
   nivel: [iniciante | intermediario | avancado]
   link: [URL]

2. [próximo curso no mesmo formato]
```

### Handoff do Udemy

**Contexto passado:** assunto especificado pelo usuário. Se o assunto não tiver sido informado, o Udemy reporta um erro e pede ao usuário para informar o assunto primeiro.

**Formato de dados da resposta:**
```
1. nome_curso: [título do curso]
   duracao: [ex: 20 horas]
   nivel: [iniciante | intermediario | avancado]
   preco: [ex: R$ 100,00]
   link: [URL]

2. [próximo curso no mesmo formato]
```
