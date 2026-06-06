# Banco de Questões

## Visão Geral

A aplicação é uma **plataforma de questões de múltipla escolha** alinhadas à BNCC. As questões são o ativo central: tudo no app (XP, ranking, badges, progresso) deriva da relação **usuário × questão**.

## Estrutura da Questão

Uma questão de múltipla escolha contém:

- **Enunciado** — texto (com suporte a markdown e imagens), exibe contexto e pergunta.
- **2 a 6 alternativas** — exatamente **uma correta** (constraint de banco).
- **Explicação** — exibida após a tentativa, justifica a resposta correta.
- **Tags de classificação BNCC** — `ano`, `unidade_tematica`, `objeto_conhecimento`, `habilidade_bncc` (código `EFyyMAzz`).
- **Dificuldade** — escala de 1 (muito fácil) a 5 (muito difícil).
- **Pontuação base** — XP creditado em caso de acerto (calculado pela dificuldade e regras; ver abaixo).

## Classificação pela BNCC

Toda questão deve ser classificada pelos 4 eixos (definidos em [`docs/01-conteudo-bncc/`](../01-conteudo-bncc/README.md)):

| Eixo | Valores | Exemplo |
|---|---|---|
| `ano` | 6, 7, 8, 9 | `7` |
| `unidade_tematica` | `numeros`, `algebra`, `geometria`, `grandezas_medidas`, `prob_estat` | `algebra` |
| `objeto_conhecimento` | string padronizada | `equacao-1-grau` |
| `habilidade_bncc` | código EFXXMA0X | `EF07MA13` |

> A classificação permite filtros precisos no cliente e gera os rankings por unidade temática.

## Dificuldade

Escala de 5 níveis, baseada em critérios qualitativos:

| Nível | Descrição | Exemplo típico |
|---|---|---|
| 1 | Reconhecimento direto, 1 passo | "Quanto é 2 + 3?" |
| 2 | Aplicação de procedimento simples | "Qual o valor de 25% de 80?" |
| 3 | Aplicação em 2 etapas | "Resolva 2x + 3 = 11" |
| 4 | Contextualização, exige interpretação | "Calcule a área do triângulo com base 8 cm e altura 5 cm" |
| 5 | Múltiplas etapas, raciocínio combinado | "Em um triângulo retângulo com catetos 6 e 8, calcule a hipotenusa" |

A dificuldade **influencia a pontuação**:

```
xp_ganho = pontuacao_base * multiplicador_dificuldade
```

| Dificuldade | Pontuação base | XP ganho típico (acerto sem bônus) |
|---|---|---|
| 1 | 10 | 10 |
| 2 | 15 | 15 |
| 3 | 20 | 20 |
| 4 | 30 | 30 |
| 5 | 50 | 50 |

> Os valores são ajustáveis. Devem ser confirmados em decisão de produto.

## Modificadores de Pontuação

Além da dificuldade, outros fatores modulam o XP ganho:

| Fator | Condição | Efeito |
|---|---|---|
| **Bônus de velocidade** | Resposta em < 30% do tempo médio da questão | +20% XP |
| **Bônus de sequência (streak)** | 3+ acertos consecutivos no mesmo dia | +5% XP por acerto na sequência (cap +25%) |
| **Bônus de primeira tentativa** | Primeira vez que o usuário vê a questão | +10% XP |
| **Bônus de unidade favorita** | Unidade em que o aluno tem > 80% de acerto | +5% XP (gamificação) |
| **Penalidade por erro** | Não há — erros não retiram XP | 0 |

> Erros **não retiram XP**, mas podem ser marcados com um "registro" para sugerir revisão.

## Fluxo de Vida da Questão

```
[RASCUNHO] → publicada = false    (visível só para admin/professor)
    │
    │  admin publica
    ▼
[PUBLICADA] → publicada = true     (visível aos alunos; versão imutável)
    │
    │  admin identifica problema (erro de gabarito, ambiguidade)
    ▼
[CORRIGIDA] → versão += 1          (nova versão; anterior fica arquivada)
```

- Versões anteriores são **mantidas** para preservar histórico de tentativas.
- Tentativas existentes continuam referenciando a versão respondida (para auditoria).
- A versão atual é a única servida em `GET /questoes`.

## Filtros Disponíveis no Cliente

O usuário pode filtrar questões por:

- `ano` (6, 7, 8 ou 9)
- `unidade_tematica` (uma das 5)
- `objeto_conhecimento` (dentro de uma unidade)
- `habilidade_bncc` (código exato)
- `dificuldade` (1-5)
- `tags` livres

Combinações resultam em queries parametrizadas no backend.

## Estatísticas por Questão (para o admin)

- Total de tentativas
- Taxa de acerto global
- Taxa de acerto por ano escolar
- Tempo médio de resposta
- % de usuários que erraram cada alternativa (detecta "distrator" muito óbvio)

## Geração de Conteúdo (Sprint 2+)

A longo prazo, o sistema pode apoiar:

- **Importação em lote** via CSV/JSON com validação automática da classificação BNCC.
- **Curadoria** por professores convidados.
- **Banco comunitário** com moderação.

> _Detalhes de versão atual e roadmap ficam em [`visao-geral.md`](../02-arquitetura/visao-geral.md)._
