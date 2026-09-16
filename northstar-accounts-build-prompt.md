# Build Prompt: Northstar Accounts

Use this as the instruction prompt for an AI coding agent to scaffold and build the **Northstar Accounts** project.

---

## Context

You are building **Accounts**, the first independently deployable remote in the Northstar commercial-banking platform.

Northstar consists of four separately owned applications:

1. **Shell** — the Angular 22 Native Federation host. It owns authentication/session context, customer context, entitlements, global navigation, platform telemetry, notifications, feature flags, global error handling, and the shared visual frame.
2. **Accounts** — this project. It owns account summaries, balances, account details, transaction history, and statement discovery.
3. **Payments** — a future remote. It will own ACH, wires, recipients, approvals, and payment history.
4. **Trading** — a future remote. It will own bond market data, positions, orders, and risk.

Accounts must operate in two modes:

- **Standalone development mode** on port `4201`, with a small development harness supplying the platform contract.
- **Federated mode** beneath the Shell’s stable `/accounts/*` route, loaded at runtime through Native Federation.

The Shell is a separate repository owned by a separate platform team. Accounts does not own the Shell’s internals and must never import them. Accounts consumes only the versioned shared runtime contract, such as `@northstar/platform-contract`, and exposes one stable federation entry point back to the Shell.

This project has two teaching goals:

1. **Modern Angular (the “how”):** Angular 22, standalone components, zoneless change detection, explicit `OnPush`, Signals and `computed()` for application state, `httpResource()` for request-driven data, modern template control flow, lazy routing, and Native Federation as a **remote**.
2. **Enterprise architecture (the “why”):** owning a bounded context, adapting an API contract at the application edge, consuming a host contract without coupling to host implementation, handing work to another domain through navigation, isolating failures, preserving accessibility, and deploying independently.

The code should teach both. **Comment generously at architectural decision points.** Comments must explain ownership, coupling, security, performance, and tradeoffs rather than translating TypeScript syntax into English.

Target comment quality:

```ts
// The API DTO stays inside data-access. Components consume AccountSummary,
// which is owned by this remote. That prevents a backend rename such as
// available_balance -> availableFunds from spreading through every template.
```

Assume the reader is an experienced Angular 2–16 developer and UI Architect who is learning Angular 22, Signals, zoneless rendering, and runtime micro-frontends.

---

## Accounts Mission

> Give an authorized commercial-banking customer a clear, fast, trustworthy view of what money they have, where it is, and what has happened to it.

Accounts is primarily an information and discovery domain. It may help the user begin a related task, but it does not move money.

The finished remote should answer four user questions:

1. What accounts can I access?
2. What is the current and available balance of each account?
3. What transactions changed an account?
4. Where can I find statements and account details?

---

## What Accounts Owns

Accounts is a bounded context containing the following capabilities.

### Account portfolio

- Account summary list for the selected organization/customer.
- Grouping by account type.
- Current balance, available balance, and currency display.
- Total current and available balances as derived UI values.
- Clear “as of” timestamps.

### Account detail

- Account name and masked account identifier.
- Account type and status.
- Current balance and available funds.
- Routing metadata suitable for display.
- Account nickname editing as a local simulated interaction, if implemented.
- Links to transaction history and statements.

### Transaction history

- Paginated or incrementally loaded transactions.
- Date-range filtering.
- Transaction-type filtering.
- Credit/debit filtering.
- Description search.
- Running or posted balance when present in the contract.
- Pending versus posted status.
- Sort newest-first by default.
- Accessible empty, loading, and error states.

### Statements

- Statement metadata list: period, generated date, document type, and availability.
- A clearly simulated download interaction.
- No fake generation of sensitive production documents.

### Internal routing

Accounts owns routes beneath the Shell mount point:

```text
/accounts
/accounts/:accountId
/accounts/:accountId/transactions
/accounts/:accountId/statements
```

