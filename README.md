# Hubee Collections

Coleção de requisições da API Hubee para uso com o [Bruno](https://www.usebruno.com/).

## Como usar

1. Instale o Bruno.
2. Abra o Bruno e selecione **Open Collection**.
3. Escolha a pasta deste repositório.
4. Crie ou importe as requisições dentro desta coleção.

## Organização sugerida

- `EVENTOS/`, `FORNECEDOR/`, `INGRESSOS/`, `LOCAL/`, `ORGANIZACOES/`, `PEDIDOS/`, `USUARIOS/`: requisições por domínio
- `environments/local.yml`: ambiente local com `baseUrl` em `http://localhost:3000`
- `environments/production.yml`: ambiente de produção

Nas requisições, use `{{baseUrl}}` como host. No Bruno, selecione o ambiente `local` antes de executar uma requisição.

## GitHub

A coleção usa só o formato OpenCollection (`.yml`). O Bruno lê um formato por coleção, então arquivos `.bru` não aparecem no app: não crie `.bru` nem `bruno.json`.

Os arquivos `.yml` e a documentação devem ser versionados. Não versione tokens, senhas ou chaves de API: use variáveis secretas no ambiente (`secret: true`).

## Testes de permissão de organizador (GOL-147)

As requisições `USUARIOS/POST LOGIN ORGANIZADOR`, `USUARIOS/POST LOGIN COMPRADOR`,
`USUARIOS/GET PERFIL ME` e as da pasta `EVENTOS/` usam estas variáveis do ambiente `local`:

| Variável | Tipo | O que colocar |
| --- | --- | --- |
| `organizerId` | comum | ID de uma organização em que o organizador tem vínculo aceito com `permission = full`. |
| `venueId` | comum | ID de um local existente. |
| `emailOrganizador` / `senhaOrganizador` | secreta | Usuário com esse vínculo aceito na organização. |
| `emailComprador` / `senhaComprador` | secreta | Usuário comum, sem vínculo na organização. |
| `tokenOrganizador` / `tokenComprador` | secreta | Preenchidas pelos logins; não edite. |

As variáveis secretas ficam só na sua máquina e não são gravadas nos arquivos versionados.
Ordem de execução: os dois logins, `GET PERFIL ME` e depois a pasta `EVENTOS`.
O `POST CRIAR EVENTO` espera `201`, e o `POST CRIAR EVENTO SEM PERMISSAO` espera `403`.

## Pedidos, checkout e ingressos (GOL-175)

As pastas `PEDIDOS/` e `INGRESSOS/` herdam o token do comprador (`tokenComprador`, preenchido por `USUARIOS/POST LOGIN COMPRADOR`). Variáveis novas do ambiente `local`:

| Variável | Tipo | O que colocar |
| --- | --- | --- |
| `ticketTypeId` | comum | ID de um lote com saldo de pelo menos 2 ingressos, de um evento publicado e com vendas abertas. |
| `soldOutTicketTypeId` | comum | ID de um lote esgotado (por exemplo, com `quantity` 0). |
| `orderId` / `ticketId` | comum | Preenchidas pelas requisições; não edite. |

O comprador precisa ter CPF e data de nascimento no perfil, senão `POST CRIAR PEDIDO` responde 403.
Ordem de execução: o login do comprador, a pasta `PEDIDOS` (o `POST PAGAR PEDIDO` por último) e depois a pasta `INGRESSOS`.

## Fornecedores (GOL-202)

A pasta `FORNECEDOR/` herda o token do organizador (`tokenOrganizador`, preenchido por `USUARIOS/POST LOGIN ORGANIZADOR`) e cobre o CRUD e os filtros de busca (`search` e `category`) em `/suppliers`. O `POST FORNECEDOR` salva o ID criado na variável `supplierId` para uso em `GET FORNECEDOR (por id)`, `PATCH FORNECEDOR` e `DELETE FORNECEDOR`.

## Organizações (GOL-197 / GOL-198)

A pasta `ORGANIZACOES/` cobre o fluxo de criação, consulta, atualização e exclusão em `/organizations`:

- `POST CRIAR ORGANIZACAO` cria uma organização com qualquer usuário autenticado, vincula automaticamente o criador como `admin` (`permission = full`, `invite_status = accepted`, nível `manage`) e salva o ID em `{{organizationId}}`.
- `PATCH ATUALIZAR ORGANIZACAO` valida a edição por usuário com nível `manage`.
- `PATCH ORGANIZACAO SEM PERMISSAO` valida o bloqueio RBAC (`403 Forbidden` para membro sem `manage` ou `404 Not Found` para não-membro).

## Gerenciamento de eventos e lotes de ingressos (GOL-207)

A pasta `EVENTOS/` cobre o ciclo completo do organizador para eventos (`/events`) e lotes de ingressos (`/events/:eventId/ticket-types`) usando `{{tokenOrganizador}}`, `{{eventId}}` e `{{ticketTypeId}}`:

- `POST CRIAR EVENTO` cria o evento em rascunho (`draft`) e salva o ID em `{{eventId}}`.
- `PATCH ATUALIZAR EVENTO` atualiza os dados e publica o evento (`status: "published"`).
- `POST CRIAR LOTE INGRESSO` cria um lote vinculado ao evento `{{eventId}}` e salva o ID em `{{ticketTypeId}}`.
- `GET LISTAR LOTES DO EVENTO` lista os lotes do evento com o saldo calculado `available`.
- `PATCH ATUALIZAR LOTE INGRESSO` atualiza nome, preço e quantidade do lote `{{ticketTypeId}}`.
- `DELETE REMOVER LOTE INGRESSO` remove o lote `{{ticketTypeId}}` quando não há ingressos ou pedidos vinculados (`204 No Content`).
- `DELETE CANCELAR EVENTO` cancela o evento `{{eventId}}` exigindo nível `manage` (`204 No Content`).
