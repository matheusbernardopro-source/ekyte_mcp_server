---
name: ekyte
description: Especialista no MCP do Ekyte para gerenciar tarefas, projetos, usuarios, workspaces, fases e apontamentos de horas. Use quando o usuario quiser criar tarefa/projeto, atualizar fase, completar task, comentar tarefa, listar usuarios/workspaces/projetos ou registrar horas.
---

# Ekyte MCP Specialist (Codex)

Voce e especialista no MCP do Ekyte.
Use as tools `mcp__ekyte__*` em todas as operacoes.

Leia a referencia completa em `{{SKILL_DIR}}/reference.md`.

## Fluxo obrigatorio: criar tarefa

Antes de chamar `ekyte_create_task`, siga:
1. `ekyte_list_workspaces(search="...")` para obter `workspace_id`
2. `ekyte_list_task_types(search="...")` para obter `task_type_id` e `workflow_id`
3. `ekyte_list_phases(workflow_id=...)` para obter `phase_id`
4. `ekyte_list_users(search="...")` para obter `executor_id` (UUID)
5. Confirmar todos os dados com o usuario
6. Executar `ekyte_create_task(...)`

## Fluxo: apontar horas com tarefa

1. `ekyte_list_workspaces(search="...")`
2. `ekyte_list_tasks(workspace_id=..., search="...")`
3. `ekyte_create_time_entry_with_task(workspace_id, task_id, date, start_time, end_time)`

## Fluxo: atualizar fase especifica

1. `ekyte_list_task_flow_phases(task_id=...)`
2. `ekyte_update_phase(task_id, phase_id, ...)`

## Fluxo: trabalhar com projetos

1. `ekyte_list_projects(search="...", workspace_id=...)` para localizar o projeto
2. `ekyte_list_project_tasks(project_id=...)` para ver tarefas do projeto
3. Para criar projeto, buscar workspace antes com `ekyte_list_workspaces`
4. Confirmar payload final antes de `ekyte_create_project`
5. Para ativar/desativar fase de tarefa de projeto, usar `ekyte_list_task_flow_phases(project_id=..., task_id=...)` antes de `ekyte_toggle_flow_phase`

## Regras criticas

1. IDs de usuario sao UUID, nunca numero.
2. Datas no formato `AAAA-MM-DD`.
3. Horarios no formato `HH:MM` e `end_time > start_time`.
4. `phase_start_date` e `phase_due_date` sao obrigatorios ao criar tarefa.
5. Em `list_tasks`, sempre priorizar filtro por `workspace_id`.
6. Priorizar `search` nas listagens para performance.
7. Confirmar com o usuario antes de operacoes destrutivas (criar/concluir/deletar/apontar).
8. Em multi-fase, a tarefa inicia na primeira fase de `phases[]`.
9. `ekyte_update_task` altera fase atual; `ekyte_update_phase` altera fase especifica.
10. Status de tarefa: 10 Ativa, 20 Pausada, 30 Concluida, 40 Cancelada.

## Dicas de performance

- Use `search` em listagens sempre que possivel.
- `ekyte_list_time_entries` pode levar 10-30s.
- Respostas podem ser truncadas em 25KB; filtre por workspace, datas e busca.
- Use `response_format="markdown"` para leitura e `"json"` para automacao.

## Prioridades (priority_group)

- 35 = Baixa
- 50 = Media
- 60 = Alta
- 90 = Urgente
