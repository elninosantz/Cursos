# SuperCourses — Guia de Arquitetura

## Estrutura de pastas

```
src/
├── cli/
│   └── app.py              # Entry point, menu de plataformas
├── platforms/
│   ├── __init__.py         # Registro de scrapers
│   ├── base.py             # Classe abstrata BaseScraper
│   └── <plataforma>/
│       ├── __init__.py
│       ├── auth.py         # Login e sessão HTTP
│       ├── parser.py       # Parse de HTML → dataclasses
│       ├── scraper.py      # Orquestração (flui auth → parse → download)
│       └── downloader.py   # Downloads de vídeo e arquivos
└── utils/
    ├── display.py          # Rich helpers + clear() + html_to_markdown()
    └── fs.py               # sanitize_name(), padded(), ensure_dir()
```

---

## Adicionando uma nova plataforma

### 1. Criar a pasta

```
src/platforms/<nome>/
├── __init__.py
├── auth.py
├── parser.py
├── scraper.py
└── downloader.py
```

### 2. Implementar `auth.py`

Responsável por criar e manter um `requests.Session` autenticado.

```python
import requests
from bs4 import BeautifulSoup

class <Nome>Auth:
    def __init__(self, base_url: str, email: str, password: str) -> None:
        self.base_url = base_url.rstrip("/")
        self.session = requests.Session()

    def login(self) -> bool:
        # 1. GET na página de login para capturar CSRF token (se houver)
        # 2. POST com credenciais
        # 3. Verificar redirecionamento ou cookie de sessão
        # Retorna True se autenticado, False caso contrário
        ...

    def get(self, path: str, **kwargs) -> requests.Response:
        url = path if path.startswith("http") else f"{self.base_url}{path}"
        return self.session.get(url, timeout=30, **kwargs)
```

### 3. Implementar `parser.py`

Converte HTML bruto em dataclasses. Sem efeitos colaterais — só recebe string, devolve dados.

**Dataclasses obrigatórias:**

```python
@dataclass
class Course:
    id: str
    title: str
    href: str
    progress: int = 0

@dataclass
class Module:
    id: str
    title: str
    description: str
    lessons: list[Lesson] = field(default_factory=list)

@dataclass
class Lesson:
    id: str
    title: str
    href: str
    duration: str = ""

@dataclass
class CourseDetail:
    title: str
    description: str
    duration: str
    content_count: str
    modules: list[Module] = field(default_factory=list)

@dataclass
class Attachment:
    name: str
    url: str

@dataclass
class LessonDetail:
    title: str
    video_url: str | None       # URL do vídeo (embed ou direto)
    content_html: str           # HTML bruto da descrição
    content_text: str           # Texto plano
    attachments: list[Attachment] = field(default_factory=list)
```

**Funções obrigatórias:**

```python
def parse_courses(html: str) -> list[Course]: ...
def parse_course_detail(html: str) -> CourseDetail: ...
def parse_lesson(html: str, base_url: str) -> LessonDetail: ...
```

### 4. Implementar `downloader.py`

Responsável por downloads de vídeo e arquivos. Reutilize as funções utilitárias de `memberkit/downloader.py` sempre que possível.

**Regras:**
- Vídeos baixados com `yt-dlp -N 20` (20 fragmentos simultâneos)
- Seguir redirects nos downloads de arquivo (`allow_redirects=True`)
- Respeitar `Content-Disposition` para nomear arquivos quando disponível
- Se a plataforma usa player diferente do Panda Video, adicionar uma função `_<player>_to_url(embed_url)` que converte o embed URL para o URL direto do stream

### 5. Implementar `scraper.py`

Orquestra o fluxo completo. Deve estender `BaseScraper` e implementar `run()`.

**Fluxo padrão:**

```
clear() + print_banner()
→ pede credenciais (com defaults do .env via os.getenv)
→ clear() + print_banner()
→ auth.login()
→ busca lista de cursos
→ clear() + print_banner()
→ exibe tabela + pede seleção
→ clear() + print_banner()
→ processa cada curso/módulo/aula com Progress bar
```

**Estrutura de pastas de saída:**

```
downloads/
└── <Nome do Curso>/
    ├── info.md
    └── <01 - Nome do Módulo>/
        ├── info.md
        └── <01 - Nome da Aula>/
            ├── <titulo>.mp4
            ├── descricao.md
            ├── descricao.html
            └── materiais/
                └── arquivo.pdf
```

**Skip de aulas já baixadas:** verificar se existe arquivo de vídeo na pasta antes de iniciar o download.

```python
_VIDEO_EXTENSIONS = {".mp4", ".mkv", ".webm", ".ts", ".m4v", ".avi", ".mov"}

def _lesson_already_downloaded(les_dir: Path) -> bool:
    if not les_dir.exists():
        return False
    return any(f.suffix.lower() in _VIDEO_EXTENSIONS for f in les_dir.iterdir())
```

### 6. Registrar no `platforms/__init__.py`

```python
from src.platforms.<nome>.scraper import <Nome>Scraper

_SCRAPERS: dict[str, type] = {
    "memberkit": MemberKitScraper,
    "<nome>": <Nome>Scraper,      # adicionar aqui
}
```

### 7. Registrar no `cli/app.py`

```python
_PLATFORMS = [
    ("1", "MemberKit", "memberkit"),
    ("2", "<Nome>", "<nome>"),    # adicionar aqui
]
```

### 8. Variáveis de ambiente

Adicionar ao `.env.example`:

```env
<NOME>_URL=https://...
<NOME>_EMAIL=
<NOME>_PASSWORD=
```

---

## Padrões gerais

| O quê | Como |
|---|---|
| Limpar terminal | `clear()` de `utils/display.py` — usa `\033c` |
| Nomes de pasta | `sanitize_name()` + `padded()` de `utils/fs.py` |
| HTML → Markdown | `html_to_markdown()` de `utils/display.py` — links como `texto (url)` |
| Progresso | `rich.progress.Progress` com `transient=True` |
| Credenciais | `os.getenv()` com fallback vazio; `.env` via `python-dotenv` |
| Erros de rede | Capturar `requests.RequestException`, logar em vermelho, continuar |
| Sem comentários | Só quando o *porquê* é não-óbvio (workaround, invariante oculta) |
