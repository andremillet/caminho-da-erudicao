# Visão Geral da Arquitetura

Documento **agnóstico de stack**. Define os componentes lógicos, responsabilidades e fluxos principais da aplicação. As decisões de tecnologia (linguagem, framework, banco) ficam em [`tecnologias.md`](tecnologias.md).

## Componentes Lógicos

```
┌──────────────────────────────────────────────────────────────────┐
│                           Cliente (UI)                           │
│  Aplicação web/mobile consumindo a API REST.                    │
│  Renderiza questões, perfil, ranking, loja de recompensas.      │
└──────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌──────────────────────────────────────────────────────────────────┐
│                          API (Backend)                           │
│  Autenticação · Questões · Tentativas · Ranking · Recompensas    │
└──────────────────────────────────────────────────────────────────┘
                                │
        ┌───────────────────────┼───────────────────────┐
        ▼                       ▼                       ▼
┌────────────────┐    ┌──────────────────┐    ┌──────────────────┐
│ Banco de Dados │    │ Serviço de Rank. │    │ Serviço de       │
│   Relacional   │    │ (periódico)      │    │ Recompensas      │
│  (usuários,    │    │ Recalcula lead.  │    │ Verifica gatilhos│
│   questões,    │    │ por período.     │    │ de badges e      │
│   tentativas)  │    │                  │    │ distribuição de  │
│                │    │                  │    │ moedas.          │
└────────────────┘    └──────────────────┘    └──────────────────┘
```

## Camadas de Responsabilidade

| Camada | Responsabilidades |
|---|---|
| **Cliente (UI)** | Apresentar questões, registrar respostas localmente (com sincronização), exibir ranking, perfil e loja. Pode cachear questões. |
| **API** | Autenticar, validar, servir questões, registrar tentativas, calcular XP, atualizar ranking, conceder badges e moedas. |
| **Banco de Dados** | Persistir usuários, questões, alternativas, tentativas, badges, compras, saldos. |
| **Serviço de Ranking** | Job periódico (cron) que recalcula leaderboards por período (diário, semanal, mensal, all-time) e por segmento (global, por ano, por unidade). |
| **Serviço de Recompensas** | Avalia eventos (`tentativa_criada`, `meta_bimestral_concluida`, etc.) e dispara concessão de badges/moedas. |

## Fluxos Principais

### 1. Cadastro e Login
```
Cliente → POST /auth/register → API → valida e cria Usuario → retorna token
Cliente → POST /auth/login    → API → valida credenciais → retorna token
```

### 2. Responder uma Questão
```
Cliente → GET  /questoes?ano=7&unidade=algebra   → API → retorna lista de questões
Cliente → POST /tentativas { questao_id, alternativa } → API
                                                      → registra tentativa
                                                      → calcula XP (acerto × dificuldade)
                                                      → soma moedas (se acertou)
                                                      → dispara evento para recompensas
                                                      → atualiza progresso do usuário
                                                      → retorna { correta, xp_ganho, moedas_ganhas, novas_badges[] }
```

### 3. Consultar Ranking
```
Cliente → GET /rankings/tipo/{global|ano|unidade}?periodo={semanal|mensal|alltime}
       → API → consulta view materializada de ranking → retorna top N + posição do usuário
```

### 4. Comprar Recompensa na Loja
```
Cliente → GET  /loja/recompensas                    → API → lista disponível
Cliente → POST /loja/comprar { recompensa_id }     → API → valida saldo
                                                       → desconta moedas
                                                       → concede item/benefício
                                                       → registra compra
                                                       → retorna { saldo_atual, item_adquirido }
```

### 5. Conquista de Badge (assíncrona)
```
Tentativa registrada → evento publicado
                       → Serviço de Recompensas avalia critérios
                          → se satisfeito, cria UsuarioBadge e soma bônus de XP/moedas
                          → notifica cliente (push/email ou polling)
```

## Princípios de Design

- **Questões imutáveis** após publicação (correções geram nova versão).
- **Tentativas são append-only** — registros históricos nunca são alterados.
- **XP e moedas derivam de tentativas** — não há campo editável manualmente pelo usuário.
- **Rankings são derivados** — recalculáveis a partir das tentativas, sem perda de histórico.
- **Badges são automáticas e idempotentes** — ganhar a mesma badge 2× não gera duplicidade.
- **BNCC como taxonomia** — toda questão carrega `ano`, `unidade_tematica`, `objeto_conhecimento`, `habilidade_bncc`.

## Requisitos Não-Funcionais (sugestões)

| Requisito | Meta |
|---|---|
| Latência (responder questão) | < 300 ms p95 |
| Disponibilidade | 99,5% |
| Suporte multiusuário | 10k+ usuários ativos simultâneos |
| Auditoria | Log de tentativas por 1 ano |
| Privacidade | LGPD — dados mínimos necessários |
| Internacionalização | PT-BR (outras línguas desacopladas via i18n) |
