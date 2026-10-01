---
name: novo-projeto
description: Cria a estrutura inicial de um novo projeto seguindo os padrões internos da empresa (back, front e branding, a partir de repositórios-template configuráveis no Azure DevOps). Use quando o usuário pedir para criar um novo projeto, inicializar um repositório novo ou fazer scaffold de projeto.
---

# Novo Projeto (padrão interno)

Monta a estrutura inicial de um projeto novo usando como referência repositórios-template da
empresa no Azure DevOps, sempre com as versões estáveis mais recentes de framework/linguagem — os
templates costumam ter versões antigas fixadas, que **não** devem ser copiadas.

Requer o MCP `azure-devops-official` conectado.

Esta skill é distribuída publicamente e por isso **não contém nomes de organização, projeto ou
repositório** — essas coordenadas são específicas de cada empresa que a usa e devem vir de
configuração, nunca ficar hardcoded aqui.

## Passo 0 — Resolver a configuração dos templates

Antes de tudo, descubra os repositórios-template a usar, nesta ordem de prioridade:

1. **Variáveis de ambiente**, se definidas:
   - `NOVO_PROJETO_ADO_ORG` — organização do Azure DevOps.
   - `NOVO_PROJETO_BACK_PROJETO` / `NOVO_PROJETO_BACK_REPO` — template de back.
   - `NOVO_PROJETO_FRONT_REACT_PROJETO` / `NOVO_PROJETO_FRONT_REACT_REPO` — template de front React/Next.js.
   - `NOVO_PROJETO_FRONT_ANGULAR_PROJETO` / `NOVO_PROJETO_FRONT_ANGULAR_REPO` (e opcionalmente
     `NOVO_PROJETO_FRONT_ANGULAR_BRANCH`, se o conteúdo não estiver na branch padrão do repositório).
   - `NOVO_PROJETO_BRANDING_PROJETO` / `NOVO_PROJETO_BRANDING_REPO` / `NOVO_PROJETO_BRANDING_PATH`
     — repositório e pasta de onde vem o manual de marca/branding mais atualizado da empresa.
2. **Instrução equivalente no `CLAUDE.md`/managed settings da organização** (arquivo privado, não
   publicado com esta skill) — procure por uma seção que defina esses mesmos valores.
3. Se nada estiver configurado, **pergunte ao usuário** os valores acima e sugira que ele os registre
   em um desses dois lugares para não precisar informar de novo nas próximas execuções.

Em nenhum caso grave esses valores de volta nos arquivos desta skill/plugin.

Antes de ler qualquer repositório-template, descubra a branch padrão real com `repo_repository`
(action `get`, campo `defaultBranch`) em vez de assumir `main` ou `master` — alguns repositórios têm
o conteúdo relevante numa branch diferente da padrão (uma variável de ambiente/config pode indicar
isso explicitamente, como em `NOVO_PROJETO_FRONT_ANGULAR_BRANCH`).

## Passo 1 — Perguntas (nunca assumir)

Pergunte ao usuário, nessa ordem:

1. **Nome do projeto**.
2. **Vai ter back, front, ou os dois?**
3. Se tiver back: quais camadas de persistência ele precisa e se vai usar mensageria/fila e um
   processo worker separado — não assuma que todo projeto novo precisa de tudo que o template de
   back tem.
4. Se tiver front: **qual stack** — React/Next.js ou Angular (ou outra, se a empresa tiver mais de
   um template configurado).
5. Se a stack pedida não tiver um template configurado: avise que só a parte agnóstica do padrão
   será aplicada (`.specs/`, `.editorconfig` genérico, estrutura de pipeline como referência) e
   pergunte o que ele quer aproveitar mesmo assim.
6. **Pasta de destino local** onde o projeto será criado.

## Passo 2 — Gerar o esqueleto com as ferramentas oficiais

Não copie `csproj`, `angular.json`, `package.json`, `tsconfig*.json` etc. literalmente dos
templates — eles costumam fixar versões antigas. Gere o esqueleto com o CLI oficial, na versão
estável mais recente disponível:

