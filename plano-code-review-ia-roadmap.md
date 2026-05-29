# Plano de Monetizacao Tech - Code Review IA-Augmented
## 27/05/2026 -> 31/12/2026 | Backend Java + IA aplicada em produto real

> **Meta:** R$ 5k/mes recorrente com programacao ate 31 de dezembro de 2026.
> **Projeto-ancora:** plataforma de code review automatizado para PRs do GitHub usando GitHub App, webhooks, Spring Boot e LLM.
> **Posicionamento:** Desenvolvedor Backend Java que sabe integrar IA em fluxo real de desenvolvimento, nao apenas usar IA como ferramenta pessoal.
> **Principio:** todo dia precisa deixar um rastro pequeno: commit, aplicacao, estudo, README, teste, issue fechada ou contato feito.

---

## Resumo executivo

O plano original tinha YorCookie como projeto-ancora. A partir daqui, o projeto principal passa a ser uma **plataforma de code review IA-augmented para Pull Requests do GitHub**.

O objetivo e construir uma demo real, usavel e defendivel em entrevista:

> "Eu usava Claude/GPT manualmente para revisar meus PRs. Automatizei esse fluxo com um GitHub App: quando abro um PR, o sistema recebe o webhook, busca o diff, chama uma LLM com contexto controlado e comenta no PR com sugestoes tecnicas."

Esse projeto conversa diretamente com:

- Backend Java moderno
- Spring Boot 3
- Spring Security
- GitHub Apps
- Webhooks
- Processamento assincrono
- LLM APIs
- Cache/idempotencia
- Multi-tenancy basica
- Observabilidade
- CI/CD
- IA aplicada em produto real

YorCookie continua podendo existir em paralelo como negocio, mas deixa de ser o principal argumento tecnico para entrevista.

---

## Nome de trabalho do projeto

**ReviewPilot AI**

Alternativas:

- PR Review Bot
- CodeReview AI
- PullRequest Copilot
- ReviewGuard

Use **ReviewPilot AI** enquanto o nome definitivo nao for importante. Nome nao pode travar execucao.

---

## Escopo do MVP

### Versao 0.1 - MVP util

Objetivo: funcionar em 1 repositorio seu e comentar em PRs reais.

Entra:

- GitHub App privado instalado em 1 repo
- Endpoint de webhook no Spring Boot
- Validacao de assinatura HMAC do GitHub
- Processamento de evento `pull_request.opened` e `pull_request.synchronize`
- Busca do diff do PR pela GitHub API
- Chunking simples do diff por arquivo
- Chamada para uma LLM barata
- Comentario unico no PR com resumo + achados prioritarios
- Cache para nao revisar o mesmo `repo + pr + commit_sha` duas vezes
- Postgres
- Docker Compose local
- README forte com arquitetura, prints e demo

Nao entra no MVP:

- Multi-tenancy completa
- Dashboard web
- Billing
- Marketplace publico
- Suporte a varias LLMs
- Comentarios inline linha a linha
- Analise profunda de seguranca

### Versao 0.2 - Diferencial tecnico

Entra depois do MVP:

- Fila com RabbitMQ ou Kafka
- Retry com backoff
- Rate limit por instalacao
- Review por severidade: `critical`, `major`, `minor`, `nit`
- Configuracao por repo via arquivo `.reviewpilot.yml`
- Comentarios inline quando a linha do diff permitir
- Tela simples de historico de reviews

### Evolucao de mensageria

Ordem correta para nao explodir escopo:

1. MVP com processamento assincrono simples:
   - Webhook valida assinatura
   - Salva evento no banco
   - Retorna `202 Accepted`
   - Dispara processamento em background com service separado ou Spring `@Async`
   - Usa idempotencia por `repository + pull_request_number + head_sha`

2. Depois do bot comentar PR real:
   - Introduzir RabbitMQ ou Kafka
   - Webhook vira producer de mensagens
   - Worker/consumer processa review
   - Adicionar retry com backoff
   - Adicionar dead letter queue
   - Controlar rate limit por repo ou installation

Ganho arquitetural:

- Evita timeout no webhook do GitHub
- Separa recebimento de evento e processamento pesado
- Permite retry quando GitHub API ou LLM falham
- Facilita escalar consumers no futuro
- Mantem o fluxo mais resiliente sem bloquear a entrega do MVP

### Versao 1.0 - Portfolio forte

Entra ate outubro:

