# Pipeline do Scraper

## Visão geral

Na Turtle, o scraping de marketplaces é feito diariamente. Para garantir consistência e qualidade dos dados, **todos os scrapers seguem a mesma pipeline**, independente do marketplace.

O fluxo completo tem 6 etapas:

```
Banco de dados → URLs de pesquisa → Paginação e listagem → Limpeza → Coleta de atributos → Armazenamento
```

---

## Etapas da pipeline

### 1. Leitura dos produtos cadastrados

O ponto de partida são os produtos já registrados no banco de dados da Turtle. O scraper **nunca busca produtos aleatoriamente** — ele sempre parte da lista de produtos cadastrados.

---

### 2. Construção das URLs de pesquisa

Para cada produto cadastrado, o scraper monta automaticamente uma URL de pesquisa no marketplace.

**Exemplo:**

| Produto cadastrado | URL gerada |
|---|---|
| GoPro Hero 12 | `www.marketplace.com.br/search=gopro_hero_12` |

> Cada marketplace tem um formato diferente de URL. Consulte a página específica do marketplace nesta documentação para ver como as URLs são construídas em cada caso.

---

### 3. Acesso às URLs e coleta da listagem

Com as URLs montadas, o scraper começa a acessá-las. Nessa etapa são coletados apenas dados superficiais de cada resultado:

- **URL** do produto
- **Título** do produto
- **ID** do produto no marketplace

#### 3.1 Paginação

O scraper percorre **todas as páginas** de resultado de cada URL de pesquisa, garantindo que nenhum produto seja perdido por estar em uma página mais funda.

---

### 4. Limpeza por palavra-chave

Após coletar a listagem bruta, os resultados passam por um filtro de limpeza. O objetivo é descartar produtos irrelevantes que apareceram na pesquisa mas não correspondem ao produto buscado.

> Para entender os critérios de limpeza e como configurá-los, consulte a documentação específica de limpeza por palavra-chave.

---

### 5. Coleta de atributos dos produtos

Somente após a limpeza o scraper acessa individualmente cada URL de produto. Nessa etapa são coletados os **atributos detalhados**, como preço, vendedor, avaliações, estoque, entre outros.

> Acessar cada URL individualmente é a etapa mais demorada do processo. Por isso, a limpeza da etapa anterior é fundamental para evitar desperdício de tempo processando produtos irrelevantes.

---

### 6. Armazenamento no banco de dados local

Todos os atributos coletados são salvos em um arquivo **SQLite** criado individualmente para cada marketplace. Esse arquivo é o que possibilita a extração e envio dos dados para o time de analistas.

---

## Resumo visual

```
[1] Produtos no banco
        |
        v
[2] Montar URLs de pesquisa
        |
        v
[3] Acessar URLs → paginar → coletar URL + Título + ID
        |
        v
[4] Filtrar resultados por palavra-chave
        |
        v
[5] Entrar em cada URL e coletar atributos detalhados
        |
        v
[6] Salvar no SQLite do marketplace
```
