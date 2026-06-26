---
name: auditing-design-system-compliance
description: Use when asked to review, audit, or check an Angular feature module/branch in the DemarcoDS / Bancodoc.Interface project for design-system consistency — toasts (incl. "salvo" form vs "inserido/editado" inline), modals (which component/pattern, incl. danger removal variant), button/action naming, form validation & duplicate-name messages, tables, breadcrumbs, form layout (header on edit, footer, gap-8/space-y-8 spacing), clickable-count→modal links (dmLinkButton), and tooltip-on-title vs alert — reporting what is out of standard with the correct pattern and the fix. Triggers on "está no padrão?", "fora do padrão", "padroniza", "revisa o módulo".
---

# Auditing Design-System Compliance (DemarcoDS)

## Overview

Audit a feature module against the project's Design System and reference screens, then report every deviation as **`fora do padrão` → `o padrão é` → `a correção`** (with `file:line` and severity). This skill is **DemarcoDS / `Bancodoc.Interface` (Angular/Nx)** specific.

**Core principle:** The standard is whatever `libs/ui` (the DS) and the mature reference apps already do — **`apps/empresas` (clientes)** and **`apps/servicos` (documentos)**. The audited module must match them, not reinvent its own components.

## How to run the audit

1. **Anchor the standard:** read the relevant DS component in `libs/ui` and how `empresas`/`servicos` use it. Never infer the standard from the module under review.
2. **Scan each dimension below** (grep the anti-patterns), across the module's pages, forms, shared dialogs.
3. **Report** a deviation table: `arquivo:linha · o que faz · o padrão · a correção · severidade`. Group by dimension. Confirm what IS compliant too.
4. Optionally verify fixes in the running app — see skill `verifying-ui-in-running-app`.

## The standards (what to check)

### 1. Toasts
- **Sucesso:** título `"[Entidade] [ação no particípio]!"`, **sem subtítulo** (a versão reduzida do toast é só título — omitir `message`).
- **Erro:** título `"Não foi possível [ação] o [item]"` + subtítulo com o erro real.
- **Info/bloqueio:** `"Sem permissão"` / título curto + subtítulo com o motivo.
- 🚩 Anti-padrões: títulos genéricos `"Sucesso"`/`"Erro"`; mensagens `"...com sucesso"`; **feedback de campo obrigatório em toast** (deve ser helper text no campo).
- Grep: `title: 'Sucesso'`, `title: 'Erro'`, `com sucesso`.

#### Vocabulário do toast de sucesso por comportamento
O particípio segue a **ação** e **concorda em gênero** com a entidade. Sempre termina com **"!"**.

> **Distinção crítica — "salvo" (formulário) vs "inserido/editado" (grid inline):**
> - **Salvar uma entidade via formulário** (botão "Salvar" de um form de página, handler único que faz create OU edit) → **`[Entidade] salvo!`** (ex.: `Grupo salvo!`, `Perfil salvo!`, `Usuário salvo!`). Referência: `empresas` (`clienteSalvo`, `usuarioSalvo`).
> - **CRUD inline em grid/lista** (adicionar/editar/renomear um item dentro de uma tabela, sem sair da tela) → **`inserido!/editado!/renomeado!`**. Referência: `servicos` (categorias, fundamentos).
> 🚩 Não use `inserido`/`editado`/`cadastrado` no toast de um **formulário** de entidade — ali é `salvo!`.

| Comportamento | Particípio (masc. / fem.) | Exemplos reais |
|---|---|---|
| **Salvar entidade (formulário)** | `salvo` / `salva` | "Grupo salvo!", "Perfil salvo!", "Usuário salvo!" |
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
   - **save de formulário de entidade** (handler único `save…`/`create…`+`update…` por trás do botão "Salvar" da página) → **`[Entidade] salvo!`**
   - cria registro novo **inline em grid** (`inserir…` numa tabela, sem sair da tela) → **inserção** → `inserido/inserida`
   - atualiza registro existente (PUT/PATCH / `update…` / `editar…`) → **edição** → `editado/editada`
   - altera **só o nome** (rename inline / `renomear…`) → **renomeação** → `renomeado/renomeada`
   - associa itens entre entidades (vincular / `vincular…` / `link…`) → **vinculação** → `vinculado/vinculada`
   - remove registro (DELETE / `excluir…` / `remover…`) → **exclusão** → `excluído/excluída`
   - gera/baixa arquivo (export / download) → **exportação** → `Arquivo exportado!`
3. **Determine o gênero do substantivo-núcleo da entidade** (o primeiro substantivo do nome, ignorando complementos): *Categoria* de dívida → **feminino** (`a`); *Tipo* de documento → **masculino** (`o`); *Orientação* geral → feminino; *Fundamento* de identificação → masculino. Concorde o particípio com esse núcleo.
4. **Decida singular vs. plural** pela quantidade que a ação atinge: uma → singular; várias → plural (`Pesquisas vinculadas!`).
5. **Monte o título** `"[Entidade] [particípio]!"` e compare com o atual. Reporte como desvio se: verbo errado para o comportamento (ex.: `salvo`/`cadastrado` numa edição), gênero/plural trocado, ou falta o `!`.

> ⚠️ Conflito comum: `cadastrado` genérico está errado. Use **`salvo!`** no toast de um **formulário** de entidade e **`inserido!/editado!/renomeado!`** apenas para **CRUD inline em grid** (ver distinção acima).

### 2. Modais — usar o componente certo do DS, não rolar o próprio
- **Confirmar exclusão (sem vínculo):** título **"Atenção!"** vermelho + ícone `trash` (`bg-error-100`/`text-error-500`) + texto de ação irreversível + `[Cancelar] [Confirmar]`.
- **Remover vínculo / excluir registro (variante `danger`):** mesmo visual **"Atenção!"** vermelho + `trash` + `bg-error-100` + `[Cancelar] [Confirmar]`. Texto no padrão do DS: **`Ao clicar em "Confirmar", você concorda que [o item] vinculado será removido [do contexto].`** (concordância de gênero/número). Num dialog genérico parametrizável, exponha uma variante `danger` em vez de criar outro componente.
- **Mudança de status (desativar/ativar):** confirmação **neutra** (ícone `warning`, `dm-dialog-alert-template`), **não** `danger` — vermelho/trash é só para **remoção/exclusão**.
- **Bloqueio por vínculo:** **"Atenção!"** + **`dm-data-table`** paginada com os itens vinculados + só `[Voltar]`.
- **Sair sem salvar:** usar **`ExitConfirmationDialogComponent`** do DS (`@bancodoc/ui`) — primária = "Continuar editando".
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

## Output format

> **🔴/🟡/🟢 [Dimensão] — `arquivo:linha`**
> Faz: `<o que está fora>` · Padrão: `<o correto, com a referência no libs/ui ou empresas/servicos>` · Correção: `<a mudança>`

Sempre listar também o que **já está conforme**, para dar confiança.

## Common mistakes when auditing
- Inferir o "padrão" a partir do próprio módulo auditado → sempre ancorar em `libs/ui` + `empresas`/`servicos`.
- Marcar como desvio algo que é decisão de produto (ex.: grupo permite "Excluir mesmo assim") → separar **inconsistência de DS** de **regra de negócio**.
- Esquecer de varrer os **dialogs compartilhados** (`shared/ui/components`), não só as páginas.