- GitHub App instalavel em mais de um repo
- Multi-tenancy por instalacao do GitHub App
- Auth administrativa
- Observabilidade com logs estruturados + metricas
- Deploy em producao
- CI/CD
- Testes >= 70%
- Demo publica documentada

---

## Arquitetura do MVP

```mermaid
flowchart LR
  A["GitHub PR aberto/atualizado"] --> B["GitHub Webhook"]
  B --> C["Spring Boot API"]
  C --> D["Valida assinatura HMAC"]
  D --> E["Salva evento no Postgres"]
  E --> F["Worker de review"]
  F --> G["GitHub API: busca diff"]
  G --> H["Chunking do diff"]
  H --> I["LLM API"]
  I --> J["Normaliza achados"]
  J --> K["GitHub API: comenta no PR"]
  K --> L["Cache por commit SHA"]
```

### Componentes

| Componente | Responsabilidade |
|---|---|
| Webhook Controller | Receber eventos do GitHub |
| Signature Verifier | Validar `X-Hub-Signature-256` |
| Event Service | Persistir evento bruto e metadados |
| Review Worker | Processar evento de forma assincrona |
| GitHub Client | Buscar PR, diff e comentar |
| Diff Chunker | Dividir diff em blocos que cabem no contexto |
| LLM Client | Chamar OpenAI, Anthropic ou Gemini |
| Review Formatter | Transformar resposta em comentario legivel |
| Review Cache | Evitar revisar o mesmo commit duas vezes |

---

## Stack recomendada

### Backend

- Java 21
- Spring Boot 3.3+
- Spring Web
- Spring Security
- Spring Data JPA
- PostgreSQL
- Flyway
- Docker / Docker Compose
- Gradle ou Maven
- Testcontainers
- JUnit 5
- Mockito

### Integracoes

- GitHub App
- GitHub REST API
- Webhooks do GitHub
- LLM API:
  - Opcao barata: Gemini Flash
  - Opcao portfolio: OpenAI GPT-4.1 mini ou equivalente atual
  - Opcao Claude: Haiku/Sonnet conforme custo

### Deploy

- Railway, Render ou Fly.io
- Postgres gerenciado
- GitHub Actions para CI/CD

### Observabilidade

- Logs estruturados
- Actuator
- Micrometer
- Prometheus/Grafana basico depois do MVP

---

## Modelo de dados inicial

```text
github_installations
- id
- github_installation_id
- account_login
- account_type
- created_at
- updated_at

repositories
- id
- installation_id
- github_repo_id
- owner
- name
- full_name
- default_branch
- created_at
- updated_at

pull_requests
- id
- repository_id
- github_pr_number
- title
- author_login
- head_sha
- base_sha
- state
- created_at
- updated_at

webhook_events
- id
- github_delivery_id
- event_type
- action
- payload_json
- received_at
- processed_at
- status
- error_message

review_runs
- id
- repository_id
- pull_request_id
- head_sha
- provider
- model
- status
- total_files
- total_tokens_estimated
- summary
- github_comment_id
- created_at
- completed_at

review_findings
- id
- review_run_id
- file_path
- severity
- category
- title
- body
- suggested_fix
- created_at
```

### Regra de idempotencia

Uma review e unica por:

```text
repository_id + pull_request_id + head_sha
```

Se o mesmo commit chegar de novo pelo webhook, o sistema nao chama a LLM novamente.

---

## Endpoints REST

### Publicos para GitHub

```http
POST /api/webhooks/github
```

Responsabilidade:

- Validar assinatura
- Validar evento suportado
- Persistir payload
- Disparar processamento
- Retornar `202 Accepted`

### Health e operacao

```http
GET /actuator/health
GET /actuator/info
```

### Admin simples para portfolio

```http
GET /api/admin/review-runs
GET /api/admin/review-runs/{id}
GET /api/admin/repositories
```

No MVP, esses endpoints podem ser protegidos por Basic Auth ou JWT simples. Nao gaste energia com auth sofisticada antes do bot funcionar.

---

## Fluxo GitHub App

1. Criar GitHub App privado
2. Configurar webhook URL
3. Configurar webhook secret
4. Permissoes minimas:
   - Pull requests: read
   - Contents: read
   - Metadata: read
   - Issues: write ou Pull requests: write, conforme estrategia de comentario
5. Eventos:
   - `pull_request`
6. Instalar app em 1 repo seu
7. Abrir PR demo
8. Confirmar que o bot comenta

