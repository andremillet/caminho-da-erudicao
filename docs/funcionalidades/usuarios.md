# Usuários e Autenticação

## Visão Geral

O sistema é multiusuário e suporta três perfis:
- **Aluno** — resolve questões, ganha XP/moedas, compete no ranking, compra na loja.
- **Professor** — visualiza desempenho de turmas, acompanha progresso por unidade temática, pode sugerir bancos de questões (a depender do produto).
- **Admin** — gerencia banco de questões, badges, recompensas e usuários.

## Cadastro

### Dados Mínimos
- `nome` — obrigatório, 2 a 60 caracteres.
- `email` — obrigatório, único, validado.
- `senha` — obrigatória, mínimo 8 caracteres, com política de complexidade.
- `ano_escolar` — opcional no cadastro; pode ser definido/editado depois. Valores: 6, 7, 8, 9.

### Regras
- Email é o identificador único de login.
- Idade mínima: 10 anos (alunos típicos do 6º ano têm 11 anos).
- Termo de uso e Política de Privacidade (LGPD) devem ser aceitos.
- Opt-in para comunicações opcionais (newsletter, notificações push).

## Login e Autenticação

- **Token stateless** (JWT ou similar) com expiração de 24h e refresh token.
- Senhas armazenadas com algoritmo de hash forte (bcrypt/argon2 — decisão de stack).
- Bloqueio progressivo após 5 tentativas inválidas em 5 minutos.
- Recuperação de senha por email com token de uso único (TTL 30 min).

## Perfil

O perfil do aluno exibe:

- **Cabeçalho** — avatar, nome, ano escolar, nível atual, barra de XP.
- **Estatísticas** — questões respondidas, taxa de acerto geral, tempo médio de resposta.
- **Progresso por unidade temática** — barra de progresso (acerto × cobertura de habilidades) para cada uma das 5 unidades.
- **Streak** — dias consecutivos de pelo menos 1 questão respondida.
- **Badges conquistadas** — em destaque as raras/épicas; ver galeria completa na aba de badges.
- **Histórico recente** — últimas 20 tentativas (questão, unidade, acerto/erro, XP ganho).
- **Saldo de moedas** — com botão "ir para a loja".

## Configurações do Aluno

- Trocar ano escolar (caso o aluno tenha passado de ano).
- Editar nome e avatar.
- Ativar/desativar notificações.
- Excluir conta (LGPD — exclusão em até 30 dias, com confirmação por email).
- Exportar dados pessoais (LGPD — portabilidade).

## Multiplataforma / Sessões

- O usuário pode manter várias sessões (ex.: celular + web).
- Listagem e revogação de sessões ativas no perfil.
- Logout invalida o refresh token da sessão.

## Segurança

- Senhas **nunca** retornam em respostas de API.
- Endpoints autenticados verificam `Authorization: Bearer <token>`.
- Rate limit por IP e por usuário em endpoints sensíveis (login, recuperação de senha).
- Logs de auditoria para ações de admin (criação de badge, alteração de preço, exclusão de questão).
- Cookies (quando aplicável) com `HttpOnly`, `Secure` e `SameSite=Lax`.
- Conformidade LGPD: dados pessoais mínimos, política de retenção, direito ao esquecimento.

## Papéis e Permissões (RBAC resumido)

| Ação | Aluno | Professor | Admin |
|---|---|---|---|
| Resolver questões | ✓ | ✓ | ✓ |
| Ver ranking | ✓ | ✓ | ✓ |
| Ver progresso próprio | ✓ | ✓ | ✓ |
| Ver progresso de turmas | ✗ | ✓ | ✓ |
| Criar/editar questões | ✗ | ✗ | ✓ |
| Criar/editar badges | ✗ | ✗ | ✓ |
| Criar/editar recompensas | ✗ | ✗ | ✓ |
| Banir/desativar usuários | ✗ | ✗ | ✓ |

## Eventos de Domínio Publicados

A API publica eventos para o Serviço de Recompensas ao:

- `usuario.criado` — boas-vindas (badge "Bem-vindo").
- `usuario.ano_alterado` — pode habilitar/desabilitar conquistas.
- `usuario.desativado` — invalida ranking.
