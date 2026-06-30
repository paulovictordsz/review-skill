---
name: review-skill
description: Use when asked to review, audit, or check an Angular feature module/branch in the DemarcoDS / Bancodoc.Interface project — both (A) for design-system consistency in the code (toasts incl. "salvo" form vs "inserido/editado" inline, modals incl. danger removal variant, button/action naming, form validation & duplicate-name messages, tables, breadcrumbs, form layout, clickable-count→modal links, tooltip-vs-alert), and (B) verifying the change in the running app (stale bundles, change-detection zones, mock-API signals). Reports what is out of standard with the correct pattern and the fix. Triggers on "está no padrão?", "fora do padrão", "padroniza", "revisa o módulo", "review em funcionamento/componentes".
---

# DemarcoDS Review (review-skill)

## Overview

One skill, two complementary jobs on the same feature module/branch:

- **Part A — Audit DS compliance (code):** check a feature module against the project's Design System and reference screens, reporting every deviation as **`fora do padrão` → `o padrão é` → `a correção`** (with `file:line` and severity).
- **Part B — Verify in the running app:** confirm a UI change actually landed and behaves correctly in a running hot-reload dev server (often with a mock API like MSW), seeing past stale bundles, change-detection traps, and misleading mock-API signals.

Use Part A to find/fix deviations; use Part B to prove the fix is real in the browser. This skill is **DemarcoDS / `Bancodoc.Interface` (Angular/Nx)** specific.

**Core principle (A):** The standard is whatever `libs/ui` (the DS) and the mature reference apps already do — **`apps/empresas` (clientes)** and **`apps/servicos` (documentos)**. The audited module must match them, not reinvent its own components.

**Core principle (B):** Confirm the build, drive with *trusted* events, and read *framework state* — not just the DOM.

---

# Part A — Auditing Design-System Compliance

## How to run the audit

1. **Anchor the standard:** read the relevant DS component in `libs/ui` and how `empresas`/`servicos` use it. Never infer the standard from the module under review.
2. **Scan each dimension below** (grep the anti-patterns), across the module's pages, forms, shared dialogs.
3. **Report** a deviation table: `arquivo:linha · o que faz · o padrão · a correção · severidade`. Group by dimension. Confirm what IS compliant too.
4. Optionally verify fixes in the running app — see **Part B** below.

## The standards (what to check)

### 1. Toasts
- **Sucesso:** título `"[Entidade] [ação no particípio]!"`, **sem subtítulo** (a versão reduzida do toast é só título — omitir `message`).
- **Erro:** título `"Não foi possível [ação] o [item]"` + subtítulo com o erro real.
- **Info/bloqueio:** `"Sem permissão"` / título curto + subtítulo com o motivo.
- 🚩 Anti-padrões: títulos genéricos `"Sucesso"`/`"Erro"`; mensagens `"...com sucesso"`; **feedback de campo obrigatório em toast** (deve ser helper text no campo).
- Grep: `title: 'Sucesso'`, `title: 'Erro'`, `com sucesso`.

#### Vocabulário do toast de sucesso por comportamento
O particípio segue a **ação** e **concorda em gênero** com a entidade. Sempre termina com **"!"**.

> **Distinção crítica do toast de "Salvar" — depende de criar vs. editar:**
> - **Salvar EDIÇÃO de uma entidade já cadastrada** (tela dedicada de *editar/gerenciar*, cadastro já concluído; botão "Salvar"/"Salvar alterações" no modo gerenciamento) → **SEMPRE `Alterações salvas!`** (texto fixo, genérico — NÃO a entidade). Este é o padrão para salvar edições.
> - **Criar uma entidade nova** (cadastro/wizard concluído com sucesso) → **`[Entidade] cadastrado!`** (ex.: `Fornecedor cadastrado!`, `Cliente cadastrado!`).
> - **Form unificado create+edit numa única tela** (a mesma tela faz os dois, sem separação clara) → **`[Entidade] salvo!`** (ex.: `Grupo salvo!`, `Perfil salvo!`, `Usuário salvo!`). Referência: `usuarios`.
> - **CRUD inline em grid/lista** (adicionar/editar/renomear um item dentro de uma tabela, sem sair da tela) → **`inserido!/editado!/renomeado!`**. Referência: `servicos` (categorias, fundamentos).
> 🚩 Numa tela de **gerenciar/editar** registro existente, NÃO use `[Entidade] salvo!` nem `editado!` — o correto é **`Alterações salvas!`**.