The Shell knows only that `/accounts/*` belongs to Accounts. It must not need to understand these internal pages.

### Local domain state

- Selected account.
- Filter and sort state.
- Account and transaction resource state.
- Derived totals.
- Local presentation preferences.

That state stays inside Accounts. Do not place it in the Shell store or a platform-wide global store.

---

## What Accounts Explicitly Does NOT Own

### Money movement

Accounts must not implement:

- ACH creation;
- wire-transfer creation;
- recipients;
- payment validation or limits;
- payment approvals;
- payment status or history.

A **Make Payment** action is a handoff to Payments, not the beginning of a Payments workflow inside Accounts.

Use navigation as the contract:

```text
/payments/wires?sourceAccount=84721
```

Accounts supplies only the stable, opaque account identifier. Payments must independently validate the parameter, verify the user’s entitlement, and retrieve any account information it needs from its approved API contract.

Accounts must never import a Payments component, service, model, store, or repository.

### Platform responsibilities

Accounts does not own:

- login or logout;
- token acquisition or refresh;
- global customer selection;
- global header or navigation;
- Shell notifications;
- platform feature-flag infrastructure;
- the Shell’s remote manifest;
- global telemetry transport;
- Shell error handling;
- the design-system implementation.

Accounts consumes those capabilities through the shared platform contract.

### Other domain responsibilities

Accounts does not own Trading positions, investment pricing, orders, or risk. An investment/custody account may appear in an account summary, but market valuation and trading workflows belong to Trading.

---

## Data Source: Local Banking API Simulator

Build Accounts against a small local HTTP simulator that behaves like a backend-for-frontend, not against hardcoded arrays embedded in Angular services.

Use a lightweight mock API process such as `json-server` on port `3001`, or an equivalently simple local HTTP fixture server. Include the simulator and seed data in this repository so a developer can run the remote without external credentials.

Recommended endpoints:

```text
GET /api/customers/:customerId/accounts
GET /api/accounts/:accountId
GET /api/accounts/:accountId/transactions
GET /api/accounts/:accountId/statements
```

If the chosen simulator cannot naturally express nested endpoints, add a thin mock-server configuration or middleware rather than forcing awkward URLs into the Angular domain model.

Support realistic query parameters for transactions:

```text
from=2026-08-01
to=2026-09-16
type=ach
direction=debit
search=acme
page=1
pageSize=25
sort=-postedAt
```

### Why use an HTTP simulator

The learning objective is not simply to render fixture data. It is to preserve the same boundary the production application would use:

```text
Angular remote
    -> HTTP API contract
        -> Accounts BFF/domain APIs
            -> core banking systems
```

The simulator allows loading, latency, retry, error, empty, and cancellation behavior to be exercised without inventing a production banking backend.

### Simulated customer data

Seed the simulator with `Gokey Farms LLC`, customer ID `C98342`, and at least:

- Operating Account — current balance `$248,721.18`.
- Payroll Account — current balance `$82,419.06`.
- Investment Account — current balance `$612,850.44`.

The total current balance should equal `$943,990.68` so the derived `computed()` value can be verified in tests.

Include at least 35 realistic transactions across the deposit accounts so filtering and pagination can be demonstrated. Use invented vendors and clearly fictional identifiers. Do not use real account numbers, personal information, or credentials.

Include pending and posted items, credits and debits, and representative types such as:

- ACH credit/debit;
- wire debit;
- check;
- card purchase;
- transfer;
- fee;
- interest;
- deposit.

Include at least six statement metadata records per eligible account.

### Production migration seam

The API base URL must come from runtime/environment configuration. Do not hardcode `http://localhost:3001` throughout services.

Document how the mock repository can later point to an Accounts BFF without changing components or domain models.

---

## Anti-Corruption Layer

Treat the external API as an untrusted boundary even though the first implementation is a local simulator.

Define transport DTOs inside `data-access/api` using API-shaped names, for example:

