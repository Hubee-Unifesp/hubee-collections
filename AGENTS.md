# AGENTS.md — Hubee Collections

> **Instruções para Agentes de IA e Desenvolvedores de Testes de API**  
> Repositório: `hubee-collections` (Coleção Oficial de Requisições da API Hubee)  
> Ferramenta de Execução: [Bruno API Client](https://www.usebruno.com)

---

## 1. Visão Geral do Repositório

O `hubee-collections` armazena as coleções declarativas de requisições HTTP para a `hubee-api`. Ele permite que desenvolvedores, QA e agentes testem manualmente e validem os contratos de API de forma reproduzível e versionada no Git, sem necessidade de ferramentas proprietárias com cloud fechada.

- **Formato dos Arquivos:** somente OpenCollection (`.yml`). O Bruno lê um único formato por coleção; como a raiz tem `opencollection.yml`, arquivos `.bru` são ignorados pelo app. **Não crie arquivos `.bru` nem `bruno.json`.**
- **Ambiente Local Padrão:** `baseUrl: http://localhost:3000`
- **Ambiente de Produção:** API hospedada no Render (`https://hubee-api-1.onrender.com`).

---

## 2. Estrutura de Diretórios e Coleções

```text
hubee-collections/
├── environments/                # local.yml e production.yml (baseUrl, ids e segredos)
├── assets/                      # imagem de exemplo para os uploads
├── HEALTH/                      # GET /health
├── AUTH/                        # registro, logins (admin, organizador, comprador) e /auth/me
├── USUARIOS/                    # CRUD administrativo de /users
├── MINHA CONTA/                 # /users/me: perfil, avatar, senha e exclusão
├── ORGANIZACOES/                # CRUD de /organizations e validação RBAC
├── MEMBROS DA ORGANIZACAO/      # /organizations/:orgId/users
├── LOCAIS/                      # CRUD de /venues
├── EVENTOS/                     # CRUD de /events, capa e resumo
│   └── CENARIOS DE BUSCA/       # testes da busca pública de eventos
├── ORGANIZACOES DO EVENTO/      # /events/:eventId/organizations
├── LOTES DE INGRESSO/           # /events/:eventId/ticket-types
├── PEDIDOS/                     # /orders e /users/me/orders
├── PAGAMENTOS/                  # checkout (/payments) e /orders/:orderId/payments
├── INGRESSOS/                   # /tickets, /users/me/tickets e /orders/:orderId/tickets
├── FORNECEDORES/                # CRUD de /suppliers
├── DESPESAS DO EVENTO/          # /events/:eventId/expenses
├── TAREFAS DO EVENTO/           # /events/:eventId/tasks
└── opencollection.yml           # Manifesto da coleção (OpenCollection)
```

Cada pasta tem um `folder.yml` com o token herdado pelas requisições
(`{{tokenAdmin}}`, `{{tokenOrganizador}}` ou `{{tokenComprador}}`) e uma descrição
do recurso. Dentro da pasta, as requisições seguem a ordem criar, listar, buscar,
atualizar e remover.

---

## 3. Regras e Boas Práticas Inegociáveis

### 3.1. Uso Obrigatório de Variáveis de Ambiente (`{{baseUrl}}`)
- **NUNCA** coloque URLs absolutas (`http://localhost:3000/events`) diretamente no campo de URL da requisição.
- Utilize sempre o placeholder `{{baseUrl}}`:
  ```text
  GET {{baseUrl}}/events?page=1&limit=10
  POST {{baseUrl}}/auth/login
  ```

### 3.2. Herança de Autenticação com `folder.yml`
- Em pastas que exigem autenticação (rotas protegidas), configure a auth no arquivo `folder.yml` e use `auth: inherit` nas requisições:
  ```yaml
  request:
    auth:
      type: bearer
      token: "{{token}}"
  ```
- No script pós-resposta (`runtime.scripts` com `type: after-response`) da requisição de login, armazene o token dinamicamente no ambiente de execução:
  ```javascript
  if (res.status === 200 || res.status === 201) {
    bru.setEnvVar("token", res.body.token || res.body.access_token);
  }
  ```

### 3.3. Proibição Absoluta de Segredos no Git
- ⚠️ **NUNCA comite tokens JWT reais, senhas de usuários reais ou credenciais de produção.**
- Payloads de exemplo devem usar credenciais de teste (ex: `teste@hubee.com.br` / `Senha123!`).
- Para credenciais pessoais, marque a variável do ambiente como secreta (`secret: true` no `environments/*.yml`). O valor fica só no app do Bruno da sua máquina e não é gravado no arquivo.

### 3.4. Padrão de Nomenclatura das Requisições
Os arquivos têm o verbo HTTP em caixa alta seguido da ação e do recurso, sem acento, e o `info.name` é igual ao nome do arquivo:
- `POST CRIAR LOTE.yml`
- `GET LISTAR LOTES.yml`
- `GET BUSCAR LOTE.yml`
- `PATCH ATUALIZAR LOTE.yml`
- `DELETE REMOVER LOTE.yml` (ou `CANCELAR`, quando a API só muda o status)

Cenários de erro levam o caso no fim do nome: `POST CRIAR EVENTO SEM PERMISSAO.yml`, `GET MEUS INGRESSOS SEM TOKEN.yml`.

### 3.5. Body de Exemplo e Documentação
- Toda requisição que recebe body traz um exemplo válido, com ids vindos de variáveis (`{{eventId}}`, `{{ticketTypeId}}`...).
- Todo `docs` segue o formato: `# MÉTODO /rota`, descrição, **Acesso** (público, autenticado, ADMIN ou nível na organização), **Campos** (obrigatórios, opcionais e valores de enum) e **Respostas** (status esperados).
- Requisições que criam recursos salvam o id com `bru.setEnvVar` no script `after-response`.

---

## 4. Como Executar as Requisições via Linha de Comando (CLI do Bruno)

Caso o agente ou o desenvolvedor queira rodar as coleções pelo terminal em pipelines de CI/CD:

```bash
# Executar a suíte inteira apontando para o ambiente local
npx @usebruno/cli run --env local

# Executar apenas uma pasta específica (ex: USUARIOS)
npx @usebruno/cli run USUARIOS --env local
```

---

## 5. Governança Git e Versionamento

1. **Branching:**
   - Base do repositório: `main`.
   - Crie branches no padrão: `docs/GOL-<ID>-<slug>` ou `feat/GOL-<ID>-<slug>` (ex: `feat/GOL-137-collections-busca-eventos`).
2. **Commits:**
   - Padrão semântico: `docs(GOL-<ID>): descrição em português` ou `test(GOL-<ID>): descrição`.
   - Exemplo: `docs(GOL-148): adiciona AGENTS.md no hubee-collections`.
3. **Revisão:**
   - Ao criar novos endpoints no backend, atualize o `hubee-collections` correspondente e abra o PR no GitHub.
4. **Proibição de Commits/Push sem Permissão:**
   - ⚠️ **NUNCA commite ou faça push sem autorização explícita do desenvolvedor.**
   - O agente deve apresentar as requisições criadas ou editadas e aguardar a aprovação expressa do desenvolvedor antes de comitar ou fazer push.
5. **Regra de Transição no YouTrack:**
   - Ao iniciar a tarefa, transicione para `State In Progress`.
   - ⚠️ **REGRA OBRIGATÓRIA:** NUNCA mova o status para verificação (`To Verify`) automaticamente. O encerramento da task e a mudança de status devem ser **explicitamente confirmados pelo desenvolvedor responsável**.
