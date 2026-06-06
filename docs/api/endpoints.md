# Contrato da API

> Documento **conceitual**. Os endpoints abaixo descrevem intenções; a forma concreta (REST/GraphQL, schemas OpenAPI, gRPC) é decisão de stack (ver [`tecnologias.md`](../arquitetura/tecnologias.md)).

> Convenção adotada: **REST + JSON** com prefixo `/api/v1`. Todos os endpoints autenticados esperam o header `Authorization: Bearer <token>`.

---

## Autenticação

### `POST /api/v1/auth/register`
Cria uma nova conta de aluno.

**Body**
```json
{
  "nome": "Maria Silva",
  "email": "maria@exemplo.com",
  "senha": "S3nh@Forte!",
  "ano_escolar": 7,
  "termos_aceitos": true
}
```

**Resposta 201**
```json
{
  "usuario": { "id": "uuid", "nome": "Maria Silva", "ano_escolar": 7 },
  "token": "jwt..."
}
```

### `POST /api/v1/auth/login`
**Body** `{ "email": "...", "senha": "..." }` → **200** `{ "token": "...", "refresh_token": "...", "usuario": {...} }`

### `POST /api/v1/auth/refresh`
**Body** `{ "refresh_token": "..." }` → **200** `{ "token": "..." }`

### `POST /api/v1/auth/logout`
Invalida o refresh token atual. **204 No Content**

### `POST /api/v1/auth/password/reset/request`
**Body** `{ "email": "..." }` → **202** (sempre, mesmo se email não existir — evitar enumeração).

### `POST /api/v1/auth/password/reset/confirm`
**Body** `{ "token": "...", "nova_senha": "..." }` → **204**

---

## Usuários

### `GET /api/v1/usuarios/me`
Retorna perfil do usuário autenticado.
**200**
```json
{
  "id": "uuid", "nome": "...", "email": "...", "ano_escolar": 7,
  "xp_total": 1840, "nivel": 8, "moedas_saldo": 230,
  "streak_atual": 5, "criado_em": "2024-03-01T..."
}
```

### `PATCH /api/v1/usuarios/me`
Atualiza campos editáveis: `nome`, `ano_escolar`, `avatar_url`.

### `GET /api/v1/usuarios/me/progresso`
Estatísticas detalhadas do aluno.
**200**
```json
{
  "questoes_respondidas": 220,
  "questoes_acertadas": 175,
  "taxa_acerto_geral": 0.795,
  "tempo_medio_resposta_segundos": 42,
  "por_unidade": [
    { "unidade": "numeros",      "respondidas": 60, "acertadas": 50, "taxa": 0.83 },
    { "unidade": "algebra",      "respondidas": 40, "acertadas": 30, "taxa": 0.75 },
    { "unidade": "geometria",    "respondidas": 50, "acertadas": 38, "taxa": 0.76 },
    { "unidade": "grandezas_medidas", "respondidas": 30, "acertadas": 25, "taxa": 0.83 },
    { "unidade": "prob_estat",   "respondidas": 40, "acertadas": 32, "taxa": 0.80 }
  ],
  "por_habilidade": [
    { "codigo": "EF07MA13", "taxa_acerto": 0.9, "respondidas": 20 },
    ...
  ]
}
```

### `GET /api/v1/usuarios/me/tentativas?limit=20&offset=0`
Histórico de tentativas paginado.

### `DELETE /api/v1/usuarios/me`
Exclusão de conta (LGPD). Solicita confirmação por email.

### `GET /api/v1/usuarios/me/export`
Retorna arquivo com dados pessoais do usuário (LGPD — portabilidade).

---

## Questões

### `GET /api/v1/questoes`
Lista questões com filtros opcionais.

