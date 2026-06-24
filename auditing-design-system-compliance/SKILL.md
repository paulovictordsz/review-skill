---
name: auditing-design-system-compliance
description: Use when asked to review, audit, or check an Angular feature module/branch in the DemarcoDS / Bancodoc.Interface project for design-system consistency — toasts, modals (which component and which pattern), button/action naming, form validation, tables, breadcrumbs — and to report what is out of standard with the correct pattern and the fix. Triggers on "está no padrão?", "fora do padrão", "padroniza", "revisa o módulo".
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
- **Sucesso:** título `"[Item] [ação no particípio]"`, **sem subtítulo** (a versão reduzida do toast é só título — omitir `message`). Ex.: `Usuário cadastrado`, `Perfil salvo`, `Grupo excluído`, `Convite reenviado`, `Perfil vinculado`/`Perfis vinculados`.
- **Erro:** título `"Não foi possível [ação] o [item]"` + subtítulo com o erro real.
- **Info/bloqueio:** `"Sem permissão"` / título curto + subtítulo com o motivo.
- 🚩 Anti-padrões: títulos genéricos `"Sucesso"`/`"Erro"`; mensagens `"...com sucesso"`; **feedback de campo obrigatório em toast** (deve ser helper text no campo).
- Grep: `title: 'Sucesso'`, `title: 'Erro'`, `com sucesso`.

### 2. Modais — usar o componente certo do DS, não rolar o próprio
- **Confirmar exclusão (sem vínculo):** título **"Atenção!"** vermelho + ícone `trash` (`bg-error-100`/`text-error-500`) + texto de ação irreversível + `[Cancelar] [Confirmar]`.
- **Bloqueio por vínculo:** **"Atenção!"** + **`dm-data-table`** paginada com os itens vinculados + só `[Voltar]`.
- **Sair sem salvar:** usar **`ExitConfirmationDialogComponent`** do DS (`@bancodoc/ui`) — primária = "Continuar editando".
- **Confirmação genérica (desativar/remover):** `dm-dialog-alert-template`, dialog `size="md"`, botões `size="sm"`.
- **Vinculação:** `dm-data-table` + busca + **alerta** `notify`/`bell` ("É necessário selecionar pelo menos um X para salvar a vinculação.") + botão **"Salvar" sempre ativo**.
- 🚩 Anti-padrões: `<table>` HTML cru dentro do dialog; modal próprio em vez do componente do DS; botão de vinculação `[disabled]` / rotulado "Vincular X"; alerta removido.

### 3. Botões, ações e nomenclatura
- Gerenciar um registro = ação única **"Gerenciar"** (ícone `gear`) — **não** "Visualizar"/"Editar" separados nem rota `/editar`. Somente-leitura é definido por **permissão**.
- Botão de confirmar em **modal de vinculação** = **"Salvar"**.
- 🚩 Anti-padrões: ações "Visualizar"+"Editar"; ícone `eye`; botão "Vincular X".

### 4. Validação de formulário
- **Required** = **`DmValidators.required`** (trata espaço em branco como vazio) — não `Validators.required` nativo.
- Ao submeter inválido: util **`markAllAsTouched`** do DS (emite `statusChanges` → renderiza erro sob OnPush + scroll-to-invalid) — não o `form.markAllAsTouched()` nativo.
- Mensagem padrão no campo: **"Campo obrigatório."**.
- **Tabela obrigatória:** erro **"Campo obrigatório." ABAIXO da tabela** (não acima, não em toast).
- Erro de servidor (duplicado 409) → **`setErrors({ duplicate: true })`** no campo (helper text via `customErrors`), nunca toast. E-mail inválido: **"Insira um e-mail válido"**.
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

## Output format

> **🔴/🟡/🟢 [Dimensão] — `arquivo:linha`**
> Faz: `<o que está fora>` · Padrão: `<o correto, com a referência no libs/ui ou empresas/servicos>` · Correção: `<a mudança>`

Sempre listar também o que **já está conforme**, para dar confiança.

## Common mistakes when auditing
- Inferir o "padrão" a partir do próprio módulo auditado → sempre ancorar em `libs/ui` + `empresas`/`servicos`.
- Marcar como desvio algo que é decisão de produto (ex.: grupo permite "Excluir mesmo assim") → separar **inconsistência de DS** de **regra de negócio**.
- Esquecer de varrer os **dialogs compartilhados** (`shared/ui/components`), não só as páginas.