| Comportamento | Toast | Exemplos reais |
|---|---|---|
| **Salvar edição (tela gerenciar/editar)** | `Alterações salvas!` (fixo) | "Alterações salvas!" (gerenciar fornecedor/cliente) |
| **Criar entidade (cadastro novo)** | `[Entidade] cadastrado/a!` | "Fornecedor cadastrado!", "Cliente cadastrado!" |
| **Salvar entidade (form unificado create+edit)** | `salvo` / `salva` | "Grupo salvo!", "Perfil salvo!", "Usuário salvo!" |
| **Inserção inline (grid)** | `inserido` / `inserida` | "Tipo de documento inserido!", "Orientação geral inserida!", "Categoria de dívida inserida!", "Cargo para análise inserido!" |
| **Edição** | `editado` / `editada` | "Tipo de critério editado!", "Categoria de motivo editada!", "Fundamento de identificação editado!" |
| **Exclusão** | `excluído` / `excluída` | "Tipo de uso excluído!", "Justificativa de motivo excluída!", "Cargo para análise excluído!" |
| **Renomeação** | `renomeado` / `renomeada` | "Tipo de orientação renomeado!", "Categoria de procedimento renomeada!" |
| **Vinculação** | `vinculado` / `vinculada` (+ plural) | "Pesquisa vinculada!", "Pesquisas vinculadas!", "Regra vinculada!", "Regras vinculadas!" |
| **Exportação** | (entidade fixa) | "Arquivo exportado!" |

Regras de redação:
- **Concordância de gênero** com o substantivo da entidade: *Categoria…* → `inserida/editada/excluída/renomeada`; *Tipo…/Fundamento…/Momento…/Cargo…* → `inserido/editado/excluído/renomeado`.
- **Plural** quando a ação atinge vários itens: `Pesquisas vinculadas!`, `Regras vinculadas!`.
- 🚩 Anti-padrões adicionais: usar `cadastrado`/`salvo` genérico quando o comportamento é inserir/editar/renomear; faltar o `!` final; gênero trocado ("Categoria … inserido!"); plural errado.

#### Como auditar toasts de sucesso (passo a passo)
Não basta olhar o texto isolado — o particípio correto depende da **ação** que o handler executa. Para cada toast:

1. **Encontre todos os toasts de sucesso.** Grep: `toastService.success(` (e variações `.success({`). Audite **cada ocorrência**, não só as que "parecem erradas".
2. **Classifique o comportamento pela ação do handler** (leia o método que chama o toast — não adivinhe pelo texto atual):
   - **salvar EDIÇÃO** numa tela de gerenciar/editar registro existente (rota `/:id`, modo edição, PATCH/update) → **`Alterações salvas!`** (fixo)
   - **criar entidade nova** (cadastro/wizard, POST/create) → **`[Entidade] cadastrado!`**
   - **save de form unificado create+edit** (uma tela só faz os dois) → **`[Entidade] salvo!`**
   - cria registro novo **inline em grid** (`inserir…` numa tabela, sem sair da tela) → **inserção** → `inserido/inserida`
   - atualiza registro existente (PUT/PATCH / `update…` / `editar…`) → **edição** → `editado/editada`
   - altera **só o nome** (rename inline / `renomear…`) → **renomeação** → `renomeado/renomeada`
   - associa itens entre entidades (vincular / `vincular…` / `link…`) → **vinculação** → `vinculado/vinculada`
   - remove registro (DELETE / `excluir…` / `remover…`) → **exclusão** → `excluído/excluída`
   - gera/baixa arquivo (export / download) → **exportação** → `Arquivo exportado!`
