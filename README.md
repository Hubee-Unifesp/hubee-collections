# Hubee Collections

Coleção de requisições da API Hubee para uso com o [Bruno](https://www.usebruno.com/).

## Como usar

1. Instale o Bruno.
2. Abra o Bruno e selecione **Open Collection**.
3. Escolha a pasta deste repositório.
4. Crie ou importe as requisições dentro desta coleção.

## Organização sugerida

- `health/`: endpoints de disponibilidade
- `environments/local.bru`: ambiente local com `baseUrl` em `http://localhost:3000`

Nas requisições, use `{{baseUrl}}` como host. No Bruno, selecione o ambiente `local` antes de executar uma requisição.

## GitHub

Os arquivos `.bru`, o `bruno.json` e a documentação devem ser versionados. Não versione tokens, senhas ou chaves de API. Para valores locais, use arquivos com a extensão `.local.bru`, que já estão no `.gitignore`.