# Guia OpenSpec + Azure DevOps — AfixRH

Este guia padroniza o uso do **OpenSpec** junto com o **Azure DevOps** no projeto AfixRH.

O objetivo é manter o backlog organizado, vincular mudanças a work items e registrar esforço/entregas de forma consistente.

---

## 1. Comandos disponíveis

O agente trabalha com cinco comandos principais:

| Comando | Quando usar | O que faz |
|---|---|---|
| `/opsx-init` | No início do projeto ou quando a estrutura de módulos mudar | Lê `Docs/apresentacao.md` e `openspec/config.yaml`, sincroniza Epic → Feature → PBI no Azure DevOps e salva `openspec/backlog-mapping.yaml` |
| `/opsx-restructure` | Quando o domínio/módulos mudarem (ex: empresa única, novo papel) | Atualiza docs, `config.yaml`, backlog no Azure DevOps e specs principais, sempre com confirmação |
| `/opsx-propose` | Sempre que for começar uma mudança | Cria uma change vinculada a um PBI e opcionalmente cria uma Task filha no Azure DevOps |
| `/opsx-archive` | Quando a change estiver implementada e os specs sincronizados | Move a change para `openspec/changes/archive/`, atualiza Tempo Realizado/Data Entrega e move a Task para **Done** |
| `/opsx-handoff` | Ao trocar de tarefa, encerrar sessão ou quando o contexto estiver cheio | Salva o estado atual do trabalho em `.agents/AGENTS.md` para continuidade em outra sessão |

> **Importante**: specs (`openspec/specs/`) são criados ou atualizados apenas quando um PBI é atacado via `/opsx-propose` + implementação. O `/opsx-init` e `/opsx-restructure` não geram specs antecipadamente.

---

## 2. Hierarquia de backlog

```
Epic (grupo de módulos, ex: Administração, Pessoas, Talentos)
└── Feature (módulo, ex: Infraestrutura, Colaboradores, Afix Profile)
    └── Product Backlog Item (entregável do módulo)
        └── Task (trabalho técnico de uma change)
```

### Estados esperados

| Tipo | Inicial | Em andamento | Finalizado |
|---|---|---|---|
| Epic | New | In Progress | Done |
| Feature | New | In Progress | Done |
| PBI | New | Approved → Committed | Ready |
| Task | To Do | In Progress | Done |

---

## 3. Fluxo completo

### 3.1. Inicializar ou sincronizar o projeto

```
/opsx-init
```

1. Lê `Docs/apresentacao.md`, `Docs/estrutura-organizacional.md` e `openspec/config.yaml`.
2. Compara com Epic/Feature/PBI existentes no Azure DevOps.
3. Mostra diff e pede confirmação.
4. Cria/atualiza work items e salva `openspec/backlog-mapping.yaml`.
5. **Não cria Tasks nem specs.**

### 3.2. Reestruturar quando o domínio mudar

```
/opsx-restructure
```

Use quando algo muda na estrutura geral do projeto (ex: removeu matriz/filial, adicionou Super-sistema, novo módulo).

1. Detecta ou recebe a mudança estrutural.
2. Propõe atualizações em `Docs/apresentacao.md`, `Docs/estrutura-organizacional.md` e `openspec/config.yaml`.
3. Após confirmação, atualiza os arquivos locais.
4. Propõe atualizações no Azure DevOps.
5. Após confirmação, atualiza Epic/Feature/PBI.
6. Revisa specs principais afetados e propõe ajustes.

### 3.3. Propor uma change

```
/opsx-propose <nome-da-change>
```

1. Pergunta o que construir (se não fornecido).
2. Lista PBIs disponíveis para vincular.
3. Pergunta se cria uma Task filha no PBI.
4. Se sim, pergunta título, Tempo Estimado, Data Estimada e Effort.
5. Cria a Task no Azure DevOps.
6. Cria a change local com `proposal.md`, `design.md`, `tasks.md`.
7. Registra PBI e Task em `proposal.md`.

### 3.4. Implementar a change

A implementação é feita pelo agente a partir do `tasks.md`. Não há comando separado — basta pedir:

> "Implemente a change `<nome>`"

O agente:
1. Lê os artefatos da change.
2. Executa as tasks pendentes.
3. Marca `- [ ]` → `- [x]` no `tasks.md`.
4. Atualiza delta specs se necessário.

