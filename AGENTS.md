# AGENTS.md — Hubee Collections

> **Instruções para Agentes de IA e Desenvolvedores de Testes de API**  
> Repositório: `hubee-collections` (Coleção Oficial de Requisições da API Hubee)  
> Ferramenta de Execução: [Bruno API Client](https://www.usebruno.com)

---

## 1. Visão Geral do Repositório

O `hubee-collections` armazena as coleções declarativas de requisições HTTP para a `hubee-api`. Ele permite que desenvolvedores, QA e agentes testem manualmente e validem os contratos de API de forma reproduzível e versionada no Git, sem necessidade de ferramentas proprietárias com cloud fechada.

- **Formato dos Arquivos:** Sintaxe nativa do Bruno (`.bru`) ou OpenCollection (`.yml`).
- **Ambiente Local Padrão:** `baseUrl: http://localhost:3000`
- **Ambiente de Produção:** API hospedada no Render (`https://hubee-api.onrender.com` ou URL oficial).

---

## 2. Estrutura de Diretórios e Coleções

```text
hubee-collections/
├── environments/                # Definição de variáveis de ambiente
│   ├── local.bru                # baseUrl: http://localhost:3000
│   ├── production.bru           # URL do ambiente de staging / produção
│   └── *.local.bru              # ⚠️ Arquivos ignorados para tokens/senhas locais
├── EVENTOS/                     # Requisições de busca, criação e gestão de eventos
│   ├── folder.bru               # Headers e auth compartilhados da pasta
│   ├── GET BUSCAR EVENTOS.bru
│   └── POST CRIAR EVENTO.bru
├── FORNECEDOR/                  # CRUD de fornecedores de eventos
├── LOCAL/                       # Endpoints de locais e venues
├── USUARIOS/                    # Autenticação, perfil /users/me e cadastro
│   ├── folder.bru
│   ├── POST LOGIN.bru
│   └── GET PERFIL ME.bru
├── bruno.json                   # Manifesto da coleção Bruno
└── opencollection.yml           # Especificação OpenCollection
```

---

## 3. Regras e Boas Práticas Inegociáveis

### 3.1. Uso Obrigatório de Variáveis de Ambiente (`{{baseUrl}}`)
- **NUNCA** coloque URLs absolutas (`http://localhost:3000/events`) diretamente no campo de URL da requisição.
- Utilize sempre o placeholder `{{baseUrl}}`:
  ```text
  GET {{baseUrl}}/events?page=1&limit=10
  POST {{baseUrl}}/auth/login
  ```

### 3.2. Herança de Autenticação com `folder.bru`
- Em pastas que exigem autenticação (rotas protegidas), configure o header no arquivo `folder.bru`:
  ```text
  headers {
    Authorization: Bearer {{token}}
  }
  ```
- No script pós-resposta (`post-response`) da requisição de login, armazene o token dinamicamente no ambiente de execução:
  ```javascript
  if (res.status === 200 || res.status === 201) {
    bru.setEnvVar("token", res.body.token || res.body.access_token);
  }
  ```

### 3.3. Proibição Absoluta de Segredos no Git
- ⚠️ **NUNCA comite tokens JWT reais, senhas de usuários reais ou credenciais de produção.**
- Payloads de exemplo devem usar credenciais de teste (ex: `teste@hubee.com.br` / `Senha123!`).
- Para guardar credenciais pessoais que persistem na sua máquina, use arquivos com terminação `.local.bru` (eles estão cobertos pelo `.gitignore`).

### 3.4. Padrão de Nomenclatura das Requisições
Os arquivos devem refletir o verbo HTTP em caixa alta seguido da intenção funcional:
- `GET LISTAR TODOS.bru`
- `GET BUSCA POR ID.bru`
- `POST CRIA USUARIO.bru`
- `PATCH ATUALIZAR.bru`
- `DELETE DELETAR.bru`

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
