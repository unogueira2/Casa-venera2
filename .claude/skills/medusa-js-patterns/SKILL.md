---
name: medusa-js-patterns
description: Conventions, architecture, and best practices for building with Medusa.js v2 (headless e-commerce). Use this skill whenever the user mentions Medusa, medusa-js, headless commerce backend, product modules, custom workflows, subscribers, API routes, admin extensions, or is building an e-commerce backend in Node/TypeScript. Also trigger for The PotFather project or any hemp/CBD/cannabis storefront backend work, since Medusa v2 is the recommended core there. Critical: Medusa v2 (released 2024) has a completely different architecture from v1 — always assume v2 unless the user explicitly says v1.
---

# Medusa.js v2 Patterns

Medusa v2 is a modular, framework-agnostic commerce platform. It is NOT the same as v1. Before writing any Medusa code, confirm the version and follow these conventions.

## Core v2 Architectural Concepts

### Modules
Everything in Medusa v2 is a module. Core modules: `product`, `cart`, `order`, `customer`, `inventory`, `pricing`, `promotion`, `payment`, `fulfillment`, `stock-location`, `sales-channel`, `region`, `tax`, `currency`, `user`, `auth`, `notification`, `file`.

Modules are isolated — they don't import each other directly. Cross-module relationships happen through **Module Links** (see below).

Custom modules live in `src/modules/<module-name>/` with this structure:
```
src/modules/my-module/
├── index.ts          # exports Module definition
├── service.ts        # business logic
├── models/           # data models (DML)
└── migrations/       # auto-generated
```

### Data Modeling Language (DML)
v2 uses a new DML instead of TypeORM entities. Models are defined with `model.define()`:

```typescript
import { model } from "@medusajs/framework/utils"

const Subscription = model.define("subscription", {
  id: model.id().primaryKey(),
  customer_id: model.text(),
  status: model.enum(["active", "paused", "cancelled"]),
  tier: model.text(),
  next_billing_date: model.dateTime(),
})

export default Subscription
```

Run `npx medusa db:generate <module-name>` then `npx medusa db:migrate` to apply.

### Module Links
To relate entities across modules (e.g., link a Product to a custom MysteryBoxTier), use links — NOT foreign keys.

```typescript
// src/links/product-mystery-box.ts
import { defineLink } from "@medusajs/framework/utils"
import ProductModule from "@medusajs/medusa/product"
import MysteryBoxModule from "../modules/mystery-box"

export default defineLink(
  ProductModule.linkable.product,
  MysteryBoxModule.linkable.mysteryBoxTier
)
```

### Workflows
Multi-step business logic belongs in **workflows**, not services or API routes. Workflows are composable, have built-in compensation (rollback) for failures, and are the ONLY correct place to orchestrate cross-module operations.

```typescript
import { createWorkflow, WorkflowResponse } from "@medusajs/framework/workflows-sdk"
import { createStep, StepResponse } from "@medusajs/framework/workflows-sdk"

const validateAgeStep = createStep(
  "validate-age",
  async (input: { customer_id: string }, { container }) => {
    const ageService = container.resolve("ageVerificationService")
    const verified = await ageService.check(input.customer_id)
    if (!verified) throw new Error("Age verification required")
    return new StepResponse({ verified: true })
  }
)

export const createHempOrderWorkflow = createWorkflow(
  "create-hemp-order",
  (input: { customer_id: string; cart_id: string }) => {
    const validated = validateAgeStep({ customer_id: input.customer_id })
    // ... more steps
    return new WorkflowResponse(validated)
  }
)
```

### API Routes
File-based routing in `src/api/`:
- `src/api/store/*` — storefront endpoints (public, use `publishable_api_key`)
- `src/api/admin/*` — admin endpoints (authenticated)
- `src/api/custom/*` — your own namespace

```typescript
// src/api/store/mystery-box/route.ts
import type { MedusaRequest, MedusaResponse } from "@medusajs/framework/http"

export const GET = async (req: MedusaRequest, res: MedusaResponse) => {
  const service = req.scope.resolve("mysteryBoxService")
  const tiers = await service.listTiers()
  res.json({ tiers })
}
```

