# Guia OpenSpec + Azure DevOps — AfixRH

Este guia padroniza o uso do **OpenSpec** junto com o **Azure DevOps** no projeto AfixRH.

O objetivo é manter o backlog organizado, vincular mudanças a work items e registrar esforço/entregas de forma consistente.

---

## 1. Comandos disponíveis

O agente trabalha com seis comandos principais:

| Comando | Quando usar | O que faz |
|---|---|---|
| `/opsx-init` | No início do projeto ou quando a estrutura de módulos mudar | Lê `Docs/apresentacao.md` e `openspec/config.yaml`, sincroniza Epic → Feature → PBI no Azure DevOps e salva `openspec/backlog-mapping.yaml` |
| `/opsx-restructure` | Quando o domínio/módulos mudarem (ex: empresa única, novo papel) | Atualiza docs, `config.yaml`, backlog no Azure DevOps e specs principais, sempre com confirmação |
| `/opsx-propose` | Sempre que for começar uma mudança | Cria uma change vinculada a um PBI e opcionalmente cria uma Task filha no Azure DevOps |
| `/opsx-apply` | Quando quiser implementar uma change a partir do `tasks.md` | Lê os artefatos da change, executa as tasks pendentes e marca progresso |
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

```
/opsx-apply <nome-da-change>
```

Se o nome for omitido, o agente infere do contexto ou lista as changes ativas.

O agente:
1. Lê os artefatos da change (`proposal.md`, `design.md`, `tasks.md`, specs).
2. Executa as tasks pendentes.
3. Marca `- [ ]` → `- [x]` no `tasks.md`.
4. Atualiza delta specs se necessário.

Também é possível pedir informalmente: "Implemente a change `<nome>`".

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
│   │   ├── opsx-apply.md        ← implementa tasks da change
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
│   └── openspec-azure-devops-guide.md
└── .agents/
    └── AGENTS.md
```

---

## 5. O que é nativo e o que foi criado

A estrutura do OpenSpec no AfixRH mistura componentes nativos (da ferramenta OpenSpec/OpenCode) com extensões criadas especificamente para este projeto.

### Nativo do OpenSpec / OpenCode

| Componente | O que é |
|---|---|
| `openspec` CLI | A linha de comando oficial: `openspec list`, `openspec status`, `openspec new change`, etc. |
| `openspec/config.yaml` | Arquivo raiz de configuração do OpenSpec (nativo, mas preenchido pelo projeto). |
| `openspec/specs/` | Diretório de specs principais (capabilities). |
| `openspec/changes/` | Diretório de changes ativas e arquivadas. |
| Estrutura de change | `proposal.md`, `design.md`, `tasks.md`, `specs/` e `.openspec.yaml` são convenções nativas do OpenSpec. |
| `.opencode/` | Framework de comandos e skills do OpenCode. |
| `session-handoff` | Skill nativo do OpenCode para troca de contexto. |

### Criado/customizado para o AfixRH

| Componente | O que é |
|---|---|
| Comandos `/opsx-*` | Atalhos do agente (`/opsx-init`, `/opsx-propose`, `/opsx-apply`, `/opsx-archive`, etc.) que orquestram o OpenSpec + Azure DevOps. |
| Skills `openspec-*` | Skills em `.opencode/skills/openspec-*` (init-project, propose, apply, archive, sync-specs, etc.) que implementam a lógica específica do AfixRH. |
| Integração Azure DevOps | Criação de Epic/Feature/PBI/Task, campos customizados (`TempoEstimado`, `TempoRealizado`, `DataEstimada`, `DataEntrega`), comentários e transições de estado. |
| `Docs/apresentacao.md` e `Docs/estrutura-organizacional.md` | Fontes de domínio usadas para sincronizar o backlog. |
| `openspec/backlog-mapping.yaml` | Gerado automaticamente pelo `/opsx-init`; mapeia IDs do Azure DevOps para o projeto local. |
| `Docs/openspec-azure-devops-guide.md` | Este guia. |
| `.agents/AGENTS.md` | Arquivo de handoff mantido pelo `/opsx-handoff`. |

### Nativo do Azure DevOps

- Hierarquia de work items: **Epic → Feature → Product Backlog Item → Task**.
- Campos customizados existentes no processo do projeto: `Custom.TempoEstimado`, `Custom.TempoRealizado`, `Custom.DataEstimada`, `Custom.DataEntrega`.
- Campo padrão `Microsoft.VSTS.Scheduling.Effort`.

### Nativo do Git

- Controle de versão: `.git/`, branches, commits, push/pull.
- O OpenSpec não substitui o Git; ele gera artefatos que devem ser versionados junto com o código.

---

## 6. Regras de negócio do Azure DevOps

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

## 7. Boas práticas

- Mantenha `Docs/apresentacao.md` e `openspec/config.yaml` atualizados.
- Use `/opsx-restructure` quando o domínio mudar.
- Sempre vincule uma change a um PBI via `/opsx-propose`.
- Crie uma Task filha para cada change — facilita o registro de esforço.
- O `/opsx-archive` sempre sincroniza specs antes de mover a change.
- Nomeie changes em kebab-case (ex: `fix-login-empresa`).

---

## 8. Troubleshooting

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

---

## 9. Usando o OpenSpec diretamente (sem comandos do agente)

Os comandos `/opsx-*` são atalhos do agente que automatizam o OpenSpec **e** a integração com Azure DevOps. Se você preferir usar o OpenSpec "na mão", a CLI nativa oferece os mesmos blocos básicos — mas aí a integração com Azure DevOps (criar Task, preencher campos customizados, arquivar no ADO, etc.) precisa ser feita manualmente.

### 9.1. Pré-requisitos

1. Instale a CLI do OpenSpec e verifique se está no PATH:

```bash
openspec --version
```

2. O projeto já deve conter um `openspec/config.yaml` válido. Exemplo mínimo:

```yaml
project: AfixRH
schema: spec-driven
```

3. Opcional: configure um "store" (repositório OpenSpec standalone) se quiser compartilhar specs entre projetos:

```bash
openspec store list --json
openspec <comando> --store <id>
```

### 9.2. Passo a passo manual

#### 1. Inicializar o backlog (substitui `/opsx-init`)

O `openspec` nativo não sincroniza Epic/Feature/PBI com Azure DevOps. No modo manual, você deve:

- Criar/organizar os work items diretamente no Azure DevOps.
- Manter `Docs/apresentacao.md` e `openspec/config.yaml` atualizados como fonte de verdade.
- (Opcional) criar `openspec/backlog-mapping.yaml` à mão para guardar os IDs do ADO.

#### 2. Propor uma change (substitui `/opsx-propose`)

Crie a change via CLI:

```bash
openspec new change <nome-da-change>
```

Depois, edite manualmente os artefatos gerados em `openspec/changes/<nome-da-change>/`:

| Arquivo | O que fazer |
|---|---|
| `proposal.md` | Escreva o `## Why`, `## What Changes`, `## Capabilities`. |
| `design.md` | Desenhe a arquitetura técnica. |
| `tasks.md` | Liste as tarefas como checklist `- [ ]`. |
| `specs/<capability>/spec.md` | Crie/adapte os delta specs. |
| `.openspec.yaml` | Mantenha os metadados da change (gerado automaticamente). |

