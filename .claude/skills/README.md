# The PotFather — Skills Pack

Oito skills prontas para deploy, focadas em construir The PotFather (hemp/THCA e-commerce) com Medusa.js + Next.js.

## Skills incluídas

| # | Skill | Propósito |
|---|-------|-----------|
| 1 | `medusa-js-patterns` | Convenções Medusa v2: módulos, DML, workflows, links, subscribers |
| 2 | `nextjs-14-app-router` | App Router: server components, server actions, streaming, middleware |
| 3 | `godfather-design-system` | Design tokens, tipografia, copy voice da marca The PotFather |
| 4 | `cannabis-compliance-checklist` | Estados bloqueados, age gate, COA, disclaimers, prohibited claims |
| 5 | `tailwind-v4-patterns` | CSS-first config, @theme, diferenças do v3 |
| 6 | `payment-gateways-hemp` | Aeropay, Authorize.net high-risk, NOWPayments, BitPay |
| 7 | `seo-cannabis` | Estratégia orgânica, schema.org, content pillars, link-building no vertical |
| 8 | `coa-parser` | Extração de dados de Certificate of Analysis (PDFs) |

## Como fazer deploy

### No Claude Code

1. Extraia o zip no seu projeto:
   ```bash
   unzip potfather-skills.zip -d ~/potfather-skills
   ```

2. Copie as pastas pro diretório de skills do Claude Code:
   ```bash
   mkdir -p ~/.claude/skills
   cp -r ~/potfather-skills/* ~/.claude/skills/
   ```

   OU, se quiser escopar só pro projeto The PotFather, coloque dentro do projeto:
   ```bash
   cd ~/projects/the-potfather
   mkdir -p .claude/skills
   cp -r ~/potfather-skills/* .claude/skills/
   ```

3. Reinicie o Claude Code. As skills aparecem automaticamente — Claude carrega as que forem relevantes pela descrição YAML.

### No Claude.ai

1. No app/web do Claude.ai, vá em **Settings → Capabilities → Skills** (ou no botão de ferramentas da conversa).
2. Clique em **Create skill** ou **Upload skill** para cada uma das 8 pastas.
3. Cole o conteúdo do `SKILL.md` correspondente OU faça upload do `.skill` empacotado (se tiver).
4. Salve. As skills ficam disponíveis em todas as suas conversas futuras.

**Dica**: skills em Claude.ai também podem ser ativadas a nível de **Project**. Se você criar um Project "The PotFather", adicione as skills só ali pra não poluir outras conversas.

## Ordem de prioridade pra ativar

Se quiser ir incremental em vez de subir tudo de uma vez:

**Semana 1** (fundação técnica):
- `medusa-js-patterns`
- `nextjs-14-app-router`
- `tailwind-v4-patterns`
- `godfather-design-system`

**Semana 2** (compliance + pagamento — antes de tocar em checkout):
- `cannabis-compliance-checklist`
- `payment-gateways-hemp`

**Semana 3+** (crescimento e operação):
- `seo-cannabis`
- `coa-parser`

## Manutenção

- **Compliance muda trimestralmente**: revise a lista de estados bloqueados em `cannabis-compliance-checklist` a cada 3 meses. Quando Farm Bill for renegociado, atualize a skill.
- **Design system evolui**: conforme você refinar mockups, atualize `godfather-design-system` com novos componentes e tokens. A skill vira seu single source of truth.
- **APIs deprecam**: se um processador mudar de endpoint (Aeropay v3 → v4, por exemplo), atualize `payment-gateways-hemp`.

## Customizações recomendadas

Antes de ir pra produção, edite:

1. **`godfather-design-system/SKILL.md`** — ajuste as cores exatas e fontes conforme o logo/branding final que você aprovar.
2. **`cannabis-compliance-checklist/SKILL.md`** — confirme a lista de estados bloqueados com seu advogado antes de lançar.
3. **`payment-gateways-hemp/SKILL.md`** — substitua os env vars placeholder pelos reais depois da aprovação do Aeropay.

## Próximos passos sugeridos

Depois de subir as skills, peça ao Claude Code:

> "Com base nas skills carregadas, cria o scaffolding inicial do projeto The PotFather: monorepo com `apps/storefront` (Next.js 14 App Router + Tailwind v4) e `apps/backend` (Medusa v2). Aplica o design system Godfather, implementa o state blocklist no middleware, e configura o Aeropay como payment provider custom."

Isso vai puxar 5-6 skills simultaneamente e gerar uma base já consistente com tudo que discutimos.

---

Boa construção. *The family is watching.*
