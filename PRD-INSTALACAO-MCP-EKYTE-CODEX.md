# PRD: Instalacao do MCP Server do Ekyte no Codex

> Jogue este arquivo no chat do Codex e diga: "Executa este PRD de instalacao".
> O Codex instala e conecta o MCP do Ekyte para operar tarefas e horas direto no chat.

---

## O que esse MCP faz

Com o MCP conectado no Codex, voce consegue:
- Criar e atualizar tarefas e projetos
- Apontar horas com ou sem tarefa
- Listar workspaces, usuarios, projetos, tipos e fases
- Completar tarefas e adicionar comentarios

Sao 21 tools: 11 de leitura e 10 de escrita.

## Pre-requisitos

- Codex CLI instalado
- Conta no Ekyte (app.ekyte.com)
- Node.js >= 18 (apenas para modo local)

---

## PRIMEIRA PERGUNTA: modo de instalacao

Pergunte ao usuario:
"Voce quer instalar o MCP do Ekyte de qual forma?"

- A) Local: roda `dist/index.js` na maquina local
- B) Remoto: aponta para URL de servidor MCP ja hospedado

---

## SEGUNDA PERGUNTA: escopo

Pergunte ao usuario:
"Quer instalar globalmente (todos os projetos) ou apenas neste repositorio?"

- Global: grava em `~/.codex/config.toml` via `codex mcp add`
- Projeto: grava em `./.codex/config.toml` na raiz do repo atual

---

## Caminho A: Instalacao Local

### 0) Verificacao inicial

Execute silenciosamente:
1. Verificar `package.json`
2. Verificar `.env` com `EKYTE_BEARER_TOKEN` e `EKYTE_COMPANY_ID`
3. Verificar `dist/index.js`

Regras:
- Se `.env` ja estiver preenchido, pular coleta de credenciais
- Se `dist/index.js` ja existir, pular build

### 1) Credenciais do Ekyte

Obrigatorias:
- `EKYTE_BEARER_TOKEN`
- `EKYTE_COMPANY_ID`

Se nao estiverem no `.env`, obter via DevTools no `app.ekyte.com`:
- Token: header `Authorization: Bearer ...`
- Company ID: trecho `/api/companies/XXXX/...`

### 2) Instalar e buildar

```bash
npm install
npm run build
```

Validar:

```bash
ls dist/index.js
```

### 3) Registrar MCP no Codex

#### Escopo Global

```bash
codex mcp add ekyte --env EKYTE_BEARER_TOKEN=SEU_JWT --env EKYTE_COMPANY_ID=SEU_COMPANY_ID --env TRANSPORT=stdio -- node /CAMINHO/ABSOLUTO/ekyte_mcp_server/dist/index.js
```

#### Escopo Projeto

Criar/atualizar `./.codex/config.toml`:

```toml
[mcp_servers.ekyte]
command = "node"
args = ["/CAMINHO/ABSOLUTO/ekyte_mcp_server/dist/index.js"]

[mcp_servers.ekyte.env]
EKYTE_BEARER_TOKEN = "SEU_JWT"
EKYTE_COMPANY_ID = "SEU_COMPANY_ID"
TRANSPORT = "stdio"
```

### 4) Validar registro

```bash
codex mcp list
```

Deve mostrar `ekyte` habilitado.

### 5) Instalar skill global do Codex

Copiar `codex/skills/ekyte/` para `~/.codex/skills/ekyte/`.

Linux/macOS:

```bash
mkdir -p ~/.codex/skills
cp -r codex/skills/ekyte ~/.codex/skills/
```

Windows (PowerShell):

```powershell
New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.codex\skills" | Out-Null
Copy-Item -Recurse codex\skills\ekyte "$env:USERPROFILE\.codex\skills\"
```

### 6) Smoke test

No Codex:
- "Liste os workspaces do Ekyte"
- "Quem sao os usuarios da empresa?"
- "Crie uma tarefa de teste no workspace X para o usuario Y"

---

## Caminho B: Instalacao Remota

### 1) Registrar MCP no Codex

#### Escopo Global

```bash
codex mcp add ekyte --url https://SEU-SERVIDOR.com/mcp
```

#### Escopo Projeto

Criar/atualizar `./.codex/config.toml`:

```toml
[mcp_servers.ekyte]
url = "https://SEU-SERVIDOR.com/mcp"
```

### 2) Validar registro

```bash
codex mcp list
```

### 3) Instalar skill global do Codex

Se tiver o repo clonado:

```powershell
New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.codex\skills" | Out-Null
Copy-Item -Recurse codex\skills\ekyte "$env:USERPROFILE\.codex\skills\"
```

Sem repo clonado, baixar arquivos direto do GitHub:

```powershell
$dst = "$env:USERPROFILE\.codex\skills\ekyte"
New-Item -ItemType Directory -Force -Path $dst | Out-Null
$base = "https://raw.githubusercontent.com/matheusbernardopro-source/ekyte_mcp_server/main/codex/skills/ekyte"
Invoke-WebRequest -Uri "$base/SKILL.md" -OutFile "$dst\SKILL.md"
Invoke-WebRequest -Uri "$base/reference.md" -OutFile "$dst\reference.md"
```

### 4) Smoke test

Mesmo teste do caminho local.

---

## Troubleshooting rapido

### "ERRO: Variaveis de ambiente obrigatorias nao definidas"
No modo local, confira `EKYTE_BEARER_TOKEN` e `EKYTE_COMPANY_ID`.

### Erro 401
Token expirado. Atualize token e reconfigure o MCP.

### Erro 500
Instabilidade temporaria do Ekyte. Tente novamente em alguns minutos.

### `ekyte` nao aparece em `codex mcp list`
1. Verificar caminho absoluto do `dist/index.js` (modo local)
2. Verificar URL com `/mcp` (modo remoto)
3. Verificar se `./.codex/config.toml` esta no repo certo (escopo projeto)
4. Remover/adicionar novamente no escopo global

### Build falhou
```bash
npm run clean
npm install
npm run build
```

### Skill nao carregou
1. Confirmar `~/.codex/skills/ekyte/SKILL.md` e `reference.md`
2. Reiniciar sessao do Codex

---

MCP + skill instalados no Codex: o assistente passa a operar o Ekyte com fluxo seguro e confirmacao nas operacoes destrutivas.