### Eventos suportados no MVP

- `pull_request.opened`
- `pull_request.synchronize`
- `pull_request.reopened`

Ignorar:

- `closed`
- `labeled`
- `assigned`
- `edited`

---

## Fluxo LLM

### Entrada para a LLM

- Nome do repo
- Titulo do PR
- Descricao do PR
- Lista de arquivos alterados
- Diff chunkado
- Regras de review

### Prompt base

```text
Voce e um revisor senior de backend Java/Spring.
Revise o diff abaixo procurando bugs reais, problemas de seguranca,
regressoes, testes ausentes e inconsistencias de arquitetura.

Priorize achados acionaveis. Nao comente estilo superficial.
Retorne no maximo 8 achados.

Formato:
- severidade: critical|major|minor|nit
- arquivo
- titulo
- explicacao
- sugestao de correcao
```

### Saida esperada

No MVP, peca JSON para facilitar parse. Depois formate em Markdown para comentar no PR.

### Controle de custo

- Limite maximo de arquivos por review no MVP: 20
- Limite maximo de caracteres por arquivo: 12.000
- Ignorar arquivos:
  - lockfiles
  - imagens
  - build output
  - `.min.js`
  - arquivos muito grandes
- Cache por `head_sha`
- Variavel `LLM_ENABLED=false` para testes locais sem gastar API

---

## Comentario no Pull Request

Formato sugerido:

```markdown
## ReviewPilot AI

Resumo: encontrei 3 pontos que merecem revisao antes do merge.

### Major

1. `src/main/java/.../OrderService.java`
   A validacao permite criar pedido sem usuario autenticado em um fluxo especifico.
   Sugestao: mover a validacao para antes da chamada ao repositorio.

### Minor

2. `src/test/java/.../OrderServiceTest.java`
   Falta teste cobrindo erro de permissao.

---

Revisado em commit `abc123`.
```

---

## Dedicacao recomendada

### Carga minima sustentavel

| Periodo | Horas |
|---|---:|
| Segunda a sexta | 2h/dia |
| Sabado | 4h |
| Domingo | 1h de revisao |
| Total semanal | 15h |
| Total mensal medio | 60h |

### Carga ideal sem quebrar rotina

| Periodo | Horas |
|---|---:|
| Segunda a sexta | 2h30/dia |
| Sabado | 5h |
| Domingo | 1h30 |
| Total semanal | 19h |
| Total mensal medio | 75h |

### Regra dos dias ruins

Se o dia estiver ruim, faca o minimo de 25 minutos:

- escrever 1 teste
- melhorar 1 trecho do README
- fechar 1 TODO
- ler 1 pagina da doc do GitHub
- aplicar em 1 vaga
- mandar 1 mensagem fria

O plano nao depende de dias perfeitos. Depende de nao zerar.

---

## Rotina diaria fixa

### Segunda a sexta - 2h

| Bloco | Tempo | Acao |
|---|---:|---|
| Bloco 1 | 25 min | Aplicacao, networking ou follow-up |
| Bloco 2 | 75 min | Desenvolvimento do ReviewPilot AI |
| Bloco 3 | 20 min | Estudo tecnico ligado ao que esta implementando |

### Sabado - 4h

| Bloco | Tempo | Acao |
|---|---:|---|
| Bloco 1 | 2h | Feature maior |
| Bloco 2 | 1h | Testes/refatoracao |
| Bloco 3 | 1h | README, prints, LinkedIn ou deploy |

### Domingo - 1h

| Bloco | Tempo | Acao |
|---|---:|---|
| Revisao | 30 min | Atualizar metricas |
| Planejamento | 20 min | Escolher tarefas da semana |
| Limpeza | 10 min | Fechar ou reorganizar issues |

---

## Regra de progresso diario

Todo dia precisa terminar com pelo menos 1 destes resultados:

- 1 commit
- 1 teste novo
- 1 issue fechada
- 1 endpoint implementado
- 1 trecho do README melhorado
- 1 aplicacao enviada
- 1 contato profissional feito
- 1 decisao tecnica documentada
- 1 bug corrigido
- 1 print/demo salvo

Se nao virou evidencia, provavelmente foi so intencao.

---

## Visao geral em 4 fases

