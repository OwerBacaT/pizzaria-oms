# Diário de construção — Pizzeria OMS

Registro de cada passo da construção do projeto: o que foi feito, quais comandos foram usados e **por que** cada coisa foi feita daquele jeito. O projeto está sendo construído do zero, aprendendo cada tecnologia no momento em que ela entra no sistema.

**Como manter este diário:** ao terminar cada passo, adicione uma nova seção no final, seguindo o mesmo formato, e faça o commit junto com o código daquele passo.

## Andamento

| Etapa | Tema | Passos | Status |
|---|---|---|---|
| 1 | Ambiente e ferramentas | 1 a 3 | Concluída |
| 2 | Git e GitHub | 4 a 7 | Concluída |
| 3 | Python local, ambiente virtual e HTTP | 8 em diante | Em andamento |
| 4 | Docker | — | Não iniciada |
| 5 | PostgreSQL e SQL | — | Não iniciada |
| 6 | SQLAlchemy e Alembic | — | Não iniciada |
| 7 | FastAPI, Pydantic e pytest | — | Não iniciada |
| 8 | JavaScript e React | — | Não iniciada |

## Decisões técnicas

| Decisão | Alternativa considerada | Por quê |
|---|---|---|
| Backend em FastAPI | Django | Aprender arquitetura de APIs camada por camada; combina com o objetivo de trabalhar com automação e IA |
| Frontend em JavaScript + React | HTMX | Mais valor para o portfólio, mesmo atrasando um pouco o primeiro uso real |
| Construir seguindo dogfooding | Construir tudo e só depois usar | O sistema é usado na pizzaria aos poucos, e o uso real decide o que construir a seguir |
| Recomeçar o projeto do zero | Continuar o repositório antigo | Aprender cada tecnologia configurando tudo com as próprias mãos; o repositório antigo fica como consulta |
| Reaproveitar `escopo.md` e `modelagem.md` | Refazer os documentos | Eles descrevem as regras da pizzaria, não a tecnologia, então continuam válidos |
| Python rodando primeiro na máquina, Docker depois | Tudo dentro do Docker desde o início | Entender o que o Docker resolve antes de usá-lo; o Docker no início esconderia muita coisa |
| `venv` + `pip` | `uv` | Aprender a base primeiro; migrar para o `uv` depois é simples quando se entende o que ele automatiza |
| Projeto em `C:\oms_pizza` | Área de Trabalho ou Documentos | Caminho curto e sem espaços, fora do OneDrive, que trava pastas com milhares de arquivos |
| Mesma versão do Python no computador e no Docker (3.13) | Versões diferentes | Evita o problema de "na minha máquina funciona" |

## Conceitos aprendidos

| Conceito | Explicação |
|---|---|
| Terminal | Forma de dar ordens ao computador escrevendo comandos; no Windows, usamos o PowerShell |
| PATH | Lista de pastas onde o Windows procura um programa quando você digita o nome dele |
| Camadas de instalação | Máquina (ferramentas), projeto (bibliotecas) e container (ambiente inteiro) |
| Repositório | Pasta acompanhada pelo Git, com todo o histórico guardado em `.git` |
| Commit | Foto do projeto num momento, com uma mensagem dizendo o que mudou |
| Área de preparação | Onde se escolhe o que entra no próximo commit (`git add`) |
| `HEAD` | Marcador de onde você está no histórico |
| Remote / `origin` | O endereço do repositório no GitHub, com o apelido padrão `origin` |
| `.gitignore` | Lista do que o Git nunca deve guardar no histórico |
| venv | Caixa isolada com as bibliotecas Python de um projeto; criada pelo próprio Python |
| Commits convencionais | Prefixos que dizem o tipo da mudança: `chore:` (configuração), `docs:` (documentação), `feat:` (funcionalidade), `fix:` (correção) |

---

# Etapa 1 — Ambiente e ferramentas

## Passo 1 — Criar a pasta do projeto

**Objetivo:** ter um lugar fixo e seguro para o projeto.

```powershell
mkdir C:\oms_pizza
cd C:\oms_pizza
```

**Por quê:** pastas como Área de Trabalho e Documentos costumam estar sincronizadas com o OneDrive, que tenta sincronizar os milhares de arquivos do `node_modules` e deixa tudo lento. Caminhos curtos e sem espaços também evitam erros em várias ferramentas.

**Resultado:** pasta `C:\oms_pizza` criada.

## Passo 2 — Conferir as ferramentas da máquina

**Objetivo:** confirmar que as ferramentas da camada "máquina" estão instaladas.

```powershell
git --version; python --version; node --version; docker --version; code --version
```

**Por quê:** existem três camadas de instalação. As ferramentas (Git, Python, Node, Docker, VS Code) ficam na máquina, instaladas uma vez só. As bibliotecas de cada projeto ficam isoladas dentro do projeto (venv no Python, `node_modules` no JavaScript). E o Docker empacota o ambiente inteiro. A venv não substitui a instalação do Python, porque é o próprio Python que cria a venv.

**Resultado:**

| Ferramenta | Versão |
|---|---|
| Git | 2.54.0 |
| Python | 3.13.5 |
| Node | 24.20.0 (LTS) |
| Docker | 29.8.2 |
| VS Code | 1.142.0 |

## Passo 3 — Conferir a identidade do Git

**Objetivo:** garantir que cada commit fique registrado com o nome e o e-mail certos.

