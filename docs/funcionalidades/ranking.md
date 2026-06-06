# Sistema de Ranking

## Visão Geral

A aplicação oferece **múltiplos rankings** para motivar competição saudável e destacar diferentes tipos de habilidade. Cada ranking é um leaderboard derivado das tentativas dos usuários.

## Tipos de Ranking

### 1. Por Escopo

| Tipo | `segmento` | Descrição |
|---|---|---|
| **Global** | `*` (todos) | Todos os usuários da plataforma. |
| **Por ano escolar** | `"6"`, `"7"`, `"8"`, `"9"` | Apenas usuários de um determinado ano. |
| **Por unidade temática** | `"numeros"`, `"algebra"`, `"geometria"`, `"grandezas_medidas"`, `"prob_estat"` | Apenas tentativas dessa unidade. |

### 2. Por Período

| Período | Janela de tempo | Reset |
|---|---|---|
| **Diário** | Últimas 24h (00:00–23:59 horário de Brasília) | Diário às 00:00 |
| **Semanal** | Última semana (segunda 00:00 → domingo 23:59) | Semanal às 00:00 de segunda |
| **Mensal** | Último mês (dia 1 → último dia) | Mensal às 00:00 do dia 1 |
| **All-time** | Desde a criação da conta | Nunca reseta |

> Total de combinações: 3 escopos × 4 períodos = **12 leaderboards**.

## Métrica Principal

A métrica padrão do ranking é o **XP acumulado no período** (somatório de `Tentativa.xp_ganho`).

Alternativas (futuro):
- Taxa de acerto (cuidado: incentiva acertar fácil)
- Quantidade de questões (volume)
- Quantidade de badges conquistadas

## Regras de Desempate

Quando dois usuários têm o mesmo XP:

1. **Maior número de acertos** no período.
2. **Maior número de questões únicas respondidas**.
3. **Menor tempo total de resposta** (agilidade).
4. **Data de cadastro mais antiga** (fidelidade).

## Visibilidade no Cliente

- **Top 100** é exibido na home.
- **Top 10** ganha destaque com medalhas (🥇🥈🥉).
- Usuário sempre vê **a sua posição** (com setas indicando se subiu ou desceu em relação ao último cálculo).
- **Posição amigável** — se estiver em 102º, mostra "102º (–5)" com queda em 5 posições.

## Cálculo e Atualização

### Estratégia 1 — Recalcular sob demanda
- Calcular a query no momento da consulta.
- Vantagem: sempre atualizado, simples.
- Desvantagem: queries pesadas em horários de pico.

### Estratégia 2 — Job periódico (recomendado)
- A cada 5 minutos (configurável), um job recalcula o top 1000 e armazena em **view materializada** ou coleção equivalente.
- Vantagens: performance previsível, controle de carga.
- Desvantagens: latência de até 5 minutos para o ranking refletir nova XP.

> A escolha da estratégia depende da stack (ver [`tecnologias.md`](../arquitetura/tecnologias.md)). Para a maioria dos casos, **Estratégia 2** é a indicada.

## Comportamento por Período

### Diário e Semanal
- Job **zera o cache** no início do novo período e começa a acumular.
- O histórico de períodos anteriores é preservado para consulta posterior (ver "Histórico de Temporadas").

### Mensal
- Mesma lógica, com janela maior.

### All-time
- Nunca zera; cresce indefinidamente.
- Pode ser apresentado como "lenda do app" para os top 5%.

## Recompensas Associadas (Gamificação)

A definir em [`recompensas.md`](recompensas.md), mas sugere-se:

- **Top 3 semanal** ganha badge sazonal "Pódio da Semana".
- **#1 do mês** ganha badge "Rei do Mês" + 100 moedas bônus.
- **#1 all-time de uma unidade** ganha badge lendária "Sumido em <Unidade>".

## Privacidade e Anti-Trapaça

- Cada tentativa é registrada com timestamp do servidor (não do cliente).
- Detecção de padrões anômalos: resolução de N questões em < M segundos pode ser sinalizada.
- Uma conta por pessoa (email único); múltiplas contas são violação dos termos.
- Reset do leaderboard é punição para casos de fraude confirmados.

## Histórico de Temporadas

Após cada reset (semanal/mensal), o sistema guarda:

- Top 10 do período finalizado.
- Badges distribuídas no período.
- Estatísticas agregadas (total de tentativas, total de acertos na plataforma).

O usuário pode consultar temporadas anteriores (limitado a, por exemplo, últimos 12 meses).

## Endpoints Resumidos (referência)

Ver [`api/endpoints.md`](../api/endpoints.md).

```
GET /rankings/global?periodo=semanal
GET /rankings/ano/7?periodo=mensal
GET /rankings/unidade/algebra?periodo=alltime
GET /usuarios/me/posicao?tipo=global&periodo=semanal
```