```ts
interface AccountApiDto {
  account_id: string;
  customer_id: string;
  account_name: string;
  account_type: string;
  masked_number: string;
  current_balance: number;
  available_balance: number;
  currency_code: string;
  status: string;
  balance_as_of: string;
}
```

Map these DTOs through dedicated adapters into Accounts-owned models:

```ts
interface AccountSummary {
  id: AccountId;
  name: string;
  type: AccountType;
  maskedNumber: string;
  currentBalance: Money;
  availableBalance: Money;
  status: AccountStatus;
  balanceAsOf: Date;
}
```

No component, template, store, or feature service outside the data-access boundary should know snake_case API fields, response envelopes, pagination headers, or backend enum spellings.

The adapter must:

- validate required identifiers;
- normalize enum values;
- parse timestamps explicitly;
- preserve currency rather than assuming every value is USD;
- reject or safely map malformed numeric data;
- mask display identifiers defensively;
- report mapping failures through the telemetry contract without logging sensitive payloads.

This is not unnecessary ceremony. Banking APIs change, legacy systems use inconsistent contracts, and component code should not absorb that instability.

---

## Domain Models

Use explicit Accounts-owned models. Suggested shapes include:

```ts
type AccountId = string & { readonly __brand: 'AccountId' };

interface Money {
  readonly amount: number;
  readonly currency: string;
}

type AccountType =
  | 'operating'
  | 'payroll'
  | 'savings'
  | 'investment'
  | 'credit';

type AccountStatus = 'active' | 'restricted' | 'closed';

interface AccountSummary {
  readonly id: AccountId;
  readonly name: string;
  readonly type: AccountType;
  readonly maskedNumber: string;
  readonly currentBalance: Money;
  readonly availableBalance: Money;
  readonly status: AccountStatus;
  readonly balanceAsOf: Date;
}

type TransactionStatus = 'pending' | 'posted' | 'reversed';
type TransactionDirection = 'credit' | 'debit';

interface AccountTransaction {
  readonly id: string;
  readonly accountId: AccountId;
  readonly description: string;
  readonly transactionType: string;
  readonly direction: TransactionDirection;
  readonly amount: Money;
  readonly status: TransactionStatus;
  readonly effectiveAt: Date;
  readonly postedAt: Date | null;
  readonly runningBalance: Money | null;
}
```

Do not use JavaScript floating-point arithmetic for production-grade money calculations without documenting the limitation. For this UI teaching simulator, decimal numbers are acceptable for display and simple totals, but place money operations behind a helper and explain that a production system should use integer minor units or a decimal library.

---

## Shared Platform Contract

Accounts consumes a versioned contract such as:

```text
@northstar/platform-contract
```

It may use only documented public interfaces or injection tokens for:

- readonly authenticated session;
- selected customer/organization context;
- entitlement checks;
- platform navigation;
- structured telemetry;
- correlation IDs;
- feature flags;
- platform notifications;
- typed cross-domain events when navigation is insufficient.

Accounts must not import from the Shell repository using relative paths, Git URLs, or source aliases.

### Standalone development harness

Because Accounts must run outside the Shell, create development-only providers implementing the same platform contract:

- mock authenticated user;
- customer `C98342` / `Gokey Farms LLC`;
- full Accounts entitlement;
- console telemetry adapter;
- local router-backed platform navigation;
- deterministic feature flags.

Production/federated configuration must receive platform services from the shared contract integration, while standalone configuration supplies development adapters. Feature code should not need conditional checks such as `if (runningInsideShell)`.

---

## Native Federation Remote Contract

Configure `@angular-architects/native-federation` for Angular 22 as a **remote** using Angular’s esbuild-based application builder.

Use:

```text
logical remote name: accounts
development port:    4201
exposed module:      ./Routes
export name:         ACCOUNTS_ROUTES
Shell mount path:    /accounts
required entitlement: accounts.read
```

Expose routed configuration rather than a single giant component:

```ts
export const ACCOUNTS_ROUTES: Routes = [
  {
    path: '',
    loadComponent: () =>
      import('./features/account-overview/account-overview.page')
        .then((module) => module.AccountOverviewPage),
  },
  // Remaining routes are relative to the Shell's /accounts mount point.
];
```

The remote must not define `/accounts` again inside its exposed child routes. The Shell owns the mount prefix; Accounts owns paths beneath it.

Share Angular framework packages and the platform-contract package as strict singletons. Document the compatibility policy. Do not casually share every Accounts dependency; unnecessary runtime sharing increases coupling between release trains.

The remote must have its own `bootstrap.ts` and local application routes so it can run standalone, while its exposed route file remains safe for the Shell to load.

---

## Cross-Remote Handoff: Make Payment

On an eligible deposit account, provide a **Make Payment** action.

Accounts should use the platform navigation contract to request:

```text
/payments/wires?sourceAccount=<opaque-account-id>
```

Rules:

- Check `payments.create` before showing an enabled action.
- Do not treat a visible button as proof of authorization.
- Pass only the opaque account ID, not balances, account numbers, or a serialized account object.
- Do not write directly to Payments state.
- Do not publish a general event if ordinary navigation fully expresses the intent.
- In standalone mode, the development navigation adapter may log or display the intended handoff because Payments is not running.
- Add contract tests for the generated destination URL.

This demonstrates loose coupling: Accounts states the destination and intent; Payments owns the receiving workflow.

---

## Requirements

### Angular and TypeScript

- Angular 22.x.
- Standalone components only; no NgModules.
- Zoneless change detection; no Zone.js dependency.
- Explicit `ChangeDetectionStrategy.OnPush` on all routed and presentation components.
- Signals for synchronous feature and UI state.
- Writable Signals remain private to their owner.
- `computed()` for totals, counts, selected-account derivation, filter summaries, and view state.
- `httpResource()` for account, detail, transaction, and statement reads where it fits the request/response lifecycle.
- Use RxJS only for stream behavior that Signals/resources do not express cleanly. Explain each such decision.
- Modern template syntax: `@if`, `@for`, `@switch`; no `*ngIf` or `*ngFor`.
- Signal inputs/outputs where appropriate.
- Strict TypeScript and strict template checking.
- No `any` outside a documented external boundary.

### Routing and data loading

- Lazy-load each routed page.
- Route parameters must drive resource requests reactively.
- Validate route account IDs before requesting data.
- Preserve transaction filters in query parameters so filtered views are linkable and refresh-safe.
- Avoid duplicate requests when navigating between child views.
- Cancel or supersede stale searches/filter requests.
- Provide route titles containing the account display name when safely available.

### State architecture

Prefer small feature stores/facades aligned with workflows:

- `AccountPortfolioStore` — summary resource and derived totals.
- `AccountDetailStore` — selected account/detail resource.
- `TransactionSearchStore` — filters, paging, and transaction resource.
- `StatementStore` — statement metadata resource.

Do not create an all-purpose `AccountsStore` containing every state variable. Do not put server responses into a global mutable cache without a clear invalidation strategy.

### Loading, empty, and error states

Every data-backed page must distinguish:

- initial loading;
- refresh loading while old data remains visible;
- successful data;
- valid empty data;
- authorization failure;
- not found;
- network/server failure;
- malformed response/adapter failure.

An empty transaction list is not automatically an error. A customer receiving zero accounts unexpectedly may indicate context, entitlement, or API behavior and should be explained carefully.

Provide finite retry actions. Do not create unbounded automatic retry loops.

### Performance

- Establish a remote bundle budget.
- Lazy-load detail, transaction, and statement pages.
- Use stable `track` expressions in `@for` blocks.
- Avoid recomputing formatted currency/date values unnecessarily.
- Use virtual scrolling only if the demonstrated data volume justifies its complexity.
- Prefer server-side filtering/paging contracts even though the first server is a simulator.
- Record resource timing through platform telemetry.
- Do not ship the entire mock database in the browser bundle.

