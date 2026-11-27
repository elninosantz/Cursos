# SuperCourses

Agregador multi-plataforma para download de cursos online. Baixa vídeos, materiais e descrições de diversas plataformas em uma estrutura organizada de diretórios.

## Plataformas suportadas

- MemberKit
- Entrega Digital
- Didati.ca
- Udemy
- Balta.io
- desenvolvedor.io

## Requisitos

- Python >= 3.11
- [uv](https://docs.astral.sh/uv/) (recomendado) ou pip

## Instalação

```bash
# Clone o repositório
git clone <url-do-repositorio>
cd SuperCourses

# Instale as dependências com uv
uv sync

# Ou com pip
pip install -e .
```

## Configuração

Crie um arquivo `.env` na raiz do projeto com as credenciais das plataformas que for usar:

```env
MEMBERKIT_URL=https://sua-escola.memberkit.com.br
MEMBERKIT_EMAIL=seu@email.com
MEMBERKIT_PASSWORD=sua_senha

ENTREGADIGITAL_URL=https://sua-escola.entregadigital.com.br
ENTREGADIGITAL_EMAIL=seu@email.com
ENTREGADIGITAL_PASSWORD=sua_senha

DIDATI_URL=https://sua-escola.didati.ca
DIDATI_EMAIL=seu@email.com
DIDATI_PASSWORD=sua_senha

BALTA_EMAIL=seu@email.com
BALTA_PASSWORD=sua_senha

DESENVOLVEDORIO_EMAIL=seu@email.com
DESENVOLVEDORIO_PASSWORD=sua_senha
```

Só é necessário preencher as credenciais das plataformas que for utilizar.

## Uso

```bash
# Com uv
uv run supercourses

# Ou diretamente
python main.py
```

O programa exibe um menu interativo com três modos:

### 1. Scraper

Faz o download completo de um curso. Após selecionar a plataforma, o programa autentica com as credenciais do `.env` (ou permite informar outras), lista os cursos disponíveis e inicia o download.

**Estrutura de saída:**

```
downloads/
└── Nome do Curso/
    └── 01 - Nome do Módulo/
        └── 01 - Nome da Aula/
            ├── video.mp4
            ├── descricao.md
            └── materiais/
```

### 2. Tester de credenciais

Testa uma lista de credenciais em lote. Lê o arquivo `seed.txt` (uma credencial por linha no formato `email:senha`) e reporta quais são válidas. Suporta MemberKit e Entrega Digital.

```
# seed.txt
usuario1@email.com:senha1
usuario2@email.com:senha2
```

### 3. Tester de proxies

Testa proxies em lote a partir do arquivo `proxy.txt` (um proxy por linha no formato `ip:porta`). Executa verificações paralelas e exibe os proxies funcionando com seus IPs externos.

```
# proxy.txt
123.45.67.89:8080
98.76.54.32:3128
```

## Arquivos opcionais

| Arquivo | Formato | Uso |
|---|---|---|
| `seed.txt` | `email:senha` por linha | Lista de contas para o tester de credenciais |
| `proxy.txt` | `ip:porta` por linha | Lista de proxies para rotação ou teste |

Linhas começando com `#` são tratadas como comentários em ambos os arquivos.
