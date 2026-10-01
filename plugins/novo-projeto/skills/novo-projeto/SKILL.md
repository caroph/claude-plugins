---
name: novo-projeto
description: Cria a estrutura inicial de um novo projeto seguindo os padrões Inlog (back .NET baseado no Inlog.ProjetoBase.Back, front React/Next.js ou Angular baseado nos respectivos ProjetoBase, branding do Grupo Inlog vindo do Inlog.Relatorios). Use quando o usuário pedir para criar um novo projeto, inicializar um repositório novo ou fazer scaffold de projeto.
---

# Novo Projeto (padrão Inlog)

Monta a estrutura inicial de um projeto novo usando como referência os repositórios-base da empresa
no Azure DevOps, sempre com as versões estáveis mais recentes de framework/linguagem — os templates
têm versões antigas fixadas, que **não** devem ser copiadas.

Requer o MCP `azure-devops-official` conectado.

## Fontes de referência (Azure DevOps)

| Camada | Projeto | Repositório | Observação |
|---|---|---|---|
| Back | `Portfolio` | `Inlog.ProjetoBase.Back` | CQRS (Commands/Handlers), MassTransit/RabbitMQ, UoW, Firebird + Postgres |
| Front React/Next.js | `Portfolio` | `Inlog.ProjetoBase.Front` | Next.js (app router), Tailwind |
| Front Angular | `Portfolio` | `Inlog.ProjetoBase.Angular.Front` | **Atenção**: o conteúdo real está na branch `main`, não na branch padrão do repositório — resolva a branch com `repo_repository` (action `get`, campo `defaultBranch`) e, se a listagem vier vazia, tente explicitamente `main`. |
| Branding (sempre que houver front) | `Manutenção de Clientes` | `Inlog.Relatorios` | pasta `/util/branding` — é a versão mais atualizada do manual da marca, **nunca** usar a dos ProjetoBase |

Antes de ler qualquer repositório, descubra a branch padrão de verdade com
`repo_repository` (action `get`) em vez de assumir `main` ou `master`.

## Passo 1 — Perguntas (nunca assumir)

Pergunte ao usuário, nessa ordem:

1. **Nome do projeto** (ex: `Inlog.NomeDoProjeto`).
2. **Vai ter back, front, ou os dois?**
3. Se tiver back: quais camadas de persistência ele precisa (Firebird, Postgres, os dois) e se vai
   usar mensageria (RabbitMQ/MassTransit) e um projeto `Worker` separado — não assuma que todo
   projeto novo precisa de tudo que o `Inlog.ProjetoBase.Back` tem.
4. Se tiver front: **qual stack** — React/Next.js ou Angular.
5. Se a linguagem/stack pedida não for .NET nem React/Angular: avise que só a parte agnóstica do
   padrão será aplicada (`.specs/`, `.editorconfig` genérico, estrutura de pipeline como referência) e
   pergunte o que ele quer aproveitar mesmo assim.
6. **Pasta de destino local** onde o projeto será criado.

## Passo 2 — Gerar o esqueleto com as ferramentas oficiais

Não copie `csproj`, `angular.json`, `package.json`, `tsconfig*.json` etc. literalmente dos
templates — eles fixam versões antigas. Gere o esqueleto com o CLI oficial, na versão estável mais
recente disponível:

- Back: `dotnet new sln`, depois um projeto por camada definida no Passo 1 (`dotnet new webapi` para
  a Api, `dotnet new classlib` para Domain/Repositories/Worker), seguindo a nomenclatura
  `<NomeProjeto>.Api`, `<NomeProjeto>.Domain`, `<NomeProjeto>.DapperRepository.Firebird`,
  `<NomeProjeto>.EntityRepository.Postgres` etc., espelhando os nomes de projeto do
  `Inlog.ProjetoBase.Back`.
- Front React/Next.js: `npx create-next-app@latest` com as flags equivalentes ao observado no
  `Inlog.ProjetoBase.Front` (app router, TypeScript, Tailwind).
- Front Angular: `ng new` (Angular CLI mais recente) com as flags observadas no
  `Inlog.ProjetoBase.Angular.Front` (SCSS, routing).

