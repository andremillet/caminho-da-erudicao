# Modelo de Dados

Esquema conceitual (agnóstico de banco) das principais entidades da aplicação.

## Diagrama Entidade-Relacionamento (resumido)

```
Usuario (1) ──< (N) Tentativa >── (1) Questao
   │                                  │
   │                                  └─< QuestaoAlternativa (1..N)
   │
   ├──< (N) UsuarioBadge >── (1) Badge
   ├──< (N) UsuarioRecompensa >── (1) Recompensa
   └──< (N) MovimentacaoMoeda (evento de ganho/gasto)

Questao ── Tag (N..N) ── Tag
Ranking (view materializada, derivada de Tentativa)
```

## Entidades

### Usuario
| Campo | Tipo | Descrição |
|---|---|---|
| `id` | UUID | Identificador único. |
| `nome` | string | Nome de exibição. |
| `email` | string | Login único. |
| `senha_hash` | string | Hash da senha (nunca a senha em texto puro). |
| `ano_escolar` | enum (6, 7, 8, 9) | Ano em que o aluno está matriculado. |
| `xp_total` | int | Pontuação acumulada (somatório das tentativas). |
| `moedas_saldo` | int | Moedas virtuais disponíveis. |
| `nivel` | int | Nível atual do aluno (função de `xp_total`). |
| `criado_em` | timestamp | |
| `ultimo_acesso_em` | timestamp | Para streaks e engajamento. |
| `papel` | enum (aluno, professor, admin) | Distingue perfis. |

### Questao
| Campo | Tipo | Descrição |
|---|---|---|
| `id` | UUID | Identificador único. |
| `enunciado` | text | Texto da questão. Pode incluir imagens (markdown/HTML restrito). |
| `ano` | enum (6, 7, 8, 9) | Ano escolar. |
| `unidade_tematica` | enum (numeros, algebra, geometria, grandezas_medidas, prob_estat) | Unidade temática BNCC. |
| `objeto_conhecimento` | string | Ex.: "frações", "teorema de pitágoras". |
| `habilidade_bncc` | string | Código oficial, ex.: `EF07MA13`. |
| `dificuldade` | int (1-5) | Curva de dificuldade. |
| `pontuacao_base` | int | XP base em caso de acerto. |
| `explicacao` | text | Explicação da resposta (exibida após tentativa). |
| `tags` | array<string> | Tags livres para filtros. |
| `criado_em` | timestamp | |
| `publicada` | bool | Se está visível aos usuários. |
| `versao` | int | Versão da questão (imutável após publicação). |

### QuestaoAlternativa
| Campo | Tipo | Descrição |
|---|---|---|
| `id` | UUID | |
| `questao_id` | UUID (FK) | |
| `texto` | text | Texto da alternativa. |
| `correta` | bool | Apenas uma deve ser `true`. |
| `ordem` | int | Ordem de apresentação. |

### Tentativa
| Campo | Tipo | Descrição |
|---|---|---|
| `id` | UUID | |
| `usuario_id` | UUID (FK) | |
| `questao_id` | UUID (FK) | |
| `alternativa_escolhida_id` | UUID (FK) | |
| `acertou` | bool | Derivado (denormalizado para consultas rápidas). |
| `tempo_segundos` | int | Tempo gasto na questão. |
| `xp_ganho` | int | XP creditado nesta tentativa. |
| `moedas_ganhas` | int | Moedas creditadas. |
| `criado_em` | timestamp | |

> Tentativas são **append-only**; nunca editadas. Regras de XP e moedas são calculadas no momento do registro e armazenadas como histórico.

### Badge
| Campo | Tipo | Descrição |
|---|---|---|
| `id` | UUID | |
| `nome` | string | Ex.: "Mestre dos Inteiros". |
| `descricao` | text | |
| `icone` | string | URL ou slug do ícone. |
| `raridade` | enum (comum, raro, epico, lendario) | |
| `criterio` | json | Regra avaliada pelo Serviço de Recompensas. |
| `xp_bonus` | int | XP extra ao conquistar. |
| `moedas_bonus` | int | Moedas extras ao conquistar. |

Exemplo de `criterio`:
```json
{
  "tipo": "acertos_unidade",
  "unidade_tematica": "numeros",
  "ano": 7,
  "quantidade": 50
}
```

### UsuarioBadge
| Campo | Tipo | Descrição |
|---|---|---|
| `usuario_id` | UUID (FK) | |
| `badge_id` | UUID (FK) | |
| `conquistado_em` | timestamp | |

> Chave primária composta `(usuario_id, badge_id)`. Concessão é idempotente.

### Recompensa
| Campo | Tipo | Descrição |
|---|---|---|
| `id` | UUID | |
| `nome` | string | |
| `descricao` | text | |
| `tipo` | enum (avatar, tema, dica, tentativa_extra, conteudo_exclusivo) | |
| `custo_moedas` | int | Preço de compra. |
| `estoque` | int ou null | `null` = ilimitado. |
| `disponivel` | bool | |

### UsuarioRecompensa
| Campo | Tipo | Descrição |
|---|---|---|
| `id` | UUID | |
| `usuario_id` | UUID (FK) | |
| `recompensa_id` | UUID (FK) | |
| `adquirido_em` | timestamp | |
| `custo_pago` | int | Histórico do preço pago. |

### MovimentacaoMoeda
| Campo | Tipo | Descrição |
|---|---|---|
| `id` | UUID | |
| `usuario_id` | UUID (FK) | |
| `tipo` | enum (ganho, gasto) | |
| `origem` | enum (acerto, badge, compra, streak, evento) | |
| `referencia_id` | UUID | ID da entidade de origem (tentativa, badge, etc.). |
| `valor` | int | Positivo para ganho, negativo para gasto. |
| `saldo_apos` | int | Saldo depois da movimentação (auditoria). |
| `criado_em` | timestamp | |

### Tag
| Campo | Tipo | Descrição |
|---|---|---|
| `id` | UUID | |
| `slug` | string | Identificador único. |
| `descricao` | string | |
| `categoria` | string | Ex.: "contexto", "tipo-de-recurso". |

### Ranking (view materializada)
Calculado a partir de `Tentativa`. Não é tabela editável.

| Campo | Tipo | Descrição |
|---|---|---|
| `tipo` | enum (global, ano, unidade) | |
| `segmento` | string | Ex.: "7", "algebra". |
| `periodo` | enum (diario, semanal, mensal, alltime) | |
| `usuario_id` | UUID | |
| `posicao` | int | |
| `pontuacao` | int | XP acumulado no período. |
| `questoes_acertadas` | int | |
| `taxa_acerto` | float | |

## Índices Sugeridos

- `Questao`: `(ano, unidade_tematica, dificuldade)`, `(habilidade_bncc)`.
- `Tentativa`: `(usuario_id, criado_em)`, `(questao_id)`, `(criado_em)`.
- `Usuario`: `(email)` único.
- `UsuarioBadge`: `(usuario_id)`.
- `MovimentacaoMoeda`: `(usuario_id, criado_em)`.

## Integridade e Consistência

- A soma de `MovimentacaoMoeda.valor` por usuário deve sempre bater com `Usuario.moedas_saldo` (verificável por job de auditoria).
- A soma de `Tentativa.xp_ganho` por usuário deve sempre bater com `Usuario.xp_total`.
- A nota `Questao` deve ter **exatamente uma** `QuestaoAlternativa` com `correta = true` (constraint de banco).