3. **Determine o gênero do substantivo-núcleo da entidade** (o primeiro substantivo do nome, ignorando complementos): *Categoria* de dívida → **feminino** (`a`); *Tipo* de documento → **masculino** (`o`); *Orientação* geral → feminino; *Fundamento* de identificação → masculino. Concorde o particípio com esse núcleo.
4. **Decida singular vs. plural** pela quantidade que a ação atinge: uma → singular; várias → plural (`Pesquisas vinculadas!`).
5. **Monte o título** `"[Entidade] [particípio]!"` e compare com o atual. Reporte como desvio se: verbo errado para o comportamento (ex.: `salvo`/`cadastrado` numa edição), gênero/plural trocado, ou falta o `!`.

> ⚠️ Conflito comum: numa tela de **gerenciar/editar** registro existente, o save é **`Alterações salvas!`** (não `[Entidade] salvo!` nem `editado!`). `[Entidade] cadastrado!` é só para **criação**; `inserido!/editado!/renomeado!` só para **CRUD inline em grid** (ver distinção acima).

### 2. Modais — usar o componente certo do DS, não rolar o próprio
- **Confirmar exclusão (sem vínculo):** título **"Atenção!"** vermelho + ícone `trash` (`bg-error-100`/`text-error-500`) + texto de ação irreversível + `[Cancelar] [Confirmar]`.
- **Remover vínculo / excluir registro (variante `danger`):** mesmo visual **"Atenção!"** vermelho + `trash` + `bg-error-100` + `[Cancelar] [Confirmar]`. Texto no padrão do DS: **`Ao clicar em "Confirmar", você concorda que [o item] vinculado será removido [do contexto].`** (concordância de gênero/número). Num dialog genérico parametrizável, exponha uma variante `danger` em vez de criar outro componente.
- **Mudança de status (desativar/ativar):** confirmação **neutra** (ícone `warning`, `dm-dialog-alert-template`), **não** `danger` — vermelho/trash é só para **remoção/exclusão**.
- **Bloqueio por vínculo:** **"Atenção!"** + **`dm-data-table`** paginada com os itens vinculados + só `[Voltar]`.
- **Sair sem salvar:** usar **`ExitConfirmationDialogComponent`** do DS (`@bancodoc/ui`) — primária = "Continuar editando". 🚩 Atenção: ao remover o botão "Cancelar" do footer, garanta que o gatilho do modal continue ligado (ex.: `(backClick)` do `dm-page-header` chamando `handleCancel()`), senão o modal fica órfão e a saída navega direto sem confirmar.
- **Confirmação genérica:** `dm-dialog-alert-template`, dialog `size="md"`, botões `size="sm"`.
- **Vinculação:** `dm-data-table` + busca + **alerta** `notify`/`bell` ("É necessário selecionar pelo menos um X para salvar a vinculação.") + botão **"Salvar" sempre ativo**.
- 🚩 Anti-padrões: `<table>` HTML cru dentro do dialog; modal próprio em vez do componente do DS; botão de vinculação `[disabled]` / rotulado "Vincular X"; alerta removido.

### 3. Botões, ações e nomenclatura
- Gerenciar um registro = ação única **"Gerenciar"** (ícone `gear`) — **não** "Visualizar"/"Editar" separados nem rota `/editar`. Somente-leitura é definido por **permissão**.
- Botão de confirmar em **modal de vinculação** = **"Salvar"**.
- Botão de salvar de **formulário** = sempre **"Salvar"** (não "Cadastrar"/"Enviar"), tanto em criação quanto em edição.
- **Footer do formulário:** `border-t border-neutral-200 bg-neutral-50 rounded-b-lg`, alinhado à direita, **só o botão primário "Salvar"** no modo edição/criação (sem "Cancelar"/"Voltar" — voltar é pelo header). Referência: `servicos`/`programas`.
- **Link clicável** (contagem → modal, "ver detalhes", etc.) = diretiva **`dmLinkButton`** do DS (cor `text-secondary-700`) — **não** `text-primary-500`, nem cor hardcoded `text-[#0075FF]`, nem `hover:underline` custom.
- 🚩 Anti-padrões: ações "Visualizar"+"Editar"; ícone `eye`; botão "Vincular X"; botão "Cadastrar"/"Enviar" em form; "Cancelar" no footer.

