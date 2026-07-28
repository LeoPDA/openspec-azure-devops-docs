# Guia OpenSpec + Azure DevOps

Este guia padroniza o uso do **OpenSpec** junto com o **Azure DevOps** no projeto AfixRH.

O objetivo é manter o backlog organizado, vincular mudanças a work items e registrar esforço/entregas de forma consistente.

Este guia pode ser usado tanto no projeto **AfixRH** quanto como referência para replicar o fluxo em outros projetos que usem OpenSpec + Azure DevOps.

---

## 1. Instalação e configuração inicial

### 1.1. Instalar a CLI `openspec`

A CLI do OpenSpec é distribuída via npm. Instale globalmente:

```bash
npm install -g @opencode-ai/openspec
```

Verifique a instalação:

```bash
openspec --version
```

Saída esperada:

```
1.6.0
```

> Se o comando não for encontrado, verifique se o npm global está no PATH da máquina.

### 1.2. Configurar o plugin no projeto

No projeto que usará o OpenSpec, crie uma pasta `.opencode/` na raiz e instale o plugin:

```bash
mkdir .opencode
cd .opencode
npm init -y
npm install @opencode-ai/plugin
```

Resultado mínimo:

```json
{
  "dependencies": {
    "@opencode-ai/plugin": "1.18.4"
  }
}
```

### 1.3. Criar a estrutura mínima

```
meu-projeto/
├── .opencode/
│   ├── package.json
│   ├── commands/
│   │   ├── opsx-init.md
│   │   ├── opsx-propose.md
│   │   ├── opsx-apply.md
│   │   ├── opsx-update.md
│   │   ├── opsx-sync.md
│   │   ├── opsx-archive.md
│   │   └── handoff.md
│   └── skills/
│       └── openspec-init-project/
│           └── SKILL.md
├── openspec/
│   ├── config.yaml
│   └── specs/
├── Docs/
│   └── apresentacao.md
└── .agents/
    └── AGENTS.md
```

Você pode copiar os comandos e skills deste repositório (`AfixRH`) como ponto de partida.

### 1.4. Inicializar o OpenSpec

Crie o arquivo `openspec/config.yaml` com o contexto do projeto:

```yaml
schema: spec-driven
context: |
  Nome do Projeto — descrição curta.

  Tech Stack:
    Backend: ...
    Frontend: ...

  Grupos de Módulos:
    Grupo 1:
      - Módulo A
      - Módulo B
    Grupo 2:
      - Módulo C

  Papéis:
    ...

rules:
  proposal:
    - Vincular a PBI/Task correspondente do Azure DevOps
  tasks:
    - Cada task deve ter Effort em horas
```

Crie o arquivo `Docs/apresentacao.md` com a visão geral, grupos e módulos do projeto.

Valide a configuração:

```bash
openspec doctor
```

Saída esperada:

```
Doctor

Root
  Location: C:\caminho\do\projeto
  OpenSpec root: ok

References
  (none declared)
```

---

## 2. Replicação em outros projetos

Para usar o mesmo fluxo em outro projeto, siga os passos abaixo.

### 2.1. Copiar comandos e skills

Copie do projeto AfixRH para o novo projeto:

- `.opencode/commands/opsx-init.md`
- `.opencode/commands/opsx-propose.md`
- `.opencode/commands/opsx-apply.md`
- `.opencode/commands/opsx-update.md`
- `.opencode/commands/opsx-sync.md`
- `.opencode/commands/opsx-archive.md`
- `.opencode/commands/handoff.md`
- `.opencode/skills/openspec-init-project/SKILL.md`

### 2.2. Adaptar comandos ao novo projeto

Em cada arquivo `.opencode/commands/`, ajuste:

- Nome do projeto Azure DevOps padrão.
- Email padrão do `AssignedTo`.
- Referências a caminhos de arquivos (ex: `Docs/apresentacao.md`).
- Campos customizados do Azure DevOps, se diferentes.
- Estados dos work items, se o template for diferente.

### 2.3. Adaptar o skill `/opsx-init`

Em `.opencode/skills/openspec-init-project/SKILL.md`, ajuste:

- Formato esperado de `Docs/apresentacao.md`.
- Formato esperado de `openspec/config.yaml`.
- Nome dos grupos e módulos.
- Campos customizados e estados.

### 2.4. Configurar o Azure DevOps no novo projeto