> No modo manual, a criação da Task filha no Azure DevOps, o preenchimento de `Tempo Estimado`, `Data Estimada` e `Effort` devem ser feitos diretamente no portal do Azure DevOps.

#### 3. Acompanhar o status da change

```bash
openspec list --json
openspec status --change "<nome-da-change>" --json
openspec show --change "<nome-da-change>"
```

Esses comandos mostram quais artefatos estão pendentes ou concluídos.

#### 4. Implementar a change (substitui `/opsx-apply`)

No modo manual, você mesmo executa as tarefas do `tasks.md` e marca-as como concluídas:

```markdown
- [x] 1.1 Criar entidade Vaga
```

Se precisar de instruções da change para o agente:

```bash
openspec instructions --change "<nome-da-change>"
```

#### 5. Validar specs e a change

```bash
openspec validate --change "<nome-da-change>"
openspec doctor
openspec context
```

- `validate` verifica se os artefatos estão consistentes.
- `doctor` diagnostica problemas no repositório OpenSpec.
- `context` exibe o contexto atual da change.

#### 6. Sincronizar delta specs com specs principais (substitui a sync do `/opsx-archive`)

O OpenSpec nativo não faz merge automático de specs. Você deve copiar/mergear manualmente os arquivos de:

```
openspec/changes/<nome-da-change>/specs/<capability>/spec.md
```

para:

```
openspec/specs/<capability>/spec.md
```

Regras básicas de merge:

- `## ADDED Requirements` → adicione na spec principal se ainda não existir.
- `## MODIFIED Requirements` → atualize os requisitos e cenários existentes, preservando o que não foi mencionado.
- `## REMOVED Requirements` → remova o bloco inteiro da spec principal.
- `## RENAMED Requirements` → renomeie `FROM` para `TO`.

Se o capability não existir ainda, crie `openspec/specs/<capability>/spec.md` com um `## Purpose` e os requisitos adicionados.

#### 7. Arquivar a change (substitui `/opsx-archive`)

```bash
openspec archive --change "<nome-da-change>"
```

Isso move a change para `openspec/changes/archive/YYYY-MM-DD-<nome-da-change>/`. O `.openspec.yaml` é preservado.

> Atenção: no modo manual, a atualização da Task no Azure DevOps (`Tempo Realizado`, `Data Entrega`, comentário e estado Done) **não** é feita pelo `openspec archive`. Faça isso manualmente no portal do ADO.

### 9.3. Quando usar o modo manual?

| Situação | Recomendação |
|---|---|
| Quer automação de Azure DevOps + validações do agente | Use os comandos `/opsx-*`. |
| Só quer planejar specs e rastrear changes localmente | Use `openspec` CLI diretamente. |
| Projeto não usa Azure DevOps | Use o modo manual. |
| Prefere controle total sobre cada passo | Use o modo manual. |

### 9.4. Resumo dos comandos nativos mais usados

```bash
openspec list --json                       # lista changes
openspec status --change "<n>" --json      # status dos artefatos
openspec show --change "<n>"               # detalhes da change
openspec new change "<n>"                  # cria change
openspec validate --change "<n>"           # valida artefatos
openspec doctor                            # diagnostica o repo
openspec context                           # contexto atual
openspec archive --change "<n>"            # arquiva change
openspec instructions --change "<n>"       # instruções da change
openspec store list --json                 # lista stores registrados
```

> Dica: comandos que leem/escrevem specs e changes (`new change`, `status`, `list`, `show`, `archive`, `validate`, `doctor`, `context`, `instructions`) aceitam `--store <id>` se você trabalhar com um store.