### Accessibility

- WCAG 2.2 AA target.
- Semantic headings and landmarks that work both standalone and inside the Shell.
- Do not create a second global `<main>` landmark when federated if the Shell contract expects the remote to render within its main content region; document the chosen landmark strategy.
- Account cards must remain operable by keyboard.
- Data tables require proper headers, captions, and responsive behavior.
- Filters require visible labels and accessible validation.
- Loading state must not repeatedly steal focus.
- Error summaries and retry actions must be keyboard reachable.
- Credits/debits and account status must not be communicated by color alone.
- Masked account numbers require understandable accessible labels.
- Currency values must include currency context.
- Respect reduced-motion and high-contrast preferences.

### Security and privacy

- Never display full account numbers.
- Never place full identifiers, balances, transactions, statement metadata, or customer information in telemetry.
- Do not persist banking data in `localStorage` or `sessionStorage`.
- Treat route/query parameters as untrusted.
- Encode all navigation parameters.
- Do not use `innerHTML` for transaction descriptions.
- Account authorization is enforced by APIs; client filtering is not security.
- Document BFF/token expectations without implementing a fake production token flow.
- Keep simulator data clearly fictional.

### Observability

Use the Shell-defined telemetry envelope and emit events such as:

```text
accounts.remote.mounted
accounts.portfolio.load.started
accounts.portfolio.load.succeeded
accounts.portfolio.load.failed
accounts.detail.load.succeeded
accounts.transactions.search.succeeded
accounts.transactions.search.failed
accounts.statements.load.failed
accounts.payment_handoff.requested
accounts.adapter.mapping_failed
```

Include duration, outcome, route, application version, and correlation ID where appropriate. Use allowlisted metadata. Never include balances, transaction descriptions, account numbers, raw API payloads, or tokens.

### Feature flags

Consume feature flags through the platform contract. Suggested typed flags:

- `accounts.statements.enabled`
- `accounts.nickname.enabled`
- `accounts.payment-handoff.enabled`

Flags control availability, not authorization. `payments.create` remains required for the payment handoff even when the feature flag is enabled.

---

## Suggested User Experience

### Account overview

Show:

- page title and concise context;
- last-updated timestamp;
- total current balance;
- total available funds;
- account cards or a responsive account table;
- account name, type, masked number, balance, available funds, and status;
- navigation to account details;
- refresh action;
- loading skeletons and accessible status text.

### Account detail

Show:

- account identity and masked number;
- current and available balance;
- account status and “as of” time;
- recent transaction preview;
- links to all transactions and statements;
- Make Payment handoff when eligible and authorized.

### Transaction history

Show:

- date range;
- search;
- type, direction, and status filters;
- active-filter summary and clear action;
- responsive transaction table/list;
- pending/posting status;
- amount and running balance;
- page controls or load-more behavior;
- filter state in the URL.

### Statements

Show:

- statement period;
- generated date;
- type;
- availability;
- simulated download action with an explicit teaching note.

Keep the visual language aligned with Northstar’s professional commercial-banking Shell. The remote should feel like part of the platform without depending on undocumented Shell CSS.

---

## Suggested Project Structure

