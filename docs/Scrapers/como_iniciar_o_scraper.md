# Como executar a configuração inicial de um Scraper

Antes de rodar um scraper pela primeira vez, é necessário preparar o ambiente na sua máquina. Esta página descreve o passo a passo completo.

> **Recomendacao:** Mantenha todas as pastas dos projetos dentro de `Documents/` no seu computador. Isso garante que os caminhos relativos usados nos comandos abaixo funcionem corretamente.

---

## Pré-requisitos

Antes de começar, certifique-se de que sua maquina possui:

- **Python** instalado
- **uv** instalado — gerenciador de pacotes e ambientes virtuais da Astral, recomendado pela Turtle pela sua velocidade. [Veja como instalar o uv](https://docs.astral.sh/uv/getting-started/installation/)
- **Git** instalado

---

## Passo a passo

### 1. Clonar o repositório `core-scrapers`

A `core-scrapers` é uma biblioteca interna criada e mantida pela Turtle. Ela é necessária para todos os scrapers de marketplace.

```bash
git clone https://github.com/Turtle-BP/core_scrapers
```

Isso criará a pasta `core_scrapers/` no diretório atual.

> Para conhecer todas as funcionalidades da biblioteca, consulte a documentação da `core-scrapers` neste site.

---

### 2. Clonar o repositório do marketplace

Cada marketplace possui seu proprio repositório no GitHub. Consulte a página específica do marketplace nesta documentação para obter a URL correta.

```bash
git clone https://github.com/Turtle-BP/<nome-do-marketplace>
```

---

### 3. Criar o ambiente virtual e instalar as dependências

Acesse a pasta do marketplace clonado e execute os comandos abaixo:

#### 3.1 Criar o ambiente virtual

```bash
uv venv
```

#### 3.2 Ativar o ambiente virtual

=== "Windows"
    ```bash
    .venv/Scripts/activate
    ```
=== "Linux / Mac"
    ```bash
    source .venv/bin/activate
    ```

#### 3.3 Instalar as dependências do marketplace

Todo projeto de marketplace possui um arquivo `requirements.txt` que lista as bibliotecas Python necessárias para rodar o scraper.

```bash
uv pip install -r requirements.txt
```

#### 3.4 Instalar a `core-scrapers` em modo editável

A `core-scrapers` precisa ser instalada separadamente dentro do ambiente virtual. O comando abaixo assume que tanto `core_scrapers/` quanto o repositório do marketplace estão dentro de `Documents/`.

```bash
uv pip install -e ../core_scrapers/
```

> A flag `-e` significa **editable install** (instalação editável). Isso faz com que qualquer alteracao feita localmente na pasta `core_scrapers/` seja refletida automaticamente no projeto, sem precisar reinstalar a biblioteca.

---

### 4. Configurar as variáveis de ambiente

Cada repositório de marketplace possui um arquivo `.env.example` na raiz. Copie-o e preencha com as credenciais corretas:

```bash
cp .env.example .env
```

Abra o `.env` e preencha as variáveis:

```env
SENHA_API_TURTLE=<sua-api-key>
```

> Consulte o responsável pelo projeto para obter a API Key da Turtle.

---

### 5. Inicializar o banco de dados

Antes de rodar o scraper, é necessário criar o banco de dados SQLite onde os dados coletados serão armazenados. Para isso, use o comando `core-db-init` fornecido pela `core-scrapers`:

```bash
core-db-init --caminho_db <caminho_banco> --caminho_esquema <caminho_schema>
```

Na grande maioria dos marketplaces, o comando será:

```bash
core-db-init --caminho_db database/ --caminho_esquema database/schemas.py
```

---

## Resumo dos comandos

```bash
# 1. Clonar os repositórios (dentro de Documents/)
git clone https://github.com/Turtle-BP/core_scrapers
git clone https://github.com/Turtle-BP/<nome-do-marketplace>

# 2. Entrar na pasta do marketplace
cd <nome-do-marketplace>

# 3. Criar e ativar o ambiente virtual
uv venv
.venv/Scripts/activate          # Windows
# source .venv/bin/activate     # Linux / Mac

# 4. Instalar dependências
uv pip install -r requirements.txt
uv pip install -e ../core_scrapers/

# 5. Configurar o .env
cp .env.example .env
# Preencha SENHA_API_TURTLE e demais variáveis

# 6. Inicializar o banco de dados
core-db-init --caminho_db database/ --caminho_esquema database/schemas.py
```

Tudo pronto. O scraper já pode ser executado.
