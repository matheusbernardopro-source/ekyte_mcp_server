# Ekyte MCP - Referencia Completa (Codex)

## Tools de leitura (11)

### ekyte_list_workspaces
- Lista workspaces
- Filtros: `search`, `active_only`, `page`, `response_format`
- Retorna: `id`, `name`, `active`

### ekyte_list_users
- Lista usuarios/membros
- Filtros: `search`, `page`, `response_format`
- Retorna: `id` (UUID), `name`, `email`

### ekyte_list_task_types
- Lista tipos de tarefa
- Filtros: `search`, `active_only`, `page`, `response_format`
- Retorna: `id`, `name`, `workflow_id`, `group`

### ekyte_list_phases
- Lista fases de um workflow
- Parametro obrigatorio: `workflow_id`
- Retorna: `id`, `name`, `sequential`, `active`

### ekyte_list_tasks
- Lista tarefas
- Filtros principais: `workspace_id`, `status`, `task_type_id`, `phase_id`, `executor_id`, datas, `search`, `page`
- Retorna: `id`, `title`, `status`, `workspace`, `task_type`, `phase_id`, executor e datas

### ekyte_get_task
- Detalhes completos de uma tarefa
- Parametro obrigatorio: `task_id`

### ekyte_list_task_flow_phases
- Lista fases reais de uma tarefa
- Parametro obrigatorio: `task_id`
- Retorna `phase_id`, executor e datas por fase

### ekyte_list_time_entries
- Lista apontamentos de horas
- Obrigatorio: `workspace_id`, `date_from`, `date_to`
- Opcionais: `user_id`, `task_id`, `page`

### ekyte_list_projects
- Lista projetos
- Filtros: `search`, `workspace_id`, `status`, `is_planning`, `page`
- Retorna: `id`, `name`, `situation`, `is_planning`, `workspace_id`

### ekyte_list_project_templates
- Lista modelos/templates de projeto
- Filtros: `search`, `page`
- Retorna: `id`, `name`, `description`

### ekyte_list_project_tasks
- Lista tarefas de um projeto
- Obrigatorio: `project_id`
- Filtros: `search`, `status`, `page`

## Tools de escrita (10)

### ekyte_create_task
Dois modos:
- Fase unica: `executor_id` + `phase_id`
- Multi-fase: `phases[]` com `phase_id`, `executor_id`, `effort_minutes`, datas

Obrigatorios: `title`, `workspace_id`, `task_type_id`, `phase_start_date`, `phase_due_date`.

### ekyte_update_task
Atualiza campos top-level da tarefa (`title`, `description`, `executor_id`, `phase_id`, datas, `priority_group`, `priority`).

### ekyte_update_phase
Atualiza fase especifica (`executor_id`, `effort_minutes`, datas).

### ekyte_toggle_flow_phase
Ativa ou desativa uma fase no fluxo de uma tarefa de projeto. Requer `project_id`, `task_id`, `phase_id` e `active`.

### ekyte_complete_task
Conclui tarefa (`situation=30`).

### ekyte_add_task_comment
Adiciona comentario na timeline da tarefa.

### ekyte_create_project
Cria projeto no Ekyte. Busque o workspace antes e confirme o payload final com o usuario.

### ekyte_create_time_entry_with_task
Cria apontamento com tarefa.

### ekyte_create_time_entry_without_task
Cria apontamento sem tarefa.

### ekyte_delete_time_entry
Remove apontamento (soft delete).

## Tipos de ID
- Workspace, Task, Task Type, Phase, Workflow, Time Entry: `number`
- User/Executor: `UUID`

## Validacoes importantes
- `end_time > start_time`
- Datas no formato `AAAA-MM-DD`
- Horarios no formato `HH:MM`
- `executor_id` sempre UUID
- Sempre filtrar listagens por workspace quando possivel

## Erros comuns
- 400: parametros invalidos
- 401: token invalido/expirado
- 404: ID inexistente
- 500: erro temporario do Ekyte
- Timeout: endpoint de horas pode demorar
