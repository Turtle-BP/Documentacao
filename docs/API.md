# API Turtle

A API Turtle é o backend central da Turtle. Ela é responsável por fornecer dados para os scrapers e registrar logs de execução.

**URL base:** `https://turtle-jiy1safcn-pedrocdellazzari-gmailcoms-projects.vercel.app`

---

## Autenticação

Todos os endpoints — exceto `/health` — exigem autenticação via API Key.

A chave deve ser enviada no header `Authorization` de cada requisição:

```
Authorization: <sua-api-key>
```

| Situação | HTTP Status | Mensagem |
|---|---|---|
| Header ausente | `401 Unauthorized` | `"API Key ausente"` |
| Chave incorreta | `401 Unauthorized` | `"API Key inválida"` |

---

## Endpoints

### Health

#### `GET /health`

Verifica se a API está no ar. **Não requer autenticação.**

**Resposta de sucesso (`200`):**
```json
{
  "status": "ok"
}
```

---

### Auth

#### `GET /v1/auth/me`

Verifica se a API Key enviada é válida.

**Resposta de sucesso (`200`):**
```json
{
  "authenticated": true,
  "message": "API Key válida"
}
```

---

### Database

Todos os endpoints abaixo requerem autenticação.

---

#### `GET /v1/database/health`

Verifica se a conexão com o banco de dados MySQL está funcionando.

**Resposta de sucesso (`200`):**
```json
{
  "status": "healthy",
  "message": "Database connection is working",
  "database": "nome_do_banco"
}
```

**Resposta de erro (`503`):**
```json
{
  "detail": "Database connection failed"
}
```

---

#### `GET /v1/database/marca_registrada/`

Retorna as URLs cadastradas para uma marca no banco de dados.

**Query Parameters:**

| Parâmetro | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `BRAND` | `string` | Sim | Nome da marca registrada |
| `MARKETPLACE` | `string` | Não | Nome do marketplace para filtrar |

**Comportamento do filtro:**

- **Sem `MARKETPLACE`:** retorna apenas URLs onde o campo `MARKETPLACE` é `"-"` (URLs sem marketplace associado)
- **Com `MARKETPLACE`:** retorna URLs onde `MARKETPLACE` é `"-"` **ou** o marketplace informado

**Validação do campo `BRAND`:** aceita apenas letras, números, espaços, hífens (`-`) e underscores (`_`). Caracteres especiais retornam `400`.

**Exemplo de requisição:**
```
GET /v1/database/marca_registrada/?BRAND=Nike&MARKETPLACE=MercadoLivre
```

**Resposta de sucesso (`200`):**
```json
{
  "marca": "Nike",
  "urls": [
    "https://www.nike.com/br",
    "https://lista.mercadolivre.com.br/nike"
  ],
  "total": 2
}
```

**Possíveis erros:**

| HTTP Status | Situação |
|---|---|
| `400` | Nome de marca com caracteres inválidos |
| `500` | Erro ao consultar o banco de dados |

---

### Logging

Endpoints para registrar o início e o fim de execuções de scrapers. Todos requerem autenticação.

---

#### `POST /v1/logging/start`

Registra o início de uma execução de scraping. Cria um novo registro na tabela `Scrapers.LogsScrapers` com status `em_andamento` e retorna o `ID` gerado.

**Body (JSON):**

| Campo | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `marketplace` | `string` | Sim | Nome do marketplace |
| `brand` | `string` | Sim | Nome da marca |
| `tipo` | `string` | Sim | Tipo do scraping (ex: `"produtos"`) |

**Exemplo de requisição:**
```json
{
  "marketplace": "MercadoLivre",
  "brand": "Nike",
  "tipo": "produtos"
}
```

**Resposta de sucesso (`200`):**
```json
{
  "id": 42,
  "message": "Log iniciado com sucesso"
}
```

> Guarde o `id` retornado — ele é necessário para finalizar o log no endpoint `/finish`.

---

#### `PUT /v1/logging/finish`

Finaliza um log de scraping existente. Atualiza o status, registra o `DataFinal` e calcula automaticamente o `TempoMinutos` com base no `DataInicio` do registro.

**Só pode ser chamado em logs com status `em_andamento`.** Tentar finalizar um log já finalizado retorna `400`.

**Body (JSON):**

| Campo | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `id` | `integer` | Sim | ID do log retornado pelo `/start` |
| `status` | `string` | Sim | Status final. Ver valores permitidos abaixo |
| `error_message` | `string` | Não | Mensagem de erro (preencher quando `status = "erro"`) |

**Valores permitidos para `status`:**

| Valor | Descrição |
|---|---|
| `sucesso` | Scraping concluído sem erros |
| `erro` | Scraping falhou |
| `teste` | Execução de teste |

> O valor `em_andamento` **não** é aceito no `/finish` — ele é definido automaticamente pelo `/start`.

**Exemplo de requisição (sucesso):**
```json
{
  "id": 42,
  "status": "sucesso",
  "error_message": null
}
```

**Exemplo de requisição (erro):**
```json
{
  "id": 42,
  "status": "erro",
  "error_message": "Timeout ao acessar a página de resultados"
}
```

**Resposta de sucesso (`200`):**
```json
{
  "id": 42,
  "message": "Log finalizado com sucesso"
}
```

**Possíveis erros:**

| HTTP Status | Situação |
|---|---|
| `400` | Log já foi finalizado anteriormente |
| `404` | ID do log não encontrado |
| `422` | Status inválido (valor fora dos permitidos) |
| `500` | Erro ao atualizar o banco de dados |

---

## Fluxo típico de uso nos scrapers

```
1. POST /v1/logging/start  →  obter log_id
2. [scraper executa]
3. PUT  /v1/logging/finish →  enviar log_id + status final
```

---

## Variáveis de ambiente

A API utiliza um arquivo `.env` na raiz do projeto com as seguintes variáveis:

| Variável | Obrigatória | Padrão | Descrição |
|---|---|---|---|
| `DB_HOST` | Sim | — | Host do banco MySQL |
| `DB_PORT` | Não | `3306` | Porta do banco MySQL |
| `DB_USER` | Sim | — | Usuário do banco |
| `DB_PASSWORD` | Sim | — | Senha do banco |
| `DB_NAME` | Sim | — | Nome do banco de dados |
| `SENHA_API_TURTLE` | Sim | — | API Key usada para autenticar requisições |
| `APP_NAME` | Não | `"Scraper API"` | Nome exibido na documentação interativa |
| `ENV` | Não | `"dev"` | Ambiente. Em `"production"`, desativa `/docs`, `/redoc` e `/openapi.json` |