| Fase | Janela | Receita-alvo tech | Foco |
|---|---|---:|---|
| 1. Fundacao | 27/05 -> 30/06 | R$ 400-700 bolsa | Ativos, vagas, GitHub App, webhook local |
| 2. MVP + caca dupla | 01/07 -> 31/08 | R$ 2-5k oferta | Bot comentando PR real + aplicacoes estagio/junior |
| 3. Producao + reputacao | 01/09 -> 31/10 | R$ 3-5k | Deploy, fila, testes, post tecnico forte |
| 4. Conversao | 01/11 -> 31/12 | R$ 5k+ recorrente | Oferta, efetivacao, freela ou combinacao |

---

## FASE 1 - Fundacao
### 27/05 -> 30/06

Objetivo: reduzir pressao financeira, preparar ativos e iniciar o ReviewPilot AI com um primeiro fluxo tecnico real.

### Semana 1 - 27/05 -> 01/06

**Meta da semana:** plano fechado, vagas iniciadas e repo do ReviewPilot criado.

| Dia | Tech | Carreira |
|---|---|---|
| Qua 27/05 | Definir escopo v0.1 e nome do repo | Email para professor Renato sobre bolsa |
| Qui 28/05 | Criar repo + README inicial + roadmap | Aplicar em 5 vagas |
| Sex 29/05 | Estudar GitHub App docs e webhooks | Ajustar LinkedIn headline |
| Sab 30/05 | Criar projeto Spring Boot + Docker Compose Postgres | Revisar curriculo PT-BR e EN-US |
| Dom 31/05 | Criar issues do MVP no GitHub | Revisao semanal de 15 min |
| Seg 01/06 | Implementar healthcheck + estrutura inicial | Lista de 20 vagas-alvo |

Entregaveis:

- Repo publico criado
- README inicial
- Issues do MVP
- Spring Boot rodando local
- Docker Compose com Postgres
- 5 aplicacoes feitas

### Semana 2 - 02/06 -> 08/06

**Meta da semana:** webhook local recebendo payload do GitHub.

Tech:

- Criar endpoint `POST /api/webhooks/github`
- Configurar ngrok ou alternativa para teste local
- Criar GitHub App privado
- Receber primeiro payload real
- Persistir payload bruto em `webhook_events`
- Criar testes do controller

Carreira:

- 10 aplicacoes novas
- Follow-up Renato se necessario
- Mensagem para coordenador do Sobrevidas sobre bolsa
- Post LinkedIn: "O que aprendi corrigindo issues OWASP no SonarQube em projeto Java real"

Horas-alvo:

- 15h total
- 9h projeto
- 3h aplicacoes/networking
- 3h estudo Spring Security/GitHub App

### Semana 3 - 09/06 -> 15/06

**Meta da semana:** validar assinatura HMAC e filtrar eventos.

Tech:

- Implementar validacao `X-Hub-Signature-256`
- Rejeitar assinatura invalida com `401`
- Aceitar apenas eventos `pull_request`
- Filtrar actions suportadas
- Modelar `repositories` e `pull_requests`
- Salvar metadados do PR

Carreira:

- 10 aplicacoes novas
- 3 mensagens frias para devs seniors
- Simulacao de entrevista 1h

Entregavel tecnico:

- Webhook seguro e persistente

### Semana 4 - 16/06 -> 22/06

**Meta da semana:** buscar diff do PR pela GitHub API.

Tech:

- Gerar JWT do GitHub App
- Trocar por installation access token
- Criar `GitHubClient`
- Buscar arquivos alterados do PR
- Buscar diff/patch por arquivo
- Criar testes unitarios do client com mocks

Carreira:

- 10 aplicacoes novas
- Atualizar README com diagrama da arquitetura
- Avaliar respostas, entrevistas e bolsa

Entregavel tecnico:

- Dado um PR real, o backend consegue buscar arquivos modificados

### Semana 5 - 23/06 -> 30/06

**Meta da semana:** preparar LLM e cache sem ainda depender de producao.

Tech:

- Criar `DiffChunker`
- Ignorar arquivos irrelevantes
- Criar `ReviewRun`
- Implementar idempotencia por `repo + pr + head_sha`
- Criar `LLMClient` com modo fake para testes
- Gerar review fake e salvar no banco

Carreira:

- 10 aplicacoes novas
- 3 follow-ups
- Atualizar curriculo com GitHub App/Webhooks em "projeto em desenvolvimento"
- Post curto mostrando arquitetura inicial do ReviewPilot AI

**Marco da Fase 1:** webhook seguro + GitHub App privado + diff de PR sendo buscado + pipeline fake de review salvo no banco.

---

## FASE 2 - MVP + caca dupla
### 01/07 -> 31/08

Objetivo: ter um bot que comenta PR real e usar isso como arma nas aplicacoes para estagio e junior.

### Julho - bot comentando PR real

#### Semana 6 - 01/07 -> 07/07

Tech:

- Integrar LLM real
- Criar prompt v1
- Receber JSON da LLM
- Tratar erro de parse
- Criar limite de custo por review

Carreira:

- 10-15 aplicacoes
- 3 mensagens frias
- 1 mock interview

#### Semana 7 - 08/07 -> 14/07

Tech:

- Formatar comentario Markdown
- Postar comentario no PR via GitHub API
- Criar PR demo no proprio repo
- Ajustar permissoes do GitHub App

Entregavel:

- Primeiro comentario real do bot em um PR

#### Semana 8 - 15/07 -> 21/07

Tech:

- Melhorar severidades
- Adicionar categorias: bug, security, test, architecture, maintainability
- Criar testes de formatter
- Adicionar exemplos no README

Carreira:

- 10-15 aplicacoes
- Atualizar curriculo com "GitHub App + LLM-powered PR reviews"

#### Semana 9 - 22/07 -> 31/07

Tech:

- Deploy em staging
- Configurar variaveis de ambiente
- Criar GitHub Actions CI
- Rodar testes no CI
- Abrir PR demo gravavel em video/GIF

Post LinkedIn:

- "Automatizei meu code review com um GitHub App + IA: arquitetura do MVP"

**Marco de Julho:** bot comentando PR real, com staging e README demonstravel.

### Agosto - robustez e entrevistas

#### Semana 10 - 01/08 -> 07/08

Tech:

- Adicionar fila simples
  - Opcao pragmatica: Spring `@Async` + tabela de status
  - Opcao portfolio: RabbitMQ
- Implementar retry
- Melhorar logs

Carreira:

- 10-15 aplicacoes
- 1 mock interview focada em projeto

#### Semana 11 - 08/08 -> 14/08

Tech:

- Configuracao por repo via `.reviewpilot.yml`
- Permitir ignorar paths
- Permitir limitar severidade minima

#### Semana 12 - 15/08 -> 21/08

Tech:

- Testcontainers com Postgres
- Cobertura de testes >= 50%
- Testes para webhook, assinatura, idempotencia e chunking

#### Semana 13 - 22/08 -> 31/08

Tech:

- Hardening do MVP
- README completo
- Prints do PR comentado
- Video curto de demo
- Checklist de custos

Carreira:

- 10-15 aplicacoes
- Follow-ups
- Reavaliar taxa de resposta

**Marco da Fase 2:** pelo menos 1 oferta aceita ou entrevistas em andamento; ReviewPilot AI funcionando em staging e comentando PRs reais.

---

## FASE 3 - Producao + reputacao
### 01/09 -> 31/10

Objetivo: transformar MVP em projeto de portfolio forte, com producao, testes e narrativa tecnica.

### Setembro - producao e diferencial tecnico

Tech:

- Deploy de producao
- Banco Postgres gerenciado
- Variaveis seguras
- Observabilidade basica
- Actuator configurado
- Logs com correlation id por webhook delivery
- RabbitMQ ou Kafka se ainda nao tiver fila real
- Cobertura de testes >= 70%

Carreira:

- Se ja tiver renda: reduzir aplicacoes e focar qualidade
- Se nao tiver oferta: manter 15 aplicacoes por semana
- Conversar com Renato sobre uso no Sobrevidas

Post LinkedIn:

- "Como meu bot de code review processa webhooks do GitHub com seguranca e idempotencia"

### Outubro - maturidade de produto

Tech:

- Instalar em mais de um repo seu
- Adicionar tela admin simples ou endpoints admin documentados
- Melhorar comentarios inline se viavel
- Criar documentacao de arquitetura
- Criar ADRs:
  - Por que GitHub App e nao OAuth simples
  - Como controlei custo de LLM
  - Como garanti idempotencia
  - Como tratei limite de contexto

Carreira:

- Post-ancora:
  - "Construindo um GitHub App com Spring Boot e IA para revisar Pull Requests"
- Usar demo em entrevistas
- Pedir feedback de 2 devs seniors

**Marco da Fase 3:** projeto em producao, README forte, testes >= 70%, demo pronta para entrevista.

