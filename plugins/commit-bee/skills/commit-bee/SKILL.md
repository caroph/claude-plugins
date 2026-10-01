---
name: commit-bee
description: Gera e realiza um commit seguindo o Guia do Commit Amigão da bee-stylish (padrão Karma Commit Messages). Use quando o usuário pedir para commitar, gerar mensagem de commit ou rodar "commit bee".
---

# Commit Bee-Stylish

Gere e realize um commit seguindo o **Guia do Commit Amigão** da bee-stylish (padrão Karma Commit Messages).

## Argumentos

`$ARGUMENTS` — número ou identificador do card (ex: `158720` ou `GESTAODEV-5030`). Opcional. É sempre usado com `#` no rodapé.

---

## Passo 1 — Coletar o contexto

Execute em paralelo:

1. `git branch --show-current` — verificar a branch atual
2. `git status` — ver arquivos modificados/staged
3. `git diff --cached` — ver o que está staged
4. `git diff` — ver o que não está staged
5. `git log --oneline -10` — entender o estilo de mensagens do projeto

Se a branch atual for `main` ou `master`, **pare imediatamente** e avise o usuário: *"Não é permitido commitar diretamente em `main`/`master` (regra da organização). Crie e mude para uma branch antes de continuar."* Não prossiga para os próximos passos.

Se o `git status` estiver limpo (nada a commitar), informe ao usuário e encerre.

## Passo 2 — Determinar o card

- Se `$ARGUMENTS` contiver um número ou identificador de card, use-o.
- Se não foi informado, **pergunte ao usuário**: *"Deseja informar o número do card para referenciá-lo no rodapé (`Refs #CARD`, e `Closes #CARD` no último commit)? (informe o número ou pressione Enter para pular)"*

## Passo 3 — Analisar as mudanças e montar a mensagem

A mensagem segue a anatomia:

```
<tipo>(<escopo>): <assunto>

<corpo>

<rodapé>
```

### Assunto (obrigatório)
- Formato: `<tipo>(<escopo>): <assunto>`
- **Máximo 50 caracteres**
- Tipo e escopo em **minúsculas**
- Assunto no **imperativo, em português** (ex: "adiciona endpoint de telemetria")
- Tipos permitidos:
  - `feat` — nova funcionalidade
  - `fix` — correção de bug
  - `refactor` — refatoração sem mudança de comportamento
  - `test` — adição ou correção de testes
  - `docs` — documentação
  - `style` — formatação, sem mudança de lógica
  - `chore` — tarefas de manutenção (build, deps, config)
- Escopo é opcional; use quando ajudar a localizar a mudança (ex: `cache`, `repository`, `health`)

### Corpo (quando necessário)
- **Máximo 80 caracteres por linha**
- Explica o **quê** e o **por quê**, não o **como**
- Deixe uma linha em branco entre o assunto e o corpo
- Omita se a mudança for trivial e o assunto for autoexplicativo

### Rodapé (quando houver card)
- Deixe uma linha em branco após o corpo (ou após o assunto, se não houver corpo)
- **Todo commit referencia o card**: `Refs #<CARD>`
- O **último commit** da série (o que encerra o trabalho do card) usa `Closes #<CARD>` **no lugar** do `Refs`
- `Closes` aparece em **um único commit** da série; os demais usam `Refs`
- Ao criar vários commits parciais de uma vez, todos recebem `Refs` e só o último recebe `Closes`
- Se não estiver claro qual é o último commit do card, use `Refs` e pergunte ao usuário se deve fechar o card

## Exemplos válidos

```
fix(repository): corrige timeout na conexão com Firebird

A query de verificação excedia o limite configurado no pool.
Ajustado o CommandTimeout para 30 segundos.

Closes #158720
```

```
feat(cache): adiciona cache de motoristas com refresh periódico

Closes #159324
```

```
refactor(cache): extrai chave de cache de motoristas

Refs #159324
```

```
chore: atualiza dependências do projeto
```

## Passo 4 — Confirmar e commitar

A confirmação deve seguir esta ordem obrigatória — **não pule etapas nem agrupe perguntas**:

1. Exiba a mensagem proposta **sem o rodapé de card** se o card ainda não foi resolvido no Passo 2.
2. **Aguarde** a resposta do usuário sobre o card antes de qualquer outra coisa.
3. Com o card definido (ou descartado), exiba a mensagem final completa.
4. Pergunte: **"Confirma o commit com essa mensagem? (s/n)"** e **aguarde** a resposta.
5. Somente após o "s" do usuário:
   - Se não houver nada staged (`git diff --cached` vazio), pergunte ao usuário quais arquivos deseja adicionar ou se deseja fazer `git add .`.
   - Crie o commit com a mensagem exata usando heredoc para preservar a formatação.
6. Se negado, peça os ajustes desejados e repita a partir do Passo 3.

## Regras

- Idioma: **português brasileiro**.
- **Nunca commitar nas branches `main` ou `master`** (regra da organização) — verificação feita no Passo 1.
- Nunca use `--no-verify`.
- Nunca use `--amend` a menos que o usuário peça explicitamente.
- Nunca faça `git push` automaticamente — apenas o commit local.
- Não inclua arquivos sensíveis (`.env`, credenciais). Alerte o usuário se detectá-los staged.
- Não adicione `Co-Authored-By` nem qualquer rodapé além de `Refs #CARD` / `Closes #CARD`.