### 4. Validação de formulário
- **Required** = **`DmValidators.required`** (trata espaço em branco como vazio) — não `Validators.required` nativo.
- Ao submeter inválido: util **`markAllAsTouched`** do DS (emite `statusChanges` → renderiza erro sob OnPush + scroll-to-invalid) — não o `form.markAllAsTouched()` nativo.
- Mensagem padrão no campo: **"Campo obrigatório."**.
- **Tabela obrigatória:** erro **"Campo obrigatório." ABAIXO da tabela** (não acima, não em toast).
- Erro de servidor (duplicado 409) → **`setErrors({ duplicate: true })`** no campo (helper text via `customErrors`), nunca toast. Mensagem de **nome duplicado**: **"Já existe um item com este nome cadastrado."** (genérico "item", não a entidade). E-mail inválido: **"Insira um e-mail válido"**; e-mail duplicado: **"Já existe um usuário com este e-mail cadastrado."**.
- Grep: `Validators.required`, `this.form.markAllAsTouched(`.

### 5. Tabelas
- Título da listagem: **"[Item]s cadastrados"** (ex.: "Grupos cadastrados").
- `pageSize` padrão **10** (10, 20, 30…).
- Empty state padrão do DS: **"Ops, não existem registros cadastrados!"**.
- Usar **`dm-data-table`** do DS.
- Grep: `signal(50)`, `signal(20)` em pageSize.

### 6. Navegação / textos
- Breadcrumb começa pelo módulo (ex.: **"Empresa"**), com chaves i18n quando o resto do app é traduzível.
- Textos em PT com **acentuação** correta (inclusive seeds da fake-api).

### 7. Layout dos formulários (página de cadastro/edição)
- **Cabeçalho** (`dm-page-header` com botão voltar + título) aparece em **criação E edição** — `Novo X` ao criar, `Gerenciar X` ao editar. 🚩 Anti-padrão: header dentro de `@if (isCreateMode)` → some na edição (botão voltar + título desaparecem).
- **Espaçamento:** grids de campos e seções usam **`gap-8` / `space-y-8`** (32px), padrão do cadastro de programas. 🚩 `gap-4`/`space-y-4` (16px) está fora do padrão. (Não confundir com `space-y-1` de label/hint.)
- **Footer:** ver dimensão 3 (`bg-neutral-50`, só "Salvar").
- Grep: `@if (isCreateMode)` em volta de `dm-page-header`; `grid gap-4`, `space-y-4` nos forms.

### 8. Contagem clicável → modal com os itens
- Em listagens/tabelas (**exceto** tabelas dentro de modais), toda **célula que mostra um número de itens vinculados** (Unidades, Usuários, Grupos, Perfis…) deve ser um **link clicável** que abre um modal listando os itens.
- Link = **`dmLinkButton`** (cor `text-secondary-700`) + ícone **`arrow-line-up-right`** (`source="ph" [size]="16"`); renderiza só quando `> 0` (senão "0" puro).
- Modal = `dm-data-table` paginada (título + subtítulo = nome do registro) + `[Cancelar]`. Um único dialog genérico reutilizável (ex.: `RelatedItemsDialogComponent`) atende todas as colunas. Referência: `programas` (consulta → "Unidades vinculadas").
- Dados: resolva ids→nomes pelas listas-mestre do store quando a linha já traz os ids; quando a linha só tem a **contagem** (sem ids), crie endpoint/fetch para buscar os itens.
- 🚩 Anti-padrões: número como texto puro não-clicável; link com cor própria (`text-primary-500`, `text-[#0075FF]`) em vez de `dmLinkButton`/`secondary-700`.

### 9. Info contextual: tooltip no título, não alert
- Informação de apoio **não-obrigatória** ao lado de um título de seção → **tooltip** no padrão DS: `<div [dmTooltip]="…" tooltipPosition="right"><dm-icon name="info" source="ph" [size]="16"></dm-icon></div>` — **não** um `dm-alert` ocupando espaço fixo. Referência: `programas` ("Conteúdo do programa ⓘ").
- `dm-alert` (`notify`/`bell`) permanece para **ação requerida** (ex.: alerta dentro do modal de vinculação: "É necessário selecionar pelo menos um X…").

## Output format (Part A)

> **🔴/🟡/🟢 [Dimensão] — `arquivo:linha`**
> Faz: `<o que está fora>` · Padrão: `<o correto, com a referência no libs/ui ou empresas/servicos>` · Correção: `<a mudança>`