1. Crie o projeto no Azure DevOps.
2. Configure os campos customizados (veja seção 13 — Configuração do Azure DevOps).
3. Garanta que os estados dos work items estejam disponíveis.
4. Configure o MCP do Azure DevOps ou a CLI `az devops` com PAT/token.

### 2.5. Testar o fluxo

1. Rode `/opsx-init` e verifique se Epic/Feature/PBI são criados.
2. Rode `/opsx-propose` e verifique se a Task é criada e vinculada ao PBI.
3. Rode `/opsx-apply` para implementar.
4. Rode `/opsx-sync` e `/opsx-archive` para finalizar.

---

## 3. O que é o OpenSpec

OpenSpec é um workflow de **especificações guiadas por change**. Cada change é uma pasta de planejamento com artefatos que descrevem o que será feito, como será feito e as tarefas de implementação.

Componentes:

- **CLI `openspec`**: ferramenta oficial instalada na máquina (`openspec --version`).
- **Comandos `/opsx-*`**: atalhos definidos no projeto em `.opencode/commands/` para padronizar como o agente usa a CLI.
- **Skills**: instruções em `.opencode/skills/` que guiam o agente em cada comando.
- **Artefatos**: arquivos como `proposal.md`, `design.md`, `tasks.md` e specs criados dentro de cada change.

---

## 4. O que é nativo e o que foi criado no projeto

| Componente | Origem | Onde fica |
|---|---|---|
| CLI `openspec` | OpenSpec oficial | Instalada na máquina |
| Comandos `/opsx-*` | Criados/configurados no projeto | `.opencode/commands/` |
| Skills do OpenSpec | Criados/configurados no projeto | `.opencode/skills/` |
| Specs principais | Documentação do projeto | `openspec/specs/` |
| Changes | Planejamento de mudanças | `openspec/changes/` |
| Backlog mapping | Criado pelo `/opsx-init` | `openspec/backlog-mapping.yaml` |

---

## 5. Hierarquia de backlog no Azure DevOps

```
Epic (agrupamento de módulos)
└── Feature (módulo)
    └── Product Backlog Item (PBI)
        └── Task
```

- **Epic**: agrupamento de módulos por área/negócio (ex: "Administração", "Pessoas", "Talentos").
- **Feature**: módulo do sistema (ex: "Afix Profile", "Colaboradores", "Recrutamento").
- **PBI**: entregável de negócio dentro do módulo (ex: "Estrutura base do Afix Profile").
- **Task**: trabalho técnico de uma change (ex: "Criar rotas e páginas base do Afix Profile").

### Exemplo no AfixRH

```
Epic: Pessoas
└── Feature: Afix Profile
    └── PBI: Estrutura base do Afix Profile
        └── Task: Criar rotas e páginas base

Epic: Administração
└── Feature: Empresas & Estrutura
    └── PBI: CRUD de Cargos
        └── Task: Corrigir validação de EmpresaId
```

### Estados dos work items

| Tipo | Inicial | Em andamento | Finalizado |
|---|---|---|---|
| Epic | New | In Progress | Done |
| Feature | New | In Progress | Done |
| PBI | New | Approved → Committed | Ready |
| Task | To Do | In Progress | Done |

---

## 6. Documento de apresentação

O comando `/opsx-init` depende do arquivo `Docs/apresentacao.md`. Ele deve conter:

- Visão geral do sistema
- Lista de módulos e grupos
- Roadmap
- Estrutura organizacional

Exemplo de conteúdo esperado:

```markdown
## Módulos do Sistema

| # | Grupo | Módulo | Status |
|---|-------|--------|--------|
| 0 | Infraestrutura | Empresa, Departamento, Cargo, Usuário, Papel, Configuração | ✅ |
| 1 | Pessoas | Colaboradores | Planejado |
| 2 | Pessoas | Afix Profile | Planejado |
| 3 | Talentos | Recrutamento | Planejado |
```

> **Importante**: mantenha `Docs/apresentacao.md` atualizado. Ele é a fonte para criação do backlog no Azure DevOps.

---

## 7. Comandos disponíveis

### `/opsx-init`

Inicia o projeto no Azure DevOps criando Epic → Feature → PBI a partir de `Docs/apresentacao.md`.

> **Epic = agrupamento de módulos** (ex: Pessoas, Administração, Talentos).  
> **Feature = módulo** (ex: Afix Profile, Colaboradores).  
> **PBI = entregável do módulo**.