**Query params** (todos opcionais)
- `ano` — 6, 7, 8, 9
- `unidade` — `numeros`, `algebra`, `geometria`, `grandezas_medidas`, `prob_estat`
- `objeto` — ex.: `equacao-1-grau`
- `habilidade` — código EFXXMA0X
- `dificuldade` — 1 a 5
- `tags` — csv
- `limit` (default 20), `offset` (default 0)
- `sort` — `aleatorio`, `dificuldade`, `recentes`

**200**
```json
{
  "items": [
    {
      "id": "uuid",
      "enunciado": "Resolva: 2x + 5 = 13",
      "alternativas": [
        { "id": "alt-1", "texto": "x = 3" },
        { "id": "alt-2", "texto": "x = 4" },
        { "id": "alt-3", "texto": "x = 5" },
        { "id": "alt-4", "texto": "x = 6" }
      ],
      "ano": 7,
      "unidade_tematica": "algebra",
      "objeto_conhecimento": "equacao-1-grau",
      "habilidade_bncc": "EF07MA15",
      "dificuldade": 2
    }
  ],
  "total": 245,
  "limit": 20,
  "offset": 0
}
```

> **Importante:** O endpoint **não** retorna qual alternativa é correta nem a explicação (anti-cheat). Esses dados só vêm em [`POST /tentativas`](#post-apiv1tentativas).

### `GET /api/v1/questoes/{id}`
Retorna metadados de uma questão (sem gabarito).

### `POST /api/v1/questoes` (admin)
Cria nova questão.

**Body**
```json
{
  "enunciado": "Resolva: 2x + 5 = 13",
  "ano": 7, "unidade_tematica": "algebra",
  "objeto_conhecimento": "equacao-1-grau",
  "habilidade_bncc": "EF07MA15",
  "dificuldade": 2, "pontuacao_base": 15,
  "explicacao": "...",
  "alternativas": [
    { "texto": "x = 3", "correta": false },
    { "texto": "x = 4", "correta": true  },
    { "texto": "x = 5", "correta": false },
    { "texto": "x = 6", "correta": false }
  ],
  "tags": ["exercicio-basico", "contexto:algebra"]
}
```

**201** → questão criada.

### `PATCH /api/v1/questoes/{id}` (admin)
Edita questão. Se já publicada, incrementa a versão.

### `POST /api/v1/questoes/{id}/publicar` (admin)
Publica a questão (torna-a visível aos alunos).

---

## Tentativas

### `POST /api/v1/tentativas`
Registra a resposta de um aluno a uma questão.

**Body**
```json
{
  "questao_id": "uuid",
  "alternativa_id": "alt-2",
  "tempo_segundos": 35
}
```

**200**
```json
{
  "tentativa_id": "uuid",
  "correta": true,
  "resposta_correta_id": "alt-2",
  "explicacao": "Subtraindo 5 dos dois lados: 2x = 8, dividindo por 2: x = 4",
  "xp_ganho": 18,
  "moedas_ganhas": 3,
  "novas_badges": [
    { "id": "uuid", "nome": "Mestre das Equações", "raridade": "rara" }
  ],
  "xp_total_atual": 1858,
  "nivel_atual": 8,
  "moedas_saldo": 233
}
```

> Após essa resposta, a questão pode ser "respondida novamente" (em outro momento) para revisão, mas conta como nova tentativa (e nova chance de XP se o aluno errar e depois acertar, conforme regras).

### `GET /api/v1/tentativas/{id}`
Detalhe de uma tentativa específica do usuário autenticado.

---

## Ranking

### `GET /api/v1/rankings/{tipo}?periodo=`
Lista de leaderboard. `tipo` ∈ `global`, `ano/{6-9}`, `unidade/{slug}`.

**200**
```json
{
  "tipo": "ano/7",
  "periodo": "semanal",
  "atualizado_em": "2024-09-12T15:00:00Z",
  "items": [
    { "posicao": 1, "usuario_id": "uuid", "nome": "Ana", "xp": 1840, "nivel": 12 },
    { "posicao": 2, "usuario_id": "uuid", "nome": "Bruno", "xp": 1700, "nivel": 11 },
    ...
  ]
}
```

### `GET /api/v1/usuarios/me/posicao?tipo=&periodo=`
Posição do usuário autenticado em um ranking.

**200**
```json
{
  "posicao": 47,
  "posicao_anterior": 52,
  "variacao": 5,
  "xp_periodo": 920
}
```

### `GET /api/v1/rankings/historico?ano=7&periodo=mensal&limite=12`
Histórico de temporadas passadas.

---

## Badges

### `GET /api/v1/badges`
Catálogo de todas as badges (conquistadas ou não).

### `GET /api/v1/usuarios/me/badges`
Badges conquistadas pelo usuário autenticado.

### `GET /api/v1/usuarios/me/badges/pendentes`
Badges com critério quase atendido (gamificação — incentivo).

---

## Loja / Recompensas

### `GET /api/v1/loja/recompensas`
Lista de itens disponíveis para compra.

**200**
```json
{
  "items": [
    { "id": "uuid", "nome": "Avatar Dragão", "tipo": "avatar", "custo_moedas": 250, "estoque": null },
    { "id": "uuid", "nome": "Dica (uso único)", "tipo": "dica", "custo_moedas": 20, "estoque": null },
    { "id": "uuid", "nome": "Tema Noturno", "tipo": "tema", "custo_moedas": 500, "estoque": null }
  ]
}
```

### `POST /api/v1/loja/comprar`
**Body** `{ "recompensa_id": "uuid" }` → **200**
```json
{ "saldo_atual": 230, "item_adquirido": { "id": "...", "tipo": "avatar" } }
```

Possíveis erros:
- `402 Payment Required` — saldo insuficiente.
- `409 Conflict` — estoque esgotado.

### `GET /api/v1/usuarios/me/inventario`
Itens comprados/adquiridos pelo usuário.

### `GET /api/v1/usuarios/me/extrato-moedas?limit=50`
Movimentações de moedas (ganhos e gastos).

---

## Administração

> Todos exigem papel `admin`.

### `POST /api/v1/admin/badges`
Cria nova badge.

### `POST /api/v1/admin/recompensas`
Cria nova recompensa.

### `GET /api/v1/admin/usuarios?search=&page=`
Lista e busca usuários (moderação).

### `PATCH /api/v1/admin/usuarios/{id}`
Suspender, reativar, alterar papel.

### `GET /api/v1/admin/questoes/estatisticas/{id}`
Estatísticas de uma questão (taxa de acerto global, distribuição de alternativas, tempo médio).

### `GET /api/v1/admin/rankings/recalcular` (job manual)
Forçar recálculo de leaderboards.

---

## Códigos de Erro Comuns

| Código | Significado |
|---|---|
| `400` | Requisição malformada (validação falhou). |
| `401` | Não autenticado / token inválido. |
| `403` | Sem permissão para o recurso. |
| `404` | Recurso não encontrado. |
| `409` | Conflito (ex.: email já cadastrado, badge já concedida). |
| `410` | Recurso removido (LGPD — conta excluída). |
| `422` | Entidade inprocessável (regras de negócio). |
| `429` | Rate limit excedido. |
| `500` | Erro interno. |
| `503` | Serviço em manutenção. |

## Modelo de Resposta Padrão

### Sucesso (item único)
```json
{ "data": { ... } }
```

### Sucesso (lista paginada)
```json
{ "items": [...], "total": 245, "limit": 20, "offset": 0 }
```

### Erro
```json
{
  "error": {
    "code": "RECURSO_NAO_ENCONTRADO",
    "message": "A questão de id X não foi encontrada.",
    "details": { "id": "X" }
  }
}
```

## Versionamento

- Versão atual: **v1**.
- Mudanças que quebrem contrato: novo prefixo `/api/v2`.
- Mudanças compatíveis (novo campo, novo endpoint): incrementam `v1.minor` (informado em changelog).