- Back .NET: `dotnet new sln`, depois um projeto por camada definida no Passo 1 (`dotnet new webapi`
  para a Api, `dotnet new classlib` para as demais camadas), seguindo a nomenclatura
  `<NomeProjeto>.<Camada>` observada no template de back.
- Front React/Next.js: `npx create-next-app@latest` com as flags equivalentes ao observado no
  template (app router, TypeScript, Tailwind, etc. — confira o que o template realmente usa).
- Front Angular: `ng new` (Angular CLI mais recente) com as flags observadas no template (SCSS,
  routing, etc.).

## Passo 3 — Replicar a estrutura de pastas e convenções (sem código de domínio)

Depois do scaffold oficial, leia o template correspondente (Passo 0) e recrie as pastas de mais alto
nível observadas — **vazias ou só com o encaixe (interfaces, módulos vazios), nunca com a lógica de
negócio real do template**. Observe no template de back se ele segue camadas simples (Api/Domain/
Repository) ou um padrão CQRS (Commands/Handlers) com mensageria, e replique só o que o usuário
confirmou precisar no Passo 1. Para o front, observe a separação típica (ex.: núcleo/core,
compartilhado/shared, módulos ou páginas) e replique a mesma divisão.

## Passo 4 — Config e infra (sempre, adaptando nomes)

- `.editorconfig`, `.gitignore`, `.gitattributes` — gerados pelo scaffold oficial; complemente com
  qualquer regra adicional vista no template que faça sentido.
- `Dockerfile` + scripts de build (ex.: config de nginx/entrypoint) do template de front escolhido,
  se existirem.
- Pipeline de CI/CD: use como modelo o pipeline do template (Azure Pipelines ou equivalente),
  adaptando nomes de solução/projeto. Se o template de back e o de front tiverem pipelines
  separados por camada, mantenha essa separação.
- `.specs/` vazio, pronto para a skill `tlc-spec-driven` (`project/`, `codebase/`, `features/`,
  `quick/`).
- `CLAUDE.md` novo, cobrindo: descrição do projeto (preencher com o usuário), stack, estrutura,
  fluxo spec-driven obrigatório e a regra de nunca commitar em `main`/`master`.
- `README.md` básico.

## Passo 5 — Branding (somente se houver front)

Copie sempre da fonte de branding configurada no Passo 0 (normalmente o projeto mais recente/
atualizado da empresa, não necessariamente o próprio repositório-template de front, que pode estar
desatualizado nesse quesito): manual da marca, logos, fontes. No `CLAUDE.md` do novo projeto, inclua
a obrigatoriedade de seguir o manual da marca em toda tela/PDF/documento gerado.

## Passo 6 — Componentes compartilhados reaproveitáveis

Liste para o usuário quais componentes do `shared`/`components` do template de front escolhido
parecem reaproveitáveis (ex.: grid de dados, diálogo de confirmação, tabela dinâmica, formulários,
modais, cabeçalho) e pergunte se ele quer portar algum agora ou só deixar anotado como ideia futura
em `.specs/project/STATE.md`.

## Passo 7 — Git

Siga a regra da organização: **nunca commitar em `main`/`master`**. Rode `git init`, crie uma branch
(ex: `feat/estrutura-inicial`) antes de qualquer commit, e só commite com confirmação do usuário —
preferencialmente invocando a skill `commit-bee`.

## Regras

- Nunca fixar as versões antigas de framework/lib vistas nos templates — gere sempre via CLI oficial
  na versão estável mais recente no momento da criação.
- Nunca copiar código de domínio/negócio dos templates (Commands, Handlers, Controllers, Models,
  componentes de tela específicos) — só a estrutura de pastas e os arquivos de infraestrutura/config.
- Nunca hardcoded nomes de organização, projeto ou repositório nesta skill — sempre via Passo 0.
- Sempre perguntar antes de assumir: nome do projeto, back/front, stack de front, camadas de
  persistência e mensageria necessárias.
- Antes de ler um repositório-template, resolva a branch padrão real via `repo_repository` (action
  `get`) — não assuma `main` nem `master`.
- Idioma: português do Brasil, em código, comentários e documentação gerados.