**Quando usar**:
- No início do projeto.
- Quando surgir um novo módulo ou novo agrupamento.
- Quando quiser preparar novos itens antes de começar as changes.

**O que faz**:
1. Pergunta qual projeto do Azure DevOps vincular.
2. Lê `Docs/apresentacao.md` e `openspec/config.yaml`.
3. Lista grupos (Epics) e módulos (Features) encontrados.
4. Pergunta quais Epic/Feature/PBI criar.
5. Cria os work items no Azure DevOps.
6. Salva mapping em `openspec/backlog-mapping.yaml`.

**Exemplo**:

```
/opsx-init
```

Saída esperada:

```
Grupos e módulos encontrados em Docs/apresentacao.md:

Epic: Pessoas
  [✓] Módulo 1 — Colaboradores
  [✓] Módulo 2 — Afix Profile

Epic: Talentos
  [✓] Módulo 3 — Recrutamento
  [✓] Módulo 4 — Desempenho
  [✓] Módulo 5 — Pesquisas

Criando no Azure DevOps:
✓ Epic: Pessoas (#5000)
✓ Feature: Afix Profile (#5001)
✓ PBI: Estrutura base do Afix Profile (#5002)

Mapping salvo em openspec/backlog-mapping.yaml
```

**Arquivo gerado**:

```yaml
projeto: AfixRH
assignedTo: leonardo.arouche@afixcode.com.br
epics:
  - nome: "Pessoas"
    id: 5000
    modulos:
      - nome: "Afix Profile"
        featureId: 5001
        pbiId: 5002
      - nome: "Colaboradores"
        featureId: 5003
        pbiId: 5004
  - nome: "Talentos"
    id: 5005
    modulos:
      - nome: "Recrutamento"
        featureId: 5006
        pbiId: 5007
```

---

### `/opsx-propose`

Cria uma nova change e vincula a um PBI existente, criando uma Task filha.

**Quando usar**: sempre que for começar uma mudança.

**O que faz**:
1. Pergunta nome da change.
2. Pergunta a qual PBI vincular (lista do mapping ou busca no Azure DevOps).
3. Pergunta título da Task.
4. Pergunta **Tempo Estimado** (horas).
5. Pergunta **Data Estimada** de entrega.
6. Pergunta **Effort** (1, 2 ou 3).
   - 1 = 2 a 4 horas
   - 2 = 1 dia
   - 3 = 2 dias
   - Acima de 3 → avisa para quebrar em outra Task.
7. Cria Task no Azure DevOps com:
   - `AssignedTo`: email padrão do mapping.
   - `State`: To Do.
   - Campos customizados: Tempo Estimado, Data Estimada, Effort.
8. Cria change local e registra vínculo.

**Exemplo**:

```
/opsx-propose estrutura-base-afix-profile
```

Perguntas do agente:

```
PBIs disponíveis:
1. Estrutura base do Afix Profile (#5002)
2. CRUD de Cargos (#5100)

Selecione o PBI: 1

Título da Task: Criar rotas e páginas base do Afix Profile
Tempo Estimado (horas): 8
Data Estimada (YYYY-MM-DD): 2026-08-01
Effort (1=2-4h, 2=1dia, 3=2dias): 2

Criar Task filha? [S/n]: S

✓ Task criada: #5200 — Criar rotas e páginas base do Afix Profile
✓ Change criada: estrutura-base-afix-profile
```

---

### `/opsx-apply`

Implementa as tarefas da change.

**Quando usar**: durante o desenvolvimento.

**O que faz**:
1. Lê os artefatos da change.
2. Implementa cada tarefa pendente.
3. Marca `- [ ]` → `- [x]` no `tasks.md`.
4. Pergunta se atualiza a Task vinculada para **In Progress**.

---

### `/opsx-update`

Revisa os artefatos de planejamento de uma change existente.

**Quando usar**: quando o escopo muda ou alguém editou um artefato manualmente.

**O que faz**:
1. Lê e reconcilia artefatos.
2. Confirma revisões com o usuário antes de escrever.
3. Se houver Task vinculada e alteração relevante, pergunta se adiciona comentário em Discussion.

---

### `/opsx-sync`

Sincroniza delta specs de uma change com os specs principais.

