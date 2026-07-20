# review-skill

[Claude Code](https://claude.com/claude-code) skill para fazer **review de um módulo de feature do DemarcoDS / `Bancodoc.Interface`** (Angular/Nx) — tanto nos **componentes/código** quanto **em funcionamento (app rodando)**.

É **uma skill só**, com duas frentes:

| Parte | O que faz |
|---|---|
| **A — Auditoria de DS (código)** | Varre o módulo procurando o que está **fora do padrão do DS** (toasts, modais, botões/ações, validação, tabelas, layout, contagem→modal, tooltip-vs-alert) e reporta **`fora do padrão → o padrão é → a correção`** com `arquivo:linha` e severidade. |
| **B — Verificação em funcionamento** | Como **verificar** uma mudança de UI na app rodando (dev server + mock API) sem cair nas armadilhas de bundle stale / change detection / sinais enganosos da mock API. Usada depois de aplicar as correções. |

## Como a auditoria funciona

A skill **não infere o padrão a partir do módulo auditado**. Ela ancora o padrão em duas fontes:
- O **Design System**: `libs/ui`.
- As telas de referência maduras: **`apps/empresas` (clientes)** e **`apps/servicos` (documentos)**.

O módulo auditado deve **espelhar** essas referências, não reinventar componentes próprios. Para cada desvio, reporta o que faz, o padrão correto (com a referência) e a correção.

## Os padrões que a skill verifica

### Toasts
- **Sucesso:** título `"[Entidade] [ação no particípio]!"`, **sem subtítulo**. O particípio segue o **comportamento** e **concorda em gênero** com a entidade.

  **Distinção crítica — "salvo" (formulário) vs "inserido/editado" (grid inline):**
  - Salvar entidade via **formulário** (botão "Salvar", handler que faz create OU edit) → **`[Entidade] salvo!`** (ex.: `Grupo salvo!`, `Usuário salvo!`). Ref.: `empresas`.
  - **CRUD inline em grid** (adicionar/editar/renomear item numa tabela) → `inserido!/editado!/renomeado!`. Ref.: `servicos`.

  | Comportamento | Particípio (masc. / fem.) | Exemplos |
  |---|---|---|
  | Salvar entidade (formulário) | `salvo` / `salva` | "Grupo salvo!", "Perfil salvo!", "Usuário salvo!" |
  | Inserção inline (grid) | `inserido` / `inserida` | "Tipo de documento inserido!", "Orientação geral inserida!" |
  | Edição | `editado` / `editada` | "Tipo de critério editado!", "Categoria de motivo editada!" |
  | Exclusão | `excluído` / `excluída` | "Tipo de uso excluído!", "Justificativa de motivo excluída!" |
  | Renomeação | `renomeado` / `renomeada` | "Tipo de orientação renomeado!", "Categoria de procedimento renomeada!" |
  | Vinculação | `vinculado` / `vinculada` (+ plural) | "Regra vinculada!", "Pesquisas vinculadas!" |
  | Exportação | (fixo) | "Arquivo exportado!" |

  Para auditar: ache todo `toastService.success(`, classifique o comportamento pela **ação do handler** (POST=inserção, PUT/PATCH=edição, rename=renomeação, vincular=vinculação, DELETE=exclusão), concorde o gênero pelo substantivo-núcleo da entidade e termine com `!`.
- **Erro:** `"Não foi possível [ação] o [item]"` + detalhe no subtítulo.
- **Info/bloqueio:** `"Sem permissão"` + motivo.
- 🚩 Fora do padrão: títulos genéricos `Sucesso`/`Erro`, `"...com sucesso"`, `cadastrado` genérico, `inserido`/`editado` num toast de **formulário** (ali é `salvo!`), gênero/plural trocado, falta de `!`, e **feedback de campo obrigatório em toast** (deve ser helper text no campo).

### Modais (usar o componente do DS, não rolar o próprio)
- **Excluir (sem vínculo):** "Atenção!" vermelho + ícone lixeira + texto irreversível + `[Cancelar][Confirmar]`.
- **Remover vínculo / excluir (variante `danger`):** mesmo visual "Atenção!" + trash; texto no padrão **`Ao clicar em "Confirmar", você concorda que [item] vinculado será removido [contexto].`** (concorda gênero/número). Use uma variante `danger` no dialog genérico, não outro componente.
- **Mudança de status (desativar/ativar):** confirmação **neutra** (ícone `warning`), **não** danger.
- **Bloqueio por vínculo:** "Atenção!" + `dm-data-table` paginada dos vinculados + só `[Voltar]`.
- **Sair sem salvar:** `ExitConfirmationDialogComponent` do DS (primária = "Continuar editando").
- **Confirmação genérica:** `dm-dialog-alert-template`, dialog `md`, botões `sm`.
- **Tamanho:** confirmação = **`md`** (714px). Conteúdo: **⚠️ em aberto** — o código tem 32 modais em `xl` (inclusive o módulo de referência) contra 17 em `lg`, e o Figma indica `lg` (824×580). Não migre nem reporte como desvio até o design decidir.
- **Largura só funciona com o host:** o componente aberto via `DmDialogService.open()` precisa de `host: { class: 'w-full inline-flex justify-center' }`, senão o `size` não tem efeito.
- **Vinculação:** `dm-data-table` + busca + alerta `notify`/`bell` + botão **"Salvar" sempre ativo**.

### Botões / ações
- Gerenciar registro = ação única **"Gerenciar"** (não "Visualizar"/"Editar"); somente-leitura via permissão.
- Confirmar em modal de vinculação = **"Salvar"**. Botão de salvar de **formulário** = sempre **"Salvar"** (não "Cadastrar").
- **Footer do formulário:** `border-t border-neutral-200 bg-neutral-50 rounded-b-lg`, à direita, **só o primário "Salvar"** (voltar é pelo header).
- **Link clicável** = diretiva **`dmLinkButton`** (cor `secondary-700`) — não `text-primary-500` nem `#0075FF` hardcoded.

### Validação de formulário
- **Required** = `DmValidators.required` (trata espaço como vazio).
- Submit inválido → util **`markAllAsTouched`** do DS (renderiza erro sob OnPush + scroll).
- Mensagem **"Campo obrigatório."**; em **tabela obrigatória**, exibida **abaixo da tabela**.
- Erro de servidor (duplicado 409) → `setErrors` no campo (não toast). Nome duplicado: **"Já existe um item com este nome cadastrado."** (genérico "item").

### Layout do formulário
- **Cabeçalho** (`dm-page-header`: voltar + título) aparece em **criação E edição** (`Novo X` / `Gerenciar X`) — nunca dentro de `@if (isCreateMode)`.
- **Espaçamento** de campos/seções = **`gap-8` / `space-y-8`** (32px), padrão do cadastro de programas (não `gap-4`/`space-y-4`).

### Contagem clicável → modal
- Número de itens vinculados numa tabela (fora de modais) = **link `dmLinkButton`** + ícone `arrow-line-up-right` que abre um **modal** (`dm-data-table` paginada) listando os itens. Só clicável quando `> 0`.

### Info contextual
- Apoio **não-obrigatório** ao lado de um título → **tooltip** (`dm-icon name="info"` + `[dmTooltip]`), não `dm-alert`. `dm-alert` fica para **ação requerida**.

### Tabelas / navegação
- Título da listagem **"[Item]s cadastrados"**; `pageSize` **10**; empty state "Ops, não existem registros cadastrados!".
- **Negrito só na 1ª coluna**; demais células sem formatação própria. **Linha não é clicável** — a ação fica no dropdown `dots-three`.
- ⚠️ Cor via utility class no `<td>` **não funciona** (a regra `.dm-table tbody tr td` do DS vence por especificidade). Cheque com `getComputedStyle` antes de confiar no realce.
- Breadcrumb começa pelo módulo (ex.: "Empresa"); textos acentuados. O breadcrumb mora no **LayoutComponent do app**, não na página.

### Layout da tela de consulta/listagem
- Referência: `servicos/documentos/pages/consulta`. `app.component` `px-8 py-8 md:px-16` → layout com breadcrumb fora + `<main>` branco → página **`p-8`** → card da listagem **`mt-6 rounded-lg border border-gray-200 bg-white`** (filtros + tabela, cada um `flex flex-col gap-8 p-8`).
- Título e seletores de contexto ficam **dentro** do conteúdo, no `dm-page-header` (ações no slot `actions`).

> ⚠️ A skill separa **inconsistência de Design System** de **regra de negócio** (ex.: "um grupo pode ser excluído com vínculo" é decisão de produto, não desvio de DS).

## Install

```bash
git clone https://github.com/paulovictordsz/review-skill.git ~/.claude/skills/review-skill
```

O `SKILL.md` fica na raiz do repo, então clonar direto em `~/.claude/skills/review-skill/` já registra a skill — e mantém o repo git para `pull`/`push` depois. Ela carrega automaticamente quando o contexto bate (ex.: "está no padrão?", "revisa o módulo", "padroniza", "review em funcionamento").

## Como foram construídas

Autoradas com o processo TDD-for-skills (RED → GREEN): rodar um agente sem a skill (baseline), ver onde ele falha, escrever a skill mirando essas falhas, validar que um agente com a skill passa a acertar.