---

## FASE 4 - Conversao
### 01/11 -> 31/12

Objetivo: converter portfolio e entrevistas em renda recorrente de R$ 5k/mes.

### Novembro

Se ja tiver oferta junior:

- Entregar bem no trabalho
- Manter ReviewPilot AI vivo com 1 melhoria por semana
- Nao trocar por ansiedade se o aprendizado estiver bom

Se tiver estagio:

- Conversa formal sobre efetivacao
- Pedir feedback escrito
- Ativar freela leve se necessario

Se nao tiver oferta:

- Plano B agressivo:
  - 30 aplicacoes no mes
  - 3 mock interviews por semana
  - Reescrever CV e LinkedIn
  - Usar demo do ReviewPilot em toda abordagem
  - Pedir indicacao direta para professores/devs

Post LinkedIn:

- "O que aprendi integrando LLM em um fluxo real de Pull Request"

### Dezembro

Objetivos:

- R$ 5k/mes recorrente atingido por:
  - junior
  - estagio + freela
  - bolsa + freela
  - PJ inicial
- ReviewPilot AI com:
  - 20+ PRs revisados
  - 2+ repos instalados
  - README com metricas reais
  - custos documentados
  - roadmap 2027

Post final:

- "Meu plano de 7 meses para sair de portfolio generico para backend Java com IA aplicada"

---

## Roadmap tecnico detalhado

| Marco | Prazo | Entregavel |
|---|---|---|
| Repo + README + issues | 01/06 | Projeto publico organizado |
| Spring Boot + Postgres | 08/06 | API local com healthcheck |
| Webhook GitHub | 15/06 | Recebe payload real |
| Assinatura HMAC | 22/06 | Webhook seguro |
| GitHub API client | 30/06 | Busca diff de PR |
| LLM fake + cache | 30/06 | Pipeline sem custo |
| LLM real | 07/07 | Review gerada por IA |
| Comentario no PR | 14/07 | Bot comentando PR real |
| Staging | 31/07 | Demo acessivel |
| Assincrono simples | 07/08 | Webhook retorna rapido e review roda em background com service separado ou `@Async` |
| Fila/retry | 15/08 | RabbitMQ ou Kafka com retry, backoff e base para DLQ |
| Testcontainers | 21/08 | Testes de integracao |
| README completo | 31/08 | Portfolio apresentavel |
| Producao | 15/09 | App rodando publicamente |
| Observabilidade | 30/09 | Logs/metricas basicas |
| Testes >= 70% | 15/10 | Qualidade defensavel |
| Multi-repo | 31/10 | Instala em 2+ repos |
| Demo entrevista | 05/11 | Video/GIF + PR real |
| 20+ PRs revisados | 31/12 | Metrica real no README |

---

## Planejamento mensal de horas

| Mes | Horas alvo | Projeto | Carreira | Estudo | Entregavel principal |
|---|---:|---:|---:|---:|---|
| Junho | 60h | 36h | 15h | 9h | Webhook + GitHub API |
| Julho | 75h | 45h | 20h | 10h | Bot comenta PR real |
| Agosto | 75h | 42h | 23h | 10h | MVP robusto + staging |
| Setembro | 65h | 42h | 13h | 10h | Producao + fila |
| Outubro | 65h | 40h | 15h | 10h | Testes 70% + post forte |
| Novembro | 55h | 25h | 25h | 5h | Conversao em oferta/freela |
| Dezembro | 45h | 20h | 20h | 5h | Meta R$ 5k + retrospectiva |

Total estimado: **440 horas** ate 31/12.

Isso e suficiente para um projeto forte se o escopo ficar controlado.

---

## Checklist semanal fixo

Toda semana precisa ter:

- 10-15 aplicacoes ou follow-ups
- 3-5 mensagens frias
- 1 mock interview
- 1 melhoria visivel no ReviewPilot AI
- 1 bloco de testes
- 1 atualizacao no README ou docs
- 1 revisao de metricas no domingo

---

## Metricas de acompanhamento

| Semana | Horas projeto | Commits | Testes novos | Aplicacoes | Respostas | Entrevistas | PRs revisados | Custo LLM |
|---|---:|---:|---:|---:|---:|---:|---:|---:|
| W1 27/05-01/06 | | | | | | | | |
| W2 02/06-08/06 | | | | | | | | |
| W3 09/06-15/06 | | | | | | | | |
| W4 16/06-22/06 | | | | | | | | |
| W5 23/06-30/06 | | | | | | | | |