**Quando usar**: antes de arquivar, quando specs da change precisam ir para `openspec/specs/`.

**O que faz**:
1. Lê delta specs da change.
2. Aplica alterações nos main specs.
3. Se houver Task vinculada, pergunta se adiciona comentário em Discussion.

---

### `/opsx-archive`

Arquiva a change concluída.

**Quando usar**: quando a change está totalmente implementada e specs sincronizados.

**O que faz**:
1. Move a pasta da change para `openspec/changes/archive/`.
2. Se houver Task vinculada:
   - Pergunta **Tempo Realizado** (horas).
   - Pergunta **Data Entrega** efetiva.
   - Adiciona comentário em Discussion com resumo do que foi feito.
   - Pergunta se atualiza estado da Task para **Done**.

---

### `/opsx-handoff`

Salva o estado atual do trabalho em `.agents/AGENTS.md`.

**Quando usar**: ao trocar de tarefa, encerrar sessão ou quando o contexto está cheio.

---

## 8. Regras de negócio do Azure DevOps

### AssignedTo

Todo novo work item criado pelo OpenSpec usa o email configurado em `openspec/backlog-mapping.yaml`:

```yaml
assignedTo: leonardo.arouche@afixcode.com.br
```

Cada desenvolvedor deve alterar esse email para o seu ao usar o `/opsx-init`.

### Campos obrigatórios para Tasks

| Campo | Referência | Quando preencher |
|---|---|---|
| Tempo Estimado | `Custom.TempoEstimado` | Na criação da Task |
| Tempo Realizado | `Custom.TempoRealizado`¹ | No fechamento da Task |
| Data Estimada | `Custom.DataEstimada` | Na criação da Task |
| Data Entrega | `Custom.DataEntrega`¹ | No fechamento da Task |
| Effort | `Microsoft.VSTS.Scheduling.Effort` | Na criação da Task |

¹ Verifique se esses campos existem no seu projeto Azure DevOps. Caso contrário, solicite a criação ao administrador.

### Effort

| Effort | Duração |
|---|---|
| 1 | 2 a 4 horas |
| 2 | 1 dia |
| 3 | 2 dias |

Se uma mudança ultrapassar 2 dias, **quebre em outra Task**.

### Horas mínimas

Sempre lance no mínimo **30 minutos** (0,5 hora) ao fechar uma Task.

### Juntar tarefas

Você pode juntar 2 ou mais tarefas leves na **mesma Task**. Nunca junte demandas de Backlog ou Bug na mesma Task.

> Neste fluxo do OpenSpec, criamos apenas Tasks. Backlogs e Bugs devem ser tratados manualmente no Azure DevOps.

### Comentários em Discussion

Sempre que:
- Finalizar uma Task
- Alterar uma Task já iniciada
- Sincronizar specs relevantes

Adicione um comentário em Discussion explicando o que foi feito.

### Consideração sobre esforço da IA

O tempo de desenvolvimento com IA é maior que o de um humano isoladamente, mas o humano ainda precisa testar, validar e revisar. Ao preencher **Tempo Realizado**, considere um valor que represente o esforço total humano + IA de forma justa.

---

## 9. Fluxo completo

```
1. /opsx-init
   → Cria Epic → Feature → PBI no Azure DevOps
   → Salva openspec/backlog-mapping.yaml

2. /opsx-propose <nome-da-change>
   → Escolhe PBI
   → Cria Task filha
   → Cria change local

3. /opsx-apply <nome-da-change>
   → Implementa código
   → Atualiza Task para In Progress (se pedir)

4. /opsx-update <nome-da-change> (se necessário)
   → Revisa plano
   → Adiciona comentário em Discussion (se necessário)

5. /opsx-sync <nome-da-change>
   → Sincroniza specs
   → Adiciona comentário em Discussion (se necessário)

6. /opsx-archive <nome-da-change>
   → Arquiva change
   → Atualiza Task para Done (se pedir)
   → Registra Tempo Realizado e Data Entrega
   → Adiciona comentário em Discussion
```

---

## 10. Estrutura de pastas