### 3.5. Arquivar a change

```
/opsx-archive <nome-da-change>
```

1. Verifica se artifacts e tasks estão completos.
2. **Obriga sincronizar delta specs** com `openspec/specs/`.
3. Lê Task vinculada em `proposal.md`.
4. Pergunta Tempo Realizado, Data Entrega e comentário.
5. Atualiza campos customizados no Azure DevOps.
6. Adiciona comentário em Discussion.
7. Pergunta se move a Task para **Done**.
8. Move a change para `openspec/changes/archive/YYYY-MM-DD-<nome>/`.

### 3.6. Handoff (troca de contexto)

```
/opsx-handoff
```

Use quando:
- O contexto da sessão estiver muito grande.
- Você for trocar de tarefa.
- For encerrar a sessão e quer continuar depois.

O agente salva o estado atual em `.agents/AGENTS.md`, incluindo o que foi feito, o que está em andamento e o próximo passo.

---

## 4. Estrutura de pastas

```
AfixRH/
├── .opencode/
│   ├── commands/
│   │   ├── opsx-init.md
│   │   ├── opsx-restructure.md
│   │   ├── opsx-propose.md
│   │   ├── opsx-archive.md
│   │   ├── opsx-handoff.md      ← salva estado da sessão
│   │   └── deprecated/          ← comandos antigos (não usar)
│   └── skills/
│       └── openspec-*/
│           └── SKILL.md
├── openspec/
│   ├── config.yaml
│   ├── backlog-mapping.yaml     ← criado/atualizado pelo /opsx-init
│   ├── changes/
│   │   ├── archive/             ← changes arquivadas
│   │   └── <change-ativa>/
│   │       ├── .openspec.yaml
│   │       ├── proposal.md
│   │       ├── design.md
│   │       ├── tasks.md
│   │       └── specs/           ← delta specs (opcional)
│   └── specs/
│       └── <capability>/
│           └── spec.md          ← specs principais
├── Docs/
│   ├── apresentacao.md          ← fonte do /opsx-init
│   ├── estrutura-organizacional.md
│   └── openpec-azure-devops-guide.md
└── .agents/
    └── AGENTS.md
```

---

## 5. Regras de negócio do Azure DevOps

### AssignedTo

Todo novo work item criado pelo OpenSpec usa o email configurado em `openspec/backlog-mapping.yaml`:

```yaml
assignedTo: leonardo.arouche@afixcode.com.br
```

### Campos obrigatórios para Tasks

| Campo | Referência | Quando preencher |
|---|---|---|
| Tempo Estimado | `Custom.TempoEstimado` | Na criação da Task |
| Tempo Realizado | `Custom.TempoRealizado` | No fechamento da Task |
| Data Estimada | `Custom.DataEstimada` | Na criação da Task |
| Data Entrega | `Custom.DataEntrega` | No fechamento da Task |
| Effort | `Microsoft.VSTS.Scheduling.Effort` | Na criação da Task |

### Effort

| Effort | Duração |
|---|---|
| 1 | 2 a 4 horas |
| 2 | 1 dia |
| 3 | 2 dias |

Se uma mudança ultrapassar 2 dias, **quebre em outra Task**.

### Horas mínimas

Sempre lance no mínimo **30 minutos** (0,5 hora) ao fechar uma Task.

---

## 6. Boas práticas

- Mantenha `Docs/apresentacao.md` e `openspec/config.yaml` atualizados.
- Use `/opsx-restructure` quando o domínio mudar.
- Sempre vincule uma change a um PBI via `/opsx-propose`.
- Crie uma Task filha para cada change — facilita o registro de esforço.
- O `/opsx-archive` sempre sincroniza specs antes de mover a change.
- Nomeie changes em kebab-case (ex: `fix-login-empresa`).

---

## 7. Troubleshooting

### `openspec` não encontrado

Instale a CLI ou verifique se está no PATH:

```bash
openspec --version
```

### Specs duplicados em archive

Isso acontece quando os delta specs não são sincronizados antes de arquivar. O `/opsx-archive` atual exige a sincronia, então isso não deve mais ocorrer.

### Task não atualiza estado

Verifique se o estado desejado existe no template do projeto. Os estados esperados são:
- Task: To Do, In Progress, Done
- PBI: New, Approved, Committed, Ready
