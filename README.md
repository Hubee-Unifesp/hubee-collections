# Hubee Collections

Coleção de requisições da API Hubee para uso com o [Bruno](https://www.usebruno.com/).

## Como usar

1. Instale o Bruno.
2. Abra o Bruno e selecione **Open Collection**.
3. Escolha a pasta deste repositório.
4. Crie ou importe as requisições dentro desta coleção.

## Organização sugerida

- `EVENTOS/`, `FORNECEDOR/`, `INGRESSOS/`, `LOCAL/`, `PEDIDOS/`, `USUARIOS/`: requisições por domínio
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