```
AfixRH/
├── .opencode/
│   ├── commands/
│   │   ├── opsx-init.md
│   │   ├── opsx-propose.md
│   │   ├── opsx-apply.md
│   │   ├── opsx-update.md
│   │   ├── opsx-sync.md
│   │   ├── opsx-archive.md
│   │   └── handoff.md
│   └── skills/
│       ├── openspec-init-project/
│       │   └── SKILL.md
│       ├── openspec-propose/
│       ├── openspec-apply-change/
│       └── ...
├── openspec/
│   ├── backlog-mapping.yaml       ← criado pelo /opsx-init
│   ├── changes/
│   │   ├── archive/               ← changes arquivadas
│   │   └── <change-ativa>/
│   │       ├── .openspec.yaml
│   │       ├── proposal.md
│   │       ├── design.md
│   │       ├── tasks.md
│   │       └── specs/             ← delta specs (opcional)
│   └── specs/
│       └── modulo0-*/
│           └── spec.md            ← specs principais
├── Docs/
│   ├── apresentacao.md            ← fonte do /opsx-init
│   └── openpec-azure-devops-guide.md
└── .agents/
    └── AGENTS.md                  ← criado/atualizado pelo /opsx-handoff
```

---

## 11. Separação de specs: `openspec/specs/` vs `openspec/changes/.../specs/`

### `openspec/specs/` — Specs principais

São as especificações oficiais do projeto. Representam o estado atual do sistema.

- `modulo0-autenticacao/spec.md`
- `modulo0-cadastros/spec.md`
- `modulo2-afix-profile/spec.md`

Atualizados via `/opsx-sync` ou manualmente.

### `openspec/changes/<change>/specs/` — Delta specs

São as alterações propostas por uma change. Representam o que **mudará** em relação aos specs principais.

Quando uma change é arquivada sem sync, os delta specs vão junto para `openspec/changes/archive/`. Por isso a change `2026-07-24-hardening-modulo0-admin-crud` ainda tem specs dentro dela: eles não foram sincronizados com `openspec/specs/` antes do arquivamento.

### Recomendação

Sempre rode `/opsx-sync` antes de `/opsx-archive`. Assim os specs principais ficam atualizados e o archive fica apenas com o histórico da change.

---

## 12. Como criar/estender comandos

### Criar um novo comando

1. Crie um arquivo em `.opencode/commands/<nome-do-comando>.md`.
2. No cabeçalho, defina `description`.
3. No corpo, descreva propósito, entrada, fluxo e saída.
4. Se necessário, crie um skill em `.opencode/skills/<nome-da-skill>/SKILL.md`.
5. Referencie o skill no comando com `Load the '<skill-name>' skill`.

### Criar um novo skill

1. Crie a pasta `.opencode/skills/<nome-da-skill>/`.
2. Crie `SKILL.md` com instruções detalhadas.
3. Use o skill em um comando ou invoque diretamente via `Skill tool`.

### Comandos úteis da CLI openspec

```bash
# Ver versão
openspec --version

# Listar changes
openspec list --json

# Status de uma change
openspec status --change "<nome>" --json

# Instruções de um artefato
openspec instructions <artifact-id> --change "<nome>" --json

# Criar nova change
openspec new change "<nome>"

# Ver saúde do projeto
openspec doctor
```

---

## 13. Configuração do Azure DevOps

Antes de usar os comandos `/opsx-init` e `/opsx-propose`, configure o Azure DevOps corretamente.

### 13.1. Projeto e permissões

1. Crie o projeto no Azure DevOps.
2. Garanta que o usuário tenha permissões para:
   - Criar/editar Epic, Feature, PBI e Task.
   - Adicionar comentários em work items.
   - Alterar estados dos work items.

### 13.2. Estados dos work items

O fluxo espera os seguintes estados:

| Tipo | Estados |
|---|---|
| Epic | New, In Progress, Done |
| Feature | New, In Progress, Done |
| PBI | New, Approved, Committed, Ready |
| Task | To Do, In Progress, Done |

Se o seu projeto usar um processo personalizado com estados diferentes, ajuste os comandos em `.opencode/commands/`.

### 13.3. Campos customizados

Crie os campos customizados abaixo no template de work items:

| Campo | Tipo | Referência | Obrigatório para |
|---|---|---|---|
| Tempo Estimado | Número | `Custom.TempoEstimado` | Task |
| Tempo Realizado | Número | `Custom.TempoRealizado` | Task |
| Data Estimada | Data/Hora | `Custom.DataEstimada` | Task |
| Data Entrega | Data/Hora | `Custom.DataEntrega` | Task |
| Effort | Número | `Microsoft.VSTS.Scheduling.Effort` | Task |