---

## Curriculo - nova narrativa

### Resumo profissional

Comecar com:

> Desenvolvedor Backend Java com experiencia em Spring Boot, PostgreSQL, testes e integracao de IA em fluxos reais de desenvolvimento. Estudante de Engenharia de Software na UFG, atuo no projeto Sobrevidas e construo o ReviewPilot AI, um GitHub App que automatiza code review de Pull Requests usando LLMs.

### Projeto no curriculo

**ReviewPilot AI - Plataforma de code review automatizado para GitHub PRs**

- Desenvolvi um GitHub App em Java 21 e Spring Boot que recebe webhooks de Pull Requests, valida assinatura HMAC, busca diffs pela GitHub API e gera revisoes automatizadas com LLM.
- Implementei idempotencia por commit SHA para evitar custo duplicado de IA.
- Modelei pipeline de processamento com persistencia em PostgreSQL, cache de reviews e logs estruturados.
- Configurei CI/CD com GitHub Actions, Docker e deploy em ambiente cloud.

Quando tiver numeros, adicionar:

- Revisou X Pull Requests reais em Y repositorios.
- Reduziu tempo manual de review em X%.
- Manteve custo medio por review abaixo de R$ X.

---

## README que impressiona recrutador

Estrutura recomendada:

```markdown
# ReviewPilot AI

GitHub App that reviews Pull Requests using Java, Spring Boot and LLMs.

## Demo
GIF or screenshot of a real PR comment.

## Why I built this
I was manually pasting diffs into LLMs before opening PRs. I automated the workflow.

## Architecture
Diagram.

## Features
- GitHub webhook signature validation
- Pull Request diff fetching
- LLM-based review generation
- Idempotency by commit SHA
- PostgreSQL persistence
- Dockerized local environment
- CI/CD

## Tech Stack
Java 21, Spring Boot 3, PostgreSQL, Docker, GitHub Apps, LLM API.

## How it works
1. GitHub sends webhook
2. API validates signature
3. Worker fetches diff
4. Diff is chunked
5. LLM reviews
6. Bot comments on PR

## Running locally
3 commands max.

## Testing
Coverage badge and command.

## Roadmap
MVP, v0.2, v1.0.

## Lessons learned
Context limits, cost control, webhook security, idempotency.
```

---

## Posts-ancora LinkedIn

### Post 1 - Junho

Tema:

> "Comecei a construir um GitHub App em Java para automatizar code review com IA"

Foco:

- Problema pessoal
- Arquitetura inicial
- Webhooks
- Por que GitHub App

### Post 2 - Julho

Tema:

> "Meu bot de IA comentou o primeiro Pull Request real"

Foco:

- Demo
- Diff -> chunking -> LLM -> comentario
- Custo e limites
- Aprendizado tecnico

### Post 3 - Setembro

Tema:

> "Como tratei idempotencia, retry e custo em uma integracao com LLM"

Foco:

- Nao revisar mesmo commit duas vezes
- Controle de tokens
- Falhas da API
- Logs e observabilidade

### Post 4 - Outubro/Novembro

Tema:

> "Construindo IA aplicada de verdade: o que aprendi criando um GitHub App com Spring Boot"

Foco:

- Arquitetura completa
- Trade-offs
- Prints
- Repos reais
- Proximos passos

---

## Aplicacoes e carreira

### Cadencia ate conseguir oferta

- 10-15 aplicacoes por semana
- 3-5 mensagens frias por semana
- 1 mock interview por semana
- 1 revisao de curriculo por mes
- Usar o ReviewPilot AI como gancho em toda conversa

### Frase para abordagem

> Estou construindo um GitHub App em Java/Spring que revisa Pull Requests automaticamente usando IA. Ele recebe webhooks do GitHub, valida assinatura, busca o diff e comenta no PR com sugestoes tecnicas. Estou usando isso como projeto de portfolio enquanto busco oportunidade de estagio ou junior backend Java.

### Empresas-alvo

Estagio:

- Invent Software
- CI&T
- TOTVS
- Compass.UOL
- Stefanini
- BRQ
- Itau Tech
- iFood
- Mercado Livre
- C6 Bank

Junior:

- Startups com Java/Spring
- Consultorias com squad Java
- Empresas com times internos de plataforma
- Empresas usando IA em produto
- Early career programs

---

