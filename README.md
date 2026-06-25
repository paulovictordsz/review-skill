# review-skill

[Claude Code](https://claude.com/claude-code) skills for reviewing a **DemarcoDS / `Bancodoc.Interface`** (Angular/Nx) feature module against the Design System — and verifying the fixes.

| Skill | O que faz |
|---|---|
| **`auditing-design-system-compliance`** | Varre o módulo procurando o que está **fora do padrão do DS** (toasts, modais, botões/ações, validação, tabelas, breadcrumb) e reporta **`fora do padrão → o padrão é → a correção`** com `arquivo:linha` e severidade. |
| **`verifying-ui-in-running-app`** | Como **verificar** uma mudança de UI na app rodando (dev server + mock API) sem cair nas armadilhas de bundle stale / change detection. Usada depois de aplicar as correções. |

## Como a auditoria funciona

A skill **não infere o padrão a partir do módulo auditado**. Ela ancora o padrão em duas fontes:
- O **Design System**: `libs/ui`.
- As telas de referência maduras: **`apps/empresas` (clientes)** e **`apps/servicos` (documentos)**.

O módulo auditado deve **espelhar** essas referências, não reinventar componentes próprios. Para cada desvio, reporta o que faz, o padrão correto (com a referência) e a correção.

## Os padrões que a skill verifica

### Toasts
- **Sucesso:** título `"[Entidade] [ação no particípio]!"`, **sem subtítulo**. O particípio segue o **comportamento** e **concorda em gênero** com a entidade:

  | Comportamento | Particípio (masc. / fem.) | Exemplos |
  |---|---|---|
  | Cadastro / inserção | `inserido` / `inserida` | "Tipo de documento inserido!", "Orientação geral inserida!" |
  | Edição | `editado` / `editada` | "Tipo de critério editado!", "Categoria de motivo editada!" |
  | Exclusão | `excluído` / `excluída` | "Tipo de uso excluído!", "Justificativa de motivo excluída!" |
  | Renomeação | `renomeado` / `renomeada` | "Tipo de orientação renomeado!", "Categoria de procedimento renomeada!" |
  | Vinculação | `vinculado` / `vinculada` (+ plural) | "Regra vinculada!", "Pesquisas vinculadas!" |
  | Exportação | (fixo) | "Arquivo exportado!" |

  Para auditar: ache todo `toastService.success(`, classifique o comportamento pela **ação do handler** (POST=inserção, PUT/PATCH=edição, rename=renomeação, vincular=vinculação, DELETE=exclusão), concorde o gênero pelo substantivo-núcleo da entidade e termine com `!`.
- **Erro:** `"Não foi possível [ação] o [item]"` + detalhe no subtítulo.
- **Info/bloqueio:** `"Sem permissão"` + motivo.
- 🚩 Fora do padrão: títulos genéricos `Sucesso`/`Erro`, `"...com sucesso"`, `cadastrado`/`salvo` genérico onde o comportamento pede `inserido`/`editado`/`renomeado`, gênero/plural trocado, falta de `!`, e **feedback de campo obrigatório em toast** (deve ser helper text no campo).

### Modais (usar o componente do DS, não rolar o próprio)
- **Excluir (sem vínculo):** "Atenção!" vermelho + ícone lixeira + texto irreversível + `[Cancelar][Confirmar]`.
- **Bloqueio por vínculo:** "Atenção!" + `dm-data-table` paginada dos vinculados + só `[Voltar]`.
- **Sair sem salvar:** `ExitConfirmationDialogComponent` do DS (primária = "Continuar editando").
- **Confirmação genérica:** `dm-dialog-alert-template`, dialog `md`, botões `sm`.
- **Vinculação:** `dm-data-table` + busca + alerta `notify`/`bell` + botão **"Salvar" sempre ativo**.

### Botões / ações
- Gerenciar registro = ação única **"Gerenciar"** (não "Visualizar"/"Editar"); somente-leitura via permissão.
- Confirmar em modal de vinculação = **"Salvar"**.

### Validação de formulário
- **Required** = `DmValidators.required` (trata espaço como vazio).
- Submit inválido → util **`markAllAsTouched`** do DS (renderiza erro sob OnPush + scroll).
- Mensagem **"Campo obrigatório."**; em **tabela obrigatória**, exibida **abaixo da tabela**.
- Erro de servidor (duplicado 409) → `setErrors` no campo (não toast).

### Tabelas / navegação
- Título da listagem **"[Item]s cadastrados"**; `pageSize` **10**; empty state "Ops, não existem registros cadastrados!".
- Breadcrumb começa pelo módulo (ex.: "Empresa"); textos acentuados.

> ⚠️ A skill separa **inconsistência de Design System** de **regra de negócio** (ex.: "um grupo pode ser excluído com vínculo" é decisão de produto, não desvio de DS).

## Install

```bash
git clone https://github.com/paulovictordsz/review-skill.git
cp -r review-skill/auditing-design-system-compliance ~/.claude/skills/
cp -r review-skill/verifying-ui-in-running-app    ~/.claude/skills/
```

As skills carregam automaticamente quando o contexto bate (ex.: "está no padrão?", "revisa o módulo", "padroniza").

## Como foram construídas

Autoradas com o processo TDD-for-skills (RED → GREEN): rodar um agente sem a skill (baseline), ver onde ele falha, escrever a skill mirando essas falhas, validar que um agente com a skill passa a acertar.