```powershell
git config --global user.name; git config --global user.email
```

**Por quê:** o GitHub liga os commits ao perfil pelo e-mail. Usar o mesmo e-mail da conta faz os commits aparecerem no gráfico de contribuições, o que conta para o portfólio.

**Resultado:** nome e e-mail conferidos.

---

# Etapa 2 — Git e GitHub

## Passo 4 — Transformar a pasta em repositório

**Objetivo:** fazer o Git começar a acompanhar o histórico do projeto.

```powershell
git init -b main
ls -Force
git status
```

| Comando | O que faz |
|---|---|
| `git init -b main` | Cria o repositório, com o branch principal chamado `main` (padrão atual) |
| `ls -Force` | Lista inclusive arquivos ocultos, para enxergar a pasta `.git` |
| `git status` | Mostra o que mudou desde o último registro; é o comando mais usado no dia a dia |

**Por quê:** a pasta `.git` guarda todo o histórico. Ela nunca é editada à mão; só o Git mexe nela.

**Resultado:** repositório criado, `git status` mostrando `No commits yet`.

## Passo 5 — README, `.gitignore` e primeiro commit

**Objetivo:** criar a capa do projeto e a proteção contra arquivos que não devem ir para o histórico.

Arquivos criados:

- **`README.md`:** a capa do projeto, primeira coisa que aparece no GitHub.
- **`.gitignore`:** ignora `.venv/`, `__pycache__/`, `*.pyc`, `node_modules/`, `dist/`, `.env`, `.vscode/` e `Thumbs.db`.

```powershell
git status
git add .
git status
git commit -m "chore: estrutura inicial do repositorio"
git log --oneline
```

**Por quê:**

- **`.gitignore` criado antes de tudo:** as pastas com bibliotecas instaladas são pesadas e podem ser recriadas; o `.env` vai guardar senhas. Criar a proteção antes evita que algo escape por acidente. A regra geral é: a lista de bibliotecas vai para o Git, as bibliotecas instaladas não.
- **Commit em duas etapas:** o `git add` escolhe o que entra na foto (área de preparação) e o `git commit` tira a foto.

**Resultado:** commit `b7a3bff` no branch `main`.

## Passo 6 — Publicar no GitHub

**Objetivo:** ter um backup na nuvem e a vitrine do portfólio.

Repositório criado no GitHub com o nome `pizzeria-oms`, **sem** README, `.gitignore` ou licença.

```powershell
git remote add origin https://github.com/SEU_USUARIO/pizzeria-oms.git
git push -u origin main
```

| Comando | O que faz |
|---|---|
| `git remote add origin ...` | Registra o endereço do GitHub com o apelido `origin` |
| `git push -u origin main` | Envia os commits e liga o `main` local ao do GitHub; o `-u` só é necessário na primeira vez |

**Por quê:** o repositório no GitHub foi criado vazio porque os arquivos já existiam localmente. Se o GitHub criasse os dele, os dois históricos começariam diferentes e o envio seria recusado.

**Resultado:** projeto publicado no GitHub.

## Passo 7 — Documentos de escopo e modelagem

**Objetivo:** guardar as decisões sobre a pizzaria junto com o código.

```powershell
mkdir docs
git add docs
git commit -m "docs: adiciona escopo v0.2 e modelagem do MVP"
git push
```

**Por quê:** quem lê o código precisa saber por que cada regra existe, e quando uma regra muda, a mudança fica registrada no histórico como qualquer outra. Os documentos vieram do projeto anterior e foram atualizados para a v0.2, com as regras RN-27 (dia de operação) e RN-28 (endereço e taxa congelados no pedido).

**Resultado:** `docs/escopo.md` e `docs/modelagem.md` no repositório. A partir daqui, o `git push` funciona sem argumentos, porque a ligação do passo 6 ficou guardada.

---

# Etapa 3 — Python local, ambiente virtual e HTTP

## Passo 8 — Criar e ativar a venv do backend

**Objetivo:** criar a caixa isolada onde vão ficar as bibliotecas Python do backend.

```powershell
mkdir backend
cd backend
python -m venv .venv
.\.venv\Scripts\Activate.ps1
(Get-Command python).Source
pip list
cd ..
git status
```

| Comando | O que faz |
|---|---|
| `python -m venv .venv` | Usa a ferramenta `venv` do Python para criar a caixa `.venv` |
| `.\.venv\Scripts\Activate.ps1` | Ativa a venv; aparece `(.venv)` no início da linha |
| `(Get-Command python).Source` | Mostra qual Python está em uso; com a venv ativa, é o de dentro da `.venv` |
| `pip list` | Lista as bibliotecas instaladas; numa venv nova, só aparece o `pip` |
| `git status` | Confirma que o Git ignora a `.venv` |

**Por quê:** a venv fica dentro de `backend/` porque pertence só ao backend; o frontend vai ter a própria caixa (`node_modules`). Ativar a venv faz o terminal usar o Python e as bibliotecas da caixa, e não os do Windows.

**Se a ativação for bloqueada:** o Windows bloqueia scripts por padrão. Liberar uma vez para o usuário:

```powershell
Set-ExecutionPolicy -Scope CurrentUser RemoteSigned
```

**Resultado:** em andamento.

---

# Problemas encontrados

| Passo | Problema | Causa | Solução |
|---|---|---|---|
| — | Nenhum até agora | — | — |