## Alertas amarelos

### Fim de junho sem webhook funcionando

Problema: escopo ou fundamento GitHub App travou.

Acao:

- Cortar tudo que nao seja webhook + diff
- Nao implementar dashboard
- Nao implementar multi-tenancy
- Fazer versao local com 1 repo apenas

### Fim de julho sem bot comentando PR

Problema: integracao LLM ou GitHub API esta demorando demais.

Acao:

- Usar LLM fake por 2 dias para terminar fluxo
- Depois trocar por LLM real
- Postar comentario simples, mesmo sem inline comments

### Fim de agosto sem entrevistas

Problema: curriculo, LinkedIn ou abordagem.

Acao:

- Reescrever curriculo inteiro
- Gravar demo de 60 segundos
- Mandar abordagem com link direto para PR comentado
- Pedir review do Renato e de 2 devs seniors

### Fim de outubro sem renda tech

Problema: conversao de mercado.

Acao:

- Ativar freela via indicacao
- Oferecer 5h/semana de backend Java
- Usar ReviewPilot como prova de capacidade
- Aceitar PJ inicial se destravar renda e experiencia

---

## Plano B se 31/08 chegar sem oferta

Setembro vira mes agressivo:

1. 30 aplicacoes novas
2. 3 mock interviews por semana
3. Reescrita completa de CV e LinkedIn
4. Demo do ReviewPilot AI no topo do GitHub
5. Post LinkedIn com video do bot comentando PR
6. Conversa direta com Renato sobre indicacoes
7. Oferta de freela leve via rede pessoal

---

## Plano C se 31/10 chegar sem oferta

Pivota a meta para combinacao:

- Bolsa ou estagio: R$ 1.500-3.500
- Freela Java/Spring: R$ 1.500-3.000
- Pequenos servicos tecnicos via indicacao

Meta de R$ 5k pode bater por soma, nao precisa vir de uma unica CLT.

---

## Checklist diario rapido

Use todo dia antes de dormir:

```text
Hoje eu:
[ ] Fiz pelo menos 25 min de progresso?
[ ] Gerei alguma evidencia concreta?
[ ] Dei commit ou atualizei uma issue?
[ ] Fiz aplicacao, contato ou follow-up?
[ ] Sei qual e a proxima tarefa de amanha?
```

Se marcou menos de 3, o dia ficou fraco. No dia seguinte, faca a primeira tarefa antes de abrir rede social.

---

## Estado atual

Atualizado em 29/05/2026:

- Repo `reviewpilot-ai` criado e conectado ao GitHub
- Projeto Spring Boot criado com Java 21, Maven e Spring Boot 3.5.14
- Roadmap e checklist de progresso adicionados ao projeto
- 10 issues iniciais criadas no GitHub
- Issue 1 concluida: bootstrap do projeto Spring Boot
- Issue 2 concluida: PostgreSQL local com Docker Compose
- Issue 3 em andamento: configuracao de application profiles
- Aplicacao roda com profile `local`
- Testes rodam com profile `test` usando H2 em memoria
- `data.sql` removido para evitar inicializacao automatica indevida em testes
- `spring.sql.init.mode=never` configurado no profile de teste
- `./mvnw test` executado com sucesso
- `/actuator/health` retorna `UP`
- PR do setup local mergeado

---

## Proxima sessao

Amanha, 30/05/2026:

Objetivo principal: fechar a issue 3, `Configure application profiles`, se o PR ainda nao tiver sido mergeado.

1. Revisar o diff final da branch `chore/configure-application-profiles`
2. Confirmar que `./mvnw test` continua passando
3. Fazer commit das alteracoes da issue 3
4. Abrir PR
5. Mergear PR e fechar a issue 3

Se a issue 3 ja estiver fechada:

1. Criar branch para a issue 4: `Create GitHub webhook endpoint`
2. Definir contrato inicial do endpoint `POST /api/webhooks/github`
3. Implementar o controller retornando `202 Accepted`
4. Criar primeiro teste do controller

Se sobrar energia:

- Aplicar em 5 vagas
- Registrar as aplicacoes no checklist
- Ler a documentacao de webhooks do GitHub App

---

## Recado final

Esse projeto nao e "mais um CRUD". Ele prova uma tese clara:

> sou backend Java, entendo fluxo real de engenharia e sei colocar IA dentro de um produto funcional.

Esse e o tipo de historia que diferencia junior forte de junior generico.
