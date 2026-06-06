# Sistema de Recompensas

A aplicação adota um **sistema híbrido** de recompensas, combinando dois mecanismos complementares:

| Mecanismo | Natureza | Valor |
|---|---|---|
| **Badges** (conquistas) | Honorífico, permanente, colecionável | Reconhecimento, status. |
| **Moedas** (economia virtual) | Monetário, gasto em loja | Customização e benefícios. |

---

## Parte 1 — Badges (Conquistas)

### Características
- **Permanentes** — uma vez conquistada, o aluno mantém.
- **Visíveis** no perfil e compartilháveis (futuro: feed social).
- **Idempotentes** — não podem ser conquistadas duas vezes.
- **Automáticas** — concedidas por gatilhos do sistema; nunca manualmente (exceto admin em casos especiais).

### Raridade

| Raridade | Cor sugerida | % de usuários que possui | XP bônus típico |
|---|---|---|---|
| Comum | Cinza | > 30% | 10 |
| Rara | Azul | 10-30% | 30 |
| Épica | Roxo | 1-10% | 100 |
| Lendária | Dourado | < 1% | 300 |

### Categorias de Badges

#### a) Badges de Boas-Vindas / Engajamento
| Badge | Critério |
|---|---|
| Primeiro Passo | Responder a 1ª questão |
| Bem-vindo | Cadastrar e validar email |
| Uma Semana | 7 dias consecutivos com pelo menos 1 questão |
| Um Mês | 30 dias consecutivos |
| Maratonista | 50 questões respondidas em um único dia |

#### b) Badges por Volume
| Badge | Critério |
|---|---|
| Centenário | 100 acertos no total |
| Quinto Centenário | 500 acertos no total |
| Milenário | 1000 acertos no total |
| Velocista | 10 acertos em < 5 minutos |

#### c) Badges por Unidade Temática
Conquistas ao dominar uma unidade em um ano específico.

| Badge | Critério (exemplo) | Raridade |
|---|---|---|
| Aprendiz dos Números (6º) | 50 acertos em Números 6º | Comum |
| Mestre dos Inteiros | 50 acertos em Números 7º | Rara |
| Sumido em Álgebra | 100 acertos em Álgebra 8º | Épica |
| Lenda da Geometria | 200 acertos em Geometria (qualquer ano) | Lendária |
| Estatístico Nato | 100 acertos em Probabilidade e Estatística | Rara |
| Calculista | 100 acertos em Grandezas e Medidas | Rara |

#### d) Badges de Habilidade BNCC Específica
| Badge | Critério |
|---|---|
| Pitagórico | 20 acertos em questões da habilidade `EF09MA13` (Teorema de Pitágoras) |
| Tales de Mileto | 20 acertos em questões da habilidade `EF09MA14` (Tales) |
| Fração Forte | 20 acertos em questões da habilidade `EF07MA08` (frações) |

#### e) Badges de Ranking
| Badge | Critério |
|---|---|
| Pódio Semanal | Top 10 do ranking semanal |
| Pódio Mensal | Top 3 do ranking mensal |
| Rei do Mês | #1 do ranking mensal |
| Lendário All-Time | Top 5% do ranking all-time |

#### f) Badges Sazonais
Concedidas em eventos especiais (Olimpíadas, Semana da Matemática, Black Friday da App, etc.).

### Notificação ao Aluno

Ao ganhar uma badge, o cliente recebe:
- Notificação in-app (toast/banner).
- Notificação push (se habilitada).
- Email semanal de "badges conquistadas" (digest).

### Visualização no Perfil

- Grid de badges, ordenável por raridade, data de conquista ou categoria.
- Cada badge exibe ícone, nome, descrição, raridade e data de conquista.
- Badges ainda não conquistadas aparecem "silhuetadas" com a descrição do critério (incentivo).

---

## Parte 2 — Moedas (Economia Virtual)

### Características
- **Numerais** — representadas por um número inteiro (`moedas_saldo`).
- **Auditáveis** — toda movimentação é registrada (ver `MovimentacaoMoeda` no [`modelo-dados.md`](../02-arquitetura/modelo-dados.md)).
- **Nunca compradas com dinheiro real** (na versão atual) — apenas ganhas em jogo.
- **Não transferíveis** entre usuários.

### Como Ganhar Moedas

| Origem | Quantidade | Detalhe |
|---|---|---|
| Acerto de questão | `dificuldade` (1-5 moedas) | 1 moeda por questão fácil, 5 por difícil. |
| Acerto em sequência (streak) | Bônus a cada 5 acertos consecutivos no dia | +5 moedas. |
| Badge conquistada | `_bonus` da badge (definido na tabela de badges) | 0-300 moedas. |
| Conclusão de meta diária | "Responder 10 questões no dia" | +20 moedas. |
| Login diário | +1 moeda | Estimular engajamento. |
| Evento especial | Variável | Datas comemorativas, campanhas. |

### Como Gastar (Loja)

| Categoria | Exemplos | Faixa de preço |
|---|---|---|
| **Avatares** | Imagens para o perfil | 50-500 moedas |
| **Temas visuais** | Cores/layout da UI | 100-1000 moedas |
| **Dicas em questões** | "Elimine 1 alternativa" | 20 moedas por uso |
| **Tentativas extras em quiz** | "Mais 1 chance" (em modos com limite) | 30 moedas |
| **Conteúdo exclusivo** | Lições, simulados extras, e-books | 200-2000 moedas |
| **Ressurreição de streak** | "Continuar streak mesmo sem acertar" | 50 moedas |

### Saldo e Auditoria

- O usuário pode ver:
  - Saldo atual.
  - Extrato das últimas 50 movimentações (com data, origem, valor).
  - Total ganho e total gasto no mês.

### Política de Saldo

- Saldo **não expira** enquanto a conta estiver ativa.
- Se a conta for desativada, o saldo é zerado após 90 dias (LGPD).
- Moedas não são herdadas por terceiros.

---

## Integração entre Badges e Moedas

| Evento | Badges | Moedas |
|---|---|---|
| Aluno acerta questão pela 1ª vez | Possível nova badge "Primeira Questão" | +1-5 (conforme dificuldade) |
| Aluno conquista badge "Centenário" | Badge creditada | +XP_bonus (raridade) + moedas_bonus (raridade) |
| Aluno faz compra na loja | — | -custo_moedas |
| Aluno entra no top 3 semanal | Badge sazonal | +100 moedas |

## Anti-Frustração

- A curva de ganho de moedas deve permitir que um aluno médio compre pelo menos **1 item na loja por semana**.
- Se o saldo ficar inacessível para qualquer item, revisar a economia (game design).
- Não criar "paywalls" com moedas — itens cosméticos e de conveniência, jamais exigidos para progredir.

## Pendências

- [ ] Confirmar valores numéricos de XP/moedas em decisão de produto
- [ ] Definir lista final de badges (apenas exemplos acima)
- [ ] Implementar sistema de eventos sazonais
- [ ] Avaliar integração com feeds sociais