```text
accounts-mfe/
├── mock-api/
│   ├── fixtures/
│   │   ├── accounts.json
│   │   ├── transactions.json
│   │   └── statements.json
│   ├── middleware.mjs
│   └── server.mjs
├── src/
│   ├── app/
│   │   ├── data-access/
│   │   │   ├── adapters/
│   │   │   ├── api/
│   │   │   ├── repositories/
│   │   │   └── runtime-config/
│   │   ├── domain/
│   │   │   ├── account.models.ts
│   │   │   ├── money.ts
│   │   │   ├── statement.models.ts
│   │   │   └── transaction.models.ts
│   │   ├── features/
│   │   │   ├── account-detail/
│   │   │   ├── account-overview/
│   │   │   ├── statements/
│   │   │   └── transactions/
│   │   ├── platform/
│   │   │   ├── development-contract/
│   │   │   ├── payment-handoff/
│   │   │   └── telemetry/
│   │   ├── shared-ui/
│   │   │   ├── account-identity/
│   │   │   ├── currency-value/
│   │   │   ├── empty-state/
│   │   │   └── resource-status/
│   │   ├── app.config.ts
│   │   ├── app.routes.ts
│   │   └── app.ts
│   ├── bootstrap.ts
│   ├── exposed.routes.ts
│   ├── main.ts
│   └── styles.scss
├── ARCHITECTURE.md
├── CONTRACT.md
├── DATA-CONTRACT.md
├── DEVELOPMENT.md
├── federation.config.mjs
├── package.json
└── README.md
```

The exact filenames may evolve, but the distinction between external DTOs, Accounts domain models, feature state, shared-platform integration, and federation entry point must remain obvious.

Do not create a generic `shared` folder that becomes an ownership junk drawer. `shared-ui` may contain only Accounts-owned presentation primitives used by multiple Accounts features.

---

## Testing Requirements

Use Angular 22’s test tooling and Vitest.

### Unit tests

- API DTO-to-domain adapters.
- Malformed DTO handling.
- Money total helper and currency mismatch behavior.
- Portfolio derived totals.
- Account sorting/grouping.
- Transaction filter serialization and parsing.
- Payment-handoff URL construction.
- Feature-flag plus entitlement decisions.
- Telemetry metadata allowlisting.

### Component tests

- Loading, populated, empty, and error states.
- Account cards and detail presentation.
- Keyboard-operable actions.
- Transaction filters and accessible labels.
- Restricted/closed account behavior.
- Statements disabled by feature flag.

### Route and contract tests

- `ACCOUNTS_ROUTES` is exported from `./Routes`.
- Child routes are relative and do not redefine `/accounts`.
- Invalid account IDs do not invoke repositories.
- The payment handoff produces `/payments/wires?sourceAccount=...`.
- Standalone and federated provider configurations satisfy the same platform interfaces.

### Integration tests

- Mock HTTP API returns and adapts portfolio data.
- Filtering/pagination query parameters reach the mock server correctly.
- Network failure produces a recoverable UI state.
- Retry succeeds after a simulated failure.
- A missing account returns a not-found experience rather than a generic crash.

### Accessibility checks

- Automated accessibility checks on all main pages.
- Keyboard path through account overview, detail, filters, and retry actions.
- Screen-reader-friendly masked account and currency labels.

---

## Documentation Deliverables

### 1. `README.md`

Include:

- Accounts mission;
- prerequisites;
- dependency installation;
- mock API startup;
- standalone startup on port `4201`;
- tests and production build;
- recommended code-reading order;
- how the Shell loads the remote;
- measured build output;
- next architectural step: Payments.

### 2. `ARCHITECTURE.md`

Explain:

- Accounts bounded context;
- what Accounts refuses to own;
- standalone versus federated composition;
- Signal stores and resource state;
- why the anti-corruption layer exists;
- why Shell state is not imported;
- why Make Payment is navigation rather than a service call;
- error isolation and retry strategy;
- privacy and telemetry constraints;
- accessibility ownership;
- independent deployment and its tradeoffs.

### 3. `CONTRACT.md`

Define:

- logical remote name `accounts`;
- port `4201`;
- exposed module `./Routes`;
- export `ACCOUNTS_ROUTES`;
- mount path `/accounts`;
- required entitlement `accounts.read`;
- platform-contract version expectations;
- session/customer fields consumed;
- telemetry events emitted;
- feature flags consumed;
- payment-handoff route contract;
- prohibited Shell imports;
- compatibility and deprecation policy.

### 4. `DATA-CONTRACT.md`

