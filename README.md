# Ekyte MCP Server for Codex

Servidor MCP (Model Context Protocol) para conectar o Codex ao Ekyte e operar tarefas, projetos, fases, usuarios, workspaces e apontamentos de horas por tools.

Descricao sugerida para o GitHub:

```text
MCP Server do Ekyte para Codex: cria e atualiza tarefas/projetos, lista workspaces/usuarios/fases e gerencia apontamentos de horas via API interna do Ekyte.
```

## Recursos

- 21 tools MCP: 11 de leitura e 10 de escrita.
- Transporte local via `stdio` para uso direto no Codex CLI.
- Transporte remoto via HTTP em `/mcp` para deploy com Docker.
- Health check HTTP em `/health`.
- Skill do Codex inclusa em `codex/skills/ekyte`.
- PRD de instalacao guiada em `PRD-INSTALACAO-MCP-EKYTE-CODEX.md`.

## Requisitos

- Conta ativa no Ekyte com acesso a `https://app.ekyte.com`.
- `EKYTE_BEARER_TOKEN`: JWT Bearer copiado do header `Authorization`.
- `EKYTE_COMPANY_ID`: ID numerico da empresa no caminho `/api/companies/XXXX/...`.
- Codex CLI instalado para registrar o servidor MCP.
- Node.js 18 ou superior para instalacao local.
- Docker, dominio HTTPS e um host com porta 3000 disponivel para instalacao remota.

## Instalacao Rapida

Clone o repositorio:

```bash
git clone https://github.com/matheusbernardopro-source/ekyte_mcp_server.git
cd ekyte_mcp_server
```

Instale as dependencias e gere o build:

```bash
npm install
npm run build
```

Crie o `.env` a partir do exemplo:

```bash
cp .env.example .env
```

Preencha as variaveis obrigatorias:

```env
EKYTE_BEARER_TOKEN=seu_jwt
EKYTE_COMPANY_ID=1234
TRANSPORT=stdio
PORT=3000
```

## Guia de Instalacao no Codex

O caminho recomendado e pedir para o Codex executar o PRD:

```text
Executa este PRD de instalacao: @PRD-INSTALACAO-MCP-EKYTE-CODEX.md
```

O PRD cobre instalacao local, instalacao remota, registro da MCP e copia da skill.

### Registro local global

```bash
codex mcp add ekyte --env EKYTE_BEARER_TOKEN=SEU_JWT --env EKYTE_COMPANY_ID=SEU_COMPANY_ID --env TRANSPORT=stdio -- node /CAMINHO/ABSOLUTO/ekyte_mcp_server/dist/index.js
```

### Registro remoto global

```bash
codex mcp add ekyte --url https://SEU-SERVIDOR.com/mcp
```

### Registro por projeto

Crie ou atualize `./.codex/config.toml`:

```toml
[mcp_servers.ekyte]
command = "node"
args = ["/CAMINHO/ABSOLUTO/ekyte_mcp_server/dist/index.js"]

[mcp_servers.ekyte.env]
EKYTE_BEARER_TOKEN = "SEU_JWT"
EKYTE_COMPANY_ID = "SEU_COMPANY_ID"
TRANSPORT = "stdio"
```

Para servidor remoto:

```toml
[mcp_servers.ekyte]
url = "https://SEU-SERVIDOR.com/mcp"
```

Valide:

```bash
codex mcp list
codex mcp get ekyte
```

## Skill do Codex

A skill ensina o Codex a operar a MCP com fluxo seguro: listar IDs antes de criar, confirmar operacoes sensiveis e usar os formatos corretos de data, hora e UUID.

Arquivos:

- `codex/skills/ekyte/SKILL.md`
- `codex/skills/ekyte/reference.md`

Instalacao global no Windows:

```powershell
New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.codex\skills" | Out-Null
Copy-Item -Recurse codex\skills\ekyte "$env:USERPROFILE\.codex\skills\"
```

Instalacao global no Linux/macOS:

```bash
mkdir -p ~/.codex/skills
cp -r codex/skills/ekyte ~/.codex/skills/
```

## Deploy Remoto com Docker

Build da imagem:

```bash
docker build -t ekyte-mcp-server .
```

Execucao:

```bash
docker run -d --name ekyte-mcp-server \
  -p 3000:3000 \
  -e EKYTE_BEARER_TOKEN=SEU_JWT \
  -e EKYTE_COMPANY_ID=SEU_COMPANY_ID \
  -e TRANSPORT=http \
  -e PORT=3000 \
  ekyte-mcp-server
```

Valide:

```bash
curl http://localhost:3000/health
```

Endpoint MCP remoto:

```text
https://SEU-SERVIDOR.com/mcp
```

## Tools Disponiveis

Leitura:

- `ekyte_list_workspaces`
- `ekyte_list_users`
- `ekyte_list_task_types`
- `ekyte_list_phases`
- `ekyte_list_tasks`
- `ekyte_get_task`
- `ekyte_list_task_flow_phases`
- `ekyte_list_projects`
- `ekyte_list_project_templates`
- `ekyte_list_project_tasks`
- `ekyte_list_time_entries`

Escrita:

- `ekyte_create_task`
- `ekyte_update_task`
- `ekyte_update_phase`
- `ekyte_toggle_flow_phase`
- `ekyte_complete_task`
- `ekyte_add_task_comment`
- `ekyte_create_project`
- `ekyte_create_time_entry_with_task`
- `ekyte_create_time_entry_without_task`
- `ekyte_delete_time_entry`

## Como Obter Credenciais

Token JWT:

1. Acesse `https://app.ekyte.com`.
2. Abra DevTools > Network.
3. Abra uma request para `api.ekyte.com`.
4. Copie o valor do header `Authorization: Bearer ...`, sem o prefixo `Bearer `.

Company ID:

- Veja o numero no caminho da request: `/api/companies/XXXX/...`.

## Seguranca

- Nunca commite `.env`, tokens ou configs locais com credenciais.
- Trate `EKYTE_BEARER_TOKEN` como senha.
- Confirme dados antes de criar, concluir, deletar ou apontar horas.
- Use `executor_id` como UUID, nunca como numero.
- Use datas em `AAAA-MM-DD` e horarios em `HH:MM`.

## Desenvolvimento

```bash
npm run dev
npm run build
npm start
```

Smoke test HTTP:

```bash
curl http://localhost:3000/health
curl -X POST http://localhost:3000/mcp -H "Content-Type: application/json" -H "Accept: application/json, text/event-stream" -d '{"jsonrpc":"2.0","id":1,"method":"tools/list"}'
```