> O campo `Microsoft.VSTS.Scheduling.Effort` já existe em alguns processos do Azure DevOps (Scrum). Os demais precisam ser criados manualmente.

### 13.4. Autenticação

O agente pode interagir com o Azure DevOps de duas formas:

**Opção A — MCP do Azure DevOps**

Configure o MCP no seu ambiente de agente com as credenciais do Azure DevOps (PAT, organização e projeto padrão).

**Opção B — Azure CLI (`az devops`)**

Instale e autentique:

```bash
az extension add --name azure-devops
az login
az devops configure --defaults organization=https://dev.azure.com/sua-org project=seu-projeto
```

> Se usar a Opção B, os comandos `/opsx-*` precisarão ser adaptados para chamar `az devops` em vez do MCP.

### 13.5. Configuração no comando `/opsx-init`

No skill `.opencode/skills/openspec-init-project/SKILL.md` e no comando `.opencode/commands/opsx-init.md`, ajuste:

- Projeto padrão do Azure DevOps.
- Email padrão do `AssignedTo`.
- Referências aos campos customizados.
- Estados dos work items.

---

## 14. Pré-requisitos

- CLI `openspec` instalada (`openspec --version`).
- Acesso ao projeto **AfixRH** no Azure DevOps.
- Permissões para criar/editar work items e adicionar comentários.
- MCP Azure DevOps conectado ao agente.
- Campos customizados configurados no Azure DevOps:
  - `Custom.TempoEstimado`
  - `Custom.TempoRealizado`
  - `Custom.DataEstimada`
  - `Custom.DataEntrega`
  - `Microsoft.VSTS.Scheduling.Effort`

> Se algum campo não existir, solicite ao administrador do Azure DevOps antes de usar o fluxo completo.

---

## 15. Exemplo completo

### Cenário

Implementar a estrutura base do Afix Profile.

### Passo 1 — Iniciar projeto (se ainda não feito)

```
/opsx-init
```

Resultado:
- Epic: #5000 — Pessoas
- Feature: #5001 — Afix Profile
- PBI: #5002 — Estrutura base do Afix Profile

### Passo 2 — Criar change

```
/opsx-propose estrutura-base-afix-profile
```

Resultado:
- Change criada localmente
- Task: #5200 — Criar rotas e páginas base do Afix Profile

### Passo 3 — Implementar

```
/opsx-apply estrutura-base-afix-profile
```

O agente implementa o código. Task #5200 vai para In Progress.

### Passo 4 — Sincronizar specs

```
/opsx-sync estrutura-base-afix-profile
```

Specs de `modulo2-afix-profile` são atualizados.

### Passo 5 — Arquivar

```
/opsx-archive estrutura-base-afix-profile
```

- Change vai para `openspec/changes/archive/`.
- Task #5200 vai para Done.
- Registrado Tempo Realizado e Data Entrega.
- Adicionado comentário em Discussion.

---

## 16. Boas práticas

- Mantenha `Docs/apresentacao.md` atualizado.
- Use `/opsx-sync` antes de `/opsx-archive`.
- Sempre vincule uma change a um PBI.
- Nomeie changes em kebab-case (ex: `fix-login-empresa`).
- Preencha Tempo Estimado e Data Estimada na criação da Task.
- Preencha Tempo Realizado e Data Entrega no fechamento.
- Adicione comentários em Discussion ao finalizar ou alterar.

---

## 17. Troubleshooting

### `openspec` não encontrado

Instale a CLI ou verifique se está no PATH:

```bash
openspec --version
```

### Campos customizados não existem

Verifique no Azure DevOps se os campos `Custom.TempoEstimado`, `Custom.TempoRealizado`, `Custom.DataEstimada`, `Custom.DataEntrega` e `Microsoft.VSTS.Scheduling.Effort` existem. Se não existirem, o fluxo de criação de Task pode falhar.

### Specs duplicados em archive

Isso acontece quando `/opsx-sync` não foi rodado antes de `/opsx-archive`. Para corrigir:

1. Restaure a change do archive (se necessário).
2. Rode `/opsx-sync <change>`.
3. Arquive novamente.

### Task não atualiza estado

Verifique se o estado desejado existe no template do projeto. Os estados esperados são:
- Task: To Do, In Progress, Done
- PBI: New, Approved, Committed, Ready