## Passo 3 — Replicar a estrutura de pastas e convenções (sem código de domínio)

Depois do scaffold oficial, recrie as pastas vistas no template correspondente — **vazias ou só com
o encaixe (interfaces, módulos vazios), nunca com a lógica de negócio real do template**:

- **Back** (`Inlog.ProjetoBase.Back`): `Controllers/`, `Commands/`, `CommandHandlers/`, `Models/`,
  `Repositories/` (interfaces), `Dto/`, `Validators/`, `Ioc/`, `Constants/`; só crie
  `Options/`, `RabbitMQ/` ou `MassTransit/` e o projeto `Worker` se o usuário confirmou mensageria no
  Passo 1.
- **Front React/Next.js** (`Inlog.ProjetoBase.Front`): `app/(pages)/`, `components/`, `hooks/`,
  `context/`, `store/` (se for usar Redux), `types/`, `config/`, `constants/`, `lib/`.
- **Front Angular** (`Inlog.ProjetoBase.Angular.Front`, branch `main`): `core/` (`guards/`, `http/`,
  `interceptors/`, `services/`), `shared/` (`components/`, `models/`, `pipes/`).

## Passo 4 — Config e infra (sempre, adaptando nomes)

- `.editorconfig`, `.gitignore`, `.gitattributes` — gerados pelo scaffold oficial; complemente com
  qualquer regra adicional vista no template que faça sentido.
- `Dockerfile` + `build_files/` (`nginx.conf`, `entrypoint.sh`) do template de front escolhido.
- Pipeline Azure DevOps: use como modelo `azure-pipelines-back.yml` / `azure-pipelines-front.yml` do
  `Inlog.Relatorios` (pipelines separados por camada), adaptando nomes de solução/projeto.
- `.specs/` vazio, pronto para a skill `tlc-spec-driven` (`project/`, `codebase/`, `features/`,
  `quick/`).
- `CLAUDE.md` novo, cobrindo: descrição do projeto (preencher com o usuário), stack, estrutura,
  fluxo spec-driven obrigatório e a regra de nunca commitar em `main`/`master` — no mesmo formato do
  `CLAUDE.md` do `Inlog.Relatorios`.
- `README.md` básico.

## Passo 5 — Branding (somente se houver front)

Copie sempre de `Inlog.Relatorios/util/branding/` (é a fonte mais atualizada, **nunca** os
ProjetoBase): `MARCA.md`, `logos/`, `fonts/` e o PDF do manual da marca. No `CLAUDE.md` do novo
projeto, inclua a obrigatoriedade de seguir o manual da marca em toda tela/PDF/documento gerado,
como no `Inlog.Relatorios`.

## Passo 6 — Componentes shared reaproveitáveis

Liste para o usuário quais componentes do `shared`/`components` do template de front escolhido
parecem reaproveitáveis (ex.: `data-grid`, `confirmation-dialog`, `dynamic-table` no Angular;
componentes de `Forms`, `Modal`, `Header` no React) e pergunte se ele quer portar algum agora ou só
deixar anotado como ideia futura em `.specs/project/STATE.md`.

## Passo 7 — Git

Siga a regra da organização: **nunca commitar em `main`/`master`**. Rode `git init`, crie uma branch
(ex: `feat/estrutura-inicial`) antes de qualquer commit, e só commite com confirmação do usuário —
preferencialmente invocando a skill `commit-bee`.

## Regras

- Nunca fixar as versões antigas de framework/lib vistas nos templates — gere sempre via CLI oficial
  na versão estável mais recente no momento da criação.
- Nunca copiar código de domínio/negócio dos templates (Commands, Handlers, Controllers, Models,
  componentes de tela específicos) — só a estrutura de pastas e os arquivos de infraestrutura/config.
- Sempre perguntar antes de assumir: nome do projeto, back/front, stack de front, camadas de
  persistência e mensageria necessárias.
- Branding sempre vem do `Inlog.Relatorios`, nunca dos `ProjetoBase`.
- Antes de ler um repositório-template, resolva a branch padrão real via `repo_repository` (action
  `get`) — não assuma `main` nem `master`.
- Idioma: português do Brasil, em código, comentários e documentação gerados.