Sempre listar também o que **já está conforme**, para dar confiança.

## Common mistakes when auditing
- Inferir o "padrão" a partir do próprio módulo auditado → sempre ancorar em `libs/ui` + `empresas`/`servicos`.
- Marcar como desvio algo que é decisão de produto (ex.: grupo permite "Excluir mesmo assim") → separar **inconsistência de DS** de **regra de negócio**.
- Esquecer de varrer os **dialogs compartilhados** (`shared/ui/components`), não só as páginas.

---

# Part B — Verifying UI Changes in a Running App

When you verify a UI change in a running app, **two layers can lie to you**: the dev server (may serve a *stale* bundle) and your automation harness (may not drive the framework's reactivity). Don't trust "it looks broken" or "it looks fixed" until you've ruled both out.

## When to use Part B
- You changed template/form/component code and "it doesn't appear" in the running app.
- Your Puppeteer/Playwright check gives a result that contradicts the code.
- App uses a watch/HMR dev server and/or a mock API (MSW, mirage, etc.).

## The traps (the meat)

| Trap | Symptom | Fix |
|---|---|---|
| **Stale bundle from a compile error** | App returns 200 and "looks up", but your change isn't there; tests reflect old behavior | A TS/template error makes the watch server keep serving the **last good bundle**. **Run the real build** (`nx build <proj>`, `nx run <proj>:typecheck`, `tsc --noEmit`) to surface the error. **"Server responds 200" is NOT proof your code compiled.** Fix, rebuild, then hard-reload. |
| **`evaluate` runs OUTSIDE the framework's change-detection / zone** | Synthetic `el.value=…; dispatchEvent(new Event('input'))` or `el.click()` change the DOM but nothing re-renders | Synthetic events in `page.evaluate` don't tick CD or update the form model. **Drive with trusted events**: real `page.click()` / `page.type()` / `fill`. Use `evaluate` only to **read state** or `fetch`. |
| **Asserting on the DOM only** | Control is invalid but no error shows; value "didn't change" | Read **framework state**: in Angular dev mode `window.ng.getComponent(el)` → `control.value/valid/errors/touched`, signals. The DOM can lag a CD tick. |
| **Mock-API signals mislead** | `fetch(endpoint)` → 200, so "it works" | 200 only proves the worker intercepts — not that *your scenario* passes. The in-memory DB usually **resets on full reload** (records created via API vanish). Use a deterministic seed (`faker.seed(n)`) so data is stable across reloads. |

## Angular OnPush form-error gotchas
- `Validators.required` treats whitespace `'  '` as **valid** → use a **trimming** required validator (DS `DmValidators.required`) for true "required".
- A form-field renders errors off the control's **`statusChanges`**. `markAllAsTouched()` does **not** emit `statusChanges`, so under OnPush the error won't render on submit. Use the DS `markAllAsTouched` util that also calls `updateValueAndValidity()` (emits the event). `setErrors()` *does* emit — use it for server/duplicate errors so the field renders.
- A conditionally-projected hint (`ContentChild`) may not render under OnPush → render it as a plain **sibling** element instead.

## Red flags — stop and check
- "The dev server is up (200)" → **did you actually build?**
- "My puppeteer click did nothing" → trusted event vs `evaluate`.
- "Control is invalid but no error shows" → `statusChanges` / `markAllAsTouched` / build.
- "Created a record but it's gone after reload" → mock DB reset; seed it.

## Reliable verification loop
1. **Build** the affected project — catch compile errors that would freeze the bundle.
2. Wait for the dev server to recompile; **hard-reload** the page.
3. Drive the UI with **real** clicks/typing (trusted events).
4. Assert on **framework state** (`window.ng`) and the rendered DOM/screenshot.
5. For mock APIs, rely on **seeded, deterministic** data — not on freshly created records surviving a reload.

## DemarcoDS run notes
- Fake API: `npm run serve-fake -- dm` (MSW via `DM_FAKE_API=true`); app em `http://localhost:4200/plataforma/`. O MSW intercepta no browser — `curl` direto a `/api/*` retorna 404 (esperado, não é erro).
- Compile gate antes de confiar no que está na tela: `nx run <proj>:typecheck`.