Always resolve services via `req.scope.resolve()` — never instantiate directly.

### Subscribers (Event Handlers)
Listen to events in `src/subscribers/`:

```typescript
// src/subscribers/order-placed-hemp-compliance.ts
import { SubscriberArgs, SubscriberConfig } from "@medusajs/framework"

export default async function orderPlacedHandler({
  event: { data },
  container,
}: SubscriberArgs<{ id: string }>) {
  const orderService = container.resolve("orderService")
  const complianceService = container.resolve("complianceService")
  const order = await orderService.retrieve(data.id)
  await complianceService.logShipmentCompliance(order)
}

export const config: SubscriberConfig = {
  event: "order.placed",
}
```

### Scheduled Jobs
File-based in `src/jobs/`:

```typescript
// src/jobs/sync-dropship-inventory.ts
export default async function syncInventory({ container }) {
  const sync = container.resolve("dropshipSyncService")
  await sync.pullFromSources()
}

export const config = {
  name: "sync-dropship-inventory",
  schedule: "0 */6 * * *", // every 6h
}
```

## Common Tasks

### Creating a custom module
1. `npx medusa plugin:add my-module` OR create `src/modules/my-module/`
2. Define model in `models/<name>.ts`
3. Write service in `service.ts` extending `MedusaService`
4. Export from `index.ts`: `export default Module("myModule", { service: MyModuleService })`
5. Register in `medusa-config.ts` under `modules`
6. Run `npx medusa db:generate my-module && npx medusa db:migrate`

### Extending an existing entity (e.g., add fields to Product)
Do NOT modify core modules. Instead, create a custom module with your extension data and link it to the core entity via Module Links.

### Payment Provider
Custom payment providers implement the `IPaymentProvider` interface in `src/modules/payment-<name>/service.ts` and are registered under the `payment` module's providers array in config.

### Caching
Use the cache module: `container.resolve("cacheService")` — don't add Redis clients directly.

## Config Essentials (medusa-config.ts)

```typescript
import { defineConfig } from "@medusajs/framework/utils"

module.exports = defineConfig({
  projectConfig: {
    databaseUrl: process.env.DATABASE_URL,
    redisUrl: process.env.REDIS_URL,
    http: {
      storeCors: process.env.STORE_CORS,
      adminCors: process.env.ADMIN_CORS,
      authCors: process.env.AUTH_CORS,
      jwtSecret: process.env.JWT_SECRET,
      cookieSecret: process.env.COOKIE_SECRET,
    },
  },
  modules: [
    { resolve: "./src/modules/mystery-box" },
    { resolve: "./src/modules/compliance" },
    // core modules are auto-loaded, only list custom/overrides
  ],
})
```

## Anti-patterns to avoid

- **Don't** write cross-module logic in services — use workflows.
- **Don't** create foreign keys between modules — use Module Links.
- **Don't** use TypeORM decorators (`@Entity`, `@Column`) — that's v1 syntax.
- **Don't** call `MedusaModule.bootstrap()` manually — the framework handles it.
- **Don't** put business logic in API routes — delegate to workflows or services.
- **Don't** import services directly — always go through `container.resolve()`.

## When the user says something ambiguous

- "Medusa" with no version → ask or default to v2
- "Add a field to product" → propose a custom module + link, not core modification
- "Create an order from backend" → use the `createOrderWorkflow`, don't manually insert rows
- "Schedule a job" → `src/jobs/`, not node-cron directly

## Useful commands

```bash
npx medusa new my-store              # scaffold new project
npx medusa db:generate <module>      # generate migrations
npx medusa db:migrate                # apply migrations
npx medusa db:seed                   # run seed
npx medusa develop                   # dev server
npx medusa exec <script>             # run a one-off script
```

## References

- Official docs: https://docs.medusajs.com
- v2 migration guide: https://docs.medusajs.com/resources/medusa-v1-to-v2
- Framework source: https://github.com/medusajs/medusa
