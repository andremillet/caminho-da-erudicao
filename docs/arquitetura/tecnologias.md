# Stack Tecnológica

> **Status:** A definir.
>
> Este documento registra os **critérios de escolha** e o **espaço reservado** para a stack final. A arquitetura foi mantida agnóstica propositadamente (ver [`visao-geral.md`](visao-geral.md)) para que a decisão seja tomada após validação de protótipos.

## Critérios de Escolha

| Critério | Peso | Justificativa |
|---|---|---|
| Velocidade de desenvolvimento | Alto | Documentação e tempo de mercado importam. |
| Maturidade do ecossistema | Alto | Evitar dependências instáveis. |
| Facilidade de contratação | Médio | Equipe pode crescer no futuro. |
| Suporte a banco relacional | Alto | Modelo de dados é relacional e tem fortes constraints. |
| Performance para consultas de ranking | Alto | Recalcular ranking com 10k+ usuários ativos. |
| Suporte a jobs periódicos (cron) | Alto | Recálculo de leaderboards. |
| Hospedagem simples e barata | Médio | Início de operação com baixo orçamento. |
| Documentação em PT-BR | Baixo | Preferência secundária. |

## Espaço para a Decisão Final

A ser preenchido após decisão:

| Camada | Tecnologia escolhida | Versão | Justificativa |
|---|---|---|---|
| Frontend | _a definir_ | | |
| Backend (linguagem) | _a definir_ | | |
| Framework backend | _a definir_ | | |
| Banco de dados | _a definir_ | | |
| Cache | _a definir_ | | |
| Fila / Mensageria | _a definir_ | | |
| Autenticação | _a definir_ | | |
| Hospedagem | _a definir_ | | |
| Monitoramento | _a definir_ | | |

## Alternativas em Consideração

A lista abaixo é apenas referência — não constitui compromisso.

### Backend
- **Node.js (Express/NestJS)** — ecossistema vasto, bom para I/O concorrente.
- **Python (FastAPI / Django REST)** — produtividade, fácil contratação, bom para jobs.
- **Go (Gin/Fiber)** — performance, baixo overhead, ideal para alto tráfego.
- **Java/Kotlin (Spring Boot)** — robustez corporativa, ferramentas maduras.

### Banco
- **PostgreSQL** — relacional completo, gratuito, suporte a JSON e índices avançados.
- **MySQL/MariaDB** — alternativa consolidada.
- **MongoDB** — caso o modelo de questões fique fortemente orientado a documentos.

### Cache / Ranking
- **Redis** — sorted sets prontos para leaderboards.
- **Postgres + materialized views** — manter tudo num banco só.

### Frontend
- **Next.js (React)** — SSR, ecossistema grande.
- **Vue/Nuxt** — curva suave.
- **React Native / Flutter** — se a estratégia for mobile-first.

## Pendências

- [ ] Definir stack do backend
- [ ] Definir banco de dados
- [ ] Definir estratégia de cache
- [ ] Definir plataforma de hospedagem
- [ ] Avaliar custo de infraestrutura para 10k usuários ativos
- [ ] Validar protótipo de cálculo de ranking
