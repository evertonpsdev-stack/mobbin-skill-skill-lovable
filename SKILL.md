---
name: mobbin-skill-skill-lovable
description: Use when the user wants to use, integrate, configure, or build with mobbin-skill — AI skill for structured UI/UX research using Mobbin MCP. Works with Claude Code, Cursor, Gemini CLI, Lovable, and any MCP-compatible agent. Activate when the conversation mentions mobbin-skill.
---

# mobbin-skill

> AI skill for structured UI/UX research using Mobbin MCP. Works with Claude Code, Cursor, Gemini CLI, Lovable, and any MCP-compatible agent.

**Repository:** https://github.com/ddruids/mobbin-skill
**Language:** —  ·  **Stars:** 17  ·  **License:** MIT


## When to use this skill

Use when the user wants to use, integrate, configure, or build with mobbin-skill — AI skill for structured UI/UX research using Mobbin MCP. Works with Claude Code, Cursor, Gemini CLI, Lovable, and any MCP-compatible agent. Activate when the conversation mentions mobbin-skill.

## Capabilities (from README)

- Translates abstract UX goals into concrete Mobbin search queries
- Searches both individual screens and multi-step user flows (onboarding, checkout, etc.)
- Visually analyzes returned screenshots (layout, hierarchy, components, color)
- Clusters findings into recurring UI patterns with linked references
- Derives research queries from PRDs and product briefs automatically
- Surfaces atomic design system primitives when asked about components
- Supports three output modes: research summary, competitive comparison, product decision log

## Workflow

1. **Orient** — Confirm what the user wants to do with mobbin-skill.
2. **Recall** — Abra `REFERENCES.md` e localize a seção do doc relevante (mirror dos paths originais).
3. **Configure** — Walk through installation/setup; ask for missing env vars or credentials.
4. **Build** — Generate code/config that matches the conventions in the docs.
5. **Validate** — Run the project's own checks (tests, lints, smoke calls) before declaring done.
6. **Cite** — Quando responder, mencione qual seção de `REFERENCES.md` você usou.

## Conventions

- Prefer official APIs/CLI documented in the repo over third-party wrappers.
- Surface required env vars and secrets explicitly; never inline real values.
- If the repo ships a CLI, prefer its commands over re-implementing logic.

## Sanitização de Dados (DLP — obrigatório)

Esta skill opera sob política **Zero Trust**. Antes de qualquer resposta, log ou
artefato gerado:

- **Nunca** exponha em logs, UI pública ou commits: chaves de API, tokens, senhas,
  strings de conexão, JWTs, dados de PII (CPF, CNPJ, e-mail real, telefone) ou
  conteúdo de arquivos `.env` reais.
- Ao citar valores sensíveis, substitua por placeholder: `ENV_VAR_PLACEHOLDER`,
  `*****` ou `<REDACTED>`.
- Toda credencial deve ser lida via `process.env.NOME_DA_VAR` — nunca hard-coded.
- Arquivos de configuração entregues ao usuário devem ser **templates** sem valores
  reais (ex.: `.env.example` com `API_KEY=ENV_VAR_PLACEHOLDER`).
- Antes de enviar dados do usuário a APIs terceiras, valide o destino e mascare
  PII que não seja estritamente necessária à chamada.

## Mitigar Riscos de Dados (mecanismo de ação)

Quando o usuário pedir **"mitigar riscos"**, **"sanitizar"** ou for solicitar
deploy/publicação, execute este protocolo antes de qualquer outra coisa:

1. **Validar** — varra o contexto atual e os arquivos gerados procurando:
   `sk_live_*`, `sk_test_*`, `pk_live_*`, `whsec_*`, `AIzaSy*`, `ghp_*`,
   `AKIA*`, `xox[baprs]-*`, `SG.*`, JWTs (`eyJ...`), connection strings
   (`postgres://user:pass@...`), CPF/CNPJ, e arquivos `.env` com valores reais.
2. **Interromper** — se algum padrão for encontrado, **pare o fluxo** e reporte
   o achado ao usuário antes de prosseguir. Não gere, não envie, não comite.
3. **Mascarar** — substitua o valor sensível por `ENV_VAR_PLACEHOLDER` (ou
   `*****` para PII) tanto no código quanto em logs/respostas.
4. **Corrigir o código** — converta chamadas inseguras (`fetch("https://api/?key=ABC123")`,
   credenciais inline em headers/SDK) para usar `process.env.X` e instrua o
   usuário a configurar a variável no ambiente.
5. **Substituir `.env` por `.env.example`** — se um `.env` real for detectado,
   gere um `.env.example` com placeholders e remova o arquivo original.
6. **Reportar** — ao final, devolva um sumário: o que foi mascarado, o que foi
   substituído, e quais ações o usuário ainda precisa executar (rotacionar
   chaves vazadas, configurar variáveis de ambiente, revisar diffs).

Se algum risco crítico não puder ser mitigado automaticamente, **recomende não
publicar** o artefato e oriente revisão manual.

## Reference index (consolidado em REFERENCES.md)

- `README.md` (consultar seção em `REFERENCES.md`)
- `SKILL.md` (consultar seção em `REFERENCES.md`)
- `references/query-patterns.md` (consultar seção em `REFERENCES.md`)
- `references/synthesis-framework.md` (consultar seção em `REFERENCES.md`)
- `_metadata.json` — Repo metadata snapshot.


## Complete repository map

```
- LICENSE
- README.md
- references/
  - query-patterns.md
  - synthesis-framework.md
- SKILL.md
```