Define:

- mock/production endpoint shapes;
- request parameters;
- DTO response envelopes;
- pagination metadata;
- domain adapter behavior;
- enum normalization;
- date/time and currency rules;
- error responses;
- sensitive-field handling;
- production BFF migration notes.

### 5. `DEVELOPMENT.md`

Include:

- starting the mock API;
- starting Accounts standalone;
- connecting Accounts to the local Shell manifest;
- testing deep links;
- testing alternate entitlements;
- simulating latency, 401/403, 404, 500, malformed data, and empty results;
- production build and bundle inspection;
- Native Federation troubleshooting;
- Git workflow for an independent `accounts-mfe` repository.

---

## Definition of Done

- [ ] Angular 22 standalone remote created.
- [ ] Native Federation remote named `accounts` configured.
- [ ] `./Routes` exposes `ACCOUNTS_ROUTES`.
- [ ] Standalone development mode runs on port `4201`.
- [ ] Federated mode loads beneath Shell `/accounts`.
- [ ] Zoneless change detection enabled.
- [ ] `OnPush` used consistently.
- [ ] Modern template control flow used throughout.
- [ ] Shared platform contract consumed without Shell source imports.
- [ ] Development platform-contract adapters provided.
- [ ] Mock HTTP API runs independently on port `3001`.
- [ ] Runtime API configuration is not hardcoded through feature services.
- [ ] API DTOs remain inside the data-access boundary.
- [ ] DTO-to-domain adapters implemented and tested.
- [ ] Account overview implemented.
- [ ] Derived total balance and available funds implemented with `computed()`.
- [ ] Account detail implemented.
- [ ] Transaction history, filtering, search, paging, and URL state implemented.
- [ ] Statement metadata list implemented.
- [ ] Make Payment handoff implemented through platform navigation.
- [ ] Payment handoff checks feature flag and `payments.create` entitlement.
- [ ] Loading, refresh, empty, not-found, unauthorized, malformed-data, and server-error states implemented.
- [ ] Finite retry behavior implemented.
- [ ] Structured telemetry events implemented without sensitive data.
- [ ] Correlation ID propagated through the platform contract/API request headers where appropriate.
- [ ] Responsive and keyboard-accessible design implemented.
- [ ] WCAG 2.2 AA expectations checked.
- [ ] Unit, component, route/contract, integration, and accessibility tests pass.
- [ ] Remote bundle budget passes.
- [ ] Production build passes.
- [ ] Accounts can be rebuilt and redeployed without rebuilding Shell.
- [ ] `README.md`, `ARCHITECTURE.md`, `CONTRACT.md`, `DATA-CONTRACT.md`, and `DEVELOPMENT.md` completed.
- [ ] No Payments or Trading business logic exists in the repository.

---

## Output Format

- Provide the complete proposed file tree first.
- Then provide the implementation file by file.
- Include meaningful inline developer comments at architectural decision points.
- Do not fill straightforward code with comments that merely restate syntax.
- Include the mock API simulator and fictional seed data.
- Include all tests as working project files, not as pseudocode or future recommendations.
- After the code, provide `README.md`, `ARCHITECTURE.md`, `CONTRACT.md`, `DATA-CONTRACT.md`, and `DEVELOPMENT.md`.
- End with:
  1. exact dependency-installation commands;
  2. exact mock-API and standalone-startup commands;
  3. exact test and production-build commands;
  4. the Shell manifest entry required to load Accounts;
  5. how to verify standalone and federated deep links;
  6. deliberate architectural decisions and tradeoffs;
  7. any steps requiring human configuration.

Do not silently skip the API adapter boundary, standalone harness, payment handoff, failure states, accessibility, tests, or documentation to save time. They are central to the exercise.

Most importantly, do not solve cross-domain integration by importing Shell or Payments internals. Accounts earns its independent deployment by respecting those boundaries even when direct imports would appear faster.
