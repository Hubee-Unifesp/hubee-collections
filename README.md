# Hubee Collections

Coleção [Bruno](https://www.usebruno.com/) com todas as rotas da [`hubee-api`](https://github.com/Hubee-Unifesp/hubee-api), no formato OpenCollection (`.yml`). Cada pasta é um recurso da API, com o CRUD completo, body de exemplo e documentação (rota, perfil de acesso, campos e respostas) na aba **Docs** de cada requisição.

## Como usar

1. Instale o Bruno e abra esta pasta em **Open Collection**.
2. Selecione o ambiente `local` (`http://localhost:3000`) ou `production` (`https://hubee-api-1.onrender.com`).
3. Em **Environments**, preencha as variáveis secretas de login (ficam só na sua máquina e não são gravadas nos arquivos):

   | Variáveis | Usuário |
   | --- | --- |
   | `emailAdmin` / `senhaAdmin` | role `ADMIN` da plataforma (no seed: `admin@hubee.com.br`, senha de `SEED_ADMIN_PASSWORD`) |
   | `emailOrganizador` / `senhaOrganizador` | usuário com vínculo aceito e `permission = full` na organização `organizerId` |
   | `emailComprador` / `senhaComprador` | usuário comum, sem vínculo na organização, com CPF e data de nascimento no perfil |

4. Preencha as variáveis comuns que não são criadas pela collection: `organizerId` (organização do organizador), `venueId` (se não for rodar LOCAIS), `ticketTypeId` e `soldOutTicketTypeId` (se não for rodar LOTES DE INGRESSO).
5. Rode os três logins da pasta **AUTH**. Eles salvam `tokenAdmin`, `tokenOrganizador` e `tokenComprador`, herdados pelas pastas conforme o perfil que as rotas exigem.

## Estrutura

As pastas seguem a ordem do fluxo da API e, dentro de cada uma, a ordem **criar → listar → buscar → atualizar → remover**.

| # | Pasta | Rota base | Token da pasta | Variável salva |
| -: | :--- | :--- | :--- | :--- |
| 1 | `HEALTH` | `/health` | nenhum | — |
| 2 | `AUTH` | `/auth` | nenhum | `tokenAdmin`, `tokenOrganizador`, `tokenComprador`, `adminUserId`, `registeredUserId` |
| 3 | `USUARIOS` | `/users` | admin | `userId`, `userToDeleteId` |
| 4 | `MINHA CONTA` | `/users/me` | comprador | — |
| 5 | `ORGANIZACOES` | `/organizations` | organizador | `organizationId` |
| 6 | `MEMBROS DA ORGANIZACAO` | `/organizations/:orgId/users` | admin | — |
| 7 | `LOCAIS` | `/venues` | organizador | `venueId` |
| 8 | `EVENTOS` (+ `CENARIOS DE BUSCA`) | `/events` | organizador | `eventId` |
| 9 | `ORGANIZACOES DO EVENTO` | `/events/:eventId/organizations` | admin | — |
| 10 | `LOTES DE INGRESSO` | `/events/:eventId/ticket-types` | organizador | `ticketTypeId` |
| 11 | `PEDIDOS` | `/orders`, `/users/me/orders` | comprador | `orderId`, `orderToCancelId` |
| 12 | `PAGAMENTOS` | `/payments`, `/orders/:orderId/payments` | comprador | — |
| 13 | `INGRESSOS` | `/tickets`, `/orders/:orderId/tickets` | comprador | `ticketId` |
| 14 | `FORNECEDORES` | `/suppliers` | organizador | `supplierId` |
| 15 | `DESPESAS DO EVENTO` | `/events/:eventId/expenses` | admin | `expenseId` |
| 16 | `TAREFAS DO EVENTO` | `/events/:eventId/tasks` | admin | `taskId` |

Requisições que exigem outro perfil trocam o token na própria aba **Auth** (por exemplo, `DELETE REMOVER ORGANIZACAO` usa `tokenAdmin`, e `POST CRIAR EVENTO SEM PERMISSAO` usa `tokenComprador`). Os cenários de erro esperados (`... SEM PERMISSAO`, `... SEM TOKEN`, `... LOTE ESGOTADO`, `... SEM ITENS`) têm testes que conferem o status de erro.

## Fluxo de compra completo

1. `AUTH` › os três logins
2. `ORGANIZACOES` › `POST CRIAR ORGANIZACAO`
3. `LOCAIS` › `POST CRIAR LOCAL`
4. `EVENTOS` › `POST CRIAR EVENTO` → `PATCH ATUALIZAR EVENTO` (publica com `status: "published"`)
5. `LOTES DE INGRESSO` › `POST CRIAR LOTE`
6. `PEDIDOS` › `POST CRIAR PEDIDO`
7. `PAGAMENTOS` › `POST PAGAR PEDIDO` (confirma o pagamento e emite os ingressos)
8. `INGRESSOS` › `GET MEUS INGRESSOS`

As requisições de remoção e cancelamento (`DELETE ...`) apagam registros que as pastas seguintes usam. Ao rodar a coleção inteira, deixe-as para o fim ou rode cada pasta isoladamente. Onde o fluxo precisa remover algo sem quebrar o resto, há um registro descartável: `POST CRIAR USUARIO PARA REMOVER` e `POST CRIAR PEDIDO PARA CANCELAR`.

## Convenções

- Somente OpenCollection (`.yml`). O Bruno lê um formato por coleção: não crie `.bru` nem `bruno.json`.
- Pastas em caixa alta, sem acento, com o nome do recurso em português (`LOTES DE INGRESSO`, `DESPESAS DO EVENTO`).
- Requisições com o verbo HTTP seguido da ação: `POST CRIAR LOTE`, `GET LISTAR LOTES`, `GET BUSCAR LOTE`, `PATCH ATUALIZAR LOTE`, `DELETE REMOVER LOTE`.
- URLs sempre com `{{baseUrl}}` e parâmetros de path (`/events/:id`) preenchidos por variável na aba **Params**. Filtros opcionais de query ficam cadastrados, mas desativados.
- O token fica no `folder.yml` (bearer com `{{tokenAdmin}}`, `{{tokenOrganizador}}` ou `{{tokenComprador}}`) e as requisições usam `auth: inherit`.
- Toda requisição tem body de exemplo (quando a rota recebe body) e `docs` no formato: rota, descrição, **Acesso**, **Campos** e **Respostas**.
- Uploads (avatar e capa do evento) usam `assets/imagem-exemplo.png`; troque o arquivo na aba **Body**.
- Ao criar ou alterar uma rota na API, atualize a requisição correspondente aqui seguindo o mesmo formato.

## Segredos

Não versione tokens, senhas ou chaves de API. Credenciais e tokens ficam em variáveis secretas do ambiente (`secret: true`), cujo valor o Bruno guarda só na sua máquina.
