# Build Prompt: Northstar Banking Shell

Use this as the instruction prompt for an AI coding agent to scaffold and build the **Northstar Banking Shell** project.

---

## Context

You are building **Northstar Banking Shell**, the host and platform foundation for a teaching example of enterprise Angular micro-frontend architecture.

Northstar is a commercial banking and fixed-income trading platform composed of four independently deployable applications:

1. **Shell** — the host and shared application platform built by this project.
2. **Accounts** — a future remote for balances, transactions, account details, and statements.
3. **Payments** — a future remote for ACH, wires, recipients, approvals, and payment history.
4. **Trading** — a future remote for bond market data, watchlists, positions, orders, and risk.

This project builds **only the Shell**. Accounts, Payments, and Trading will be created later as separate repositories, applications, pipelines, and build prompts. Do not quietly place their business features inside the Shell merely because the remotes do not exist yet.

The Shell should present believable navigation and clearly labeled local placeholder pages for those three future capabilities. Those placeholders prove the route, entitlement, layout, and failure-state contracts while keeping the business-domain boundary intact. Later, each placeholder will be replaced by a runtime-loaded remote without changing the public URL or global navigation.

This project has two teaching goals, and they are not the same thing:

1. **Modern Angular (the “how”):** Angular 22, standalone components, Signals as the default state model, zoneless change detection, `OnPush` change detection, modern template control flow, lazy routing, and Native Federation as a dynamic **host**.
2. **Enterprise UI architecture (the “why”):** making independently deployed business applications feel like one coherent platform; enforcing bounded-context ownership; defining narrow, versioned runtime contracts; separating authentication from authorization; handling missing remotes gracefully; and giving multiple autonomous teams a stable integration surface.

The code should teach both. **Comment generously and explain architectural reasoning, not merely syntax.**

A comment such as:

```ts
// Toggle the menu.
```

does not add value.

The expected quality bar is closer to:

```ts
// Writable navigation state remains private to the Shell. Remotes may react to
// platform context, but they must not be able to open or replace global chrome.
// Exposing a readonly Signal makes that ownership boundary visible in the API.
```

Assume the reader is an experienced Angular 2–16 developer and UI Architect who understands enterprise applications but is learning Angular 22, Signals, zoneless rendering, and runtime micro-frontend composition.

---

## Program Mission

Build a modern enterprise banking platform that supports independently owned business capabilities while preserving a unified, secure, accessible, observable, and high-performance customer experience.

The Shell’s mission within that program is:

> Make several independently developed and deployed banking applications feel like one trustworthy application, without taking ownership of their business logic.

The end user should see one Northstar platform. They should not need to know when navigation crosses from the host into a remote.

---

## What the Shell Owns

The Shell is the enterprise application platform. It owns capabilities that must remain consistent across every banking domain.

### Application startup and composition

- Bootstrap the Angular 22 application.
- Initialize Native Federation before Angular starts.
- Load remote locations from a runtime federation manifest.
- Mount remotes beneath stable, Shell-owned route prefixes.
- Continue operating when no remotes are configured.
- Show a useful failure experience if one remote is unavailable.

### Authentication and session context

- Provide one source of truth for the signed-in user.
- Provide the selected organization/customer context.
- Model authentication state, session status, and user identity through a narrow platform contract.
- Use a clearly identified mock session provider for this teaching build.
- Create a seam that can later be replaced by OIDC Authorization Code Flow with PKCE or a backend-for-frontend without rewriting every remote.

### Authorization and entitlements

- Define typed business entitlements such as:
  - `accounts.read`
  - `payments.create`
  - `payments.approve`
  - `trading.read`
  - `trading.execute`
- Prevent unauthorized routes from loading through route guards.
- Hide or disable navigation the user cannot access.
- Explain in comments and documentation that client guards improve UX and code-loading behavior but do **not** secure APIs. Every backend must validate tokens and permissions independently.

### Global experience

- Header and product identity.
- Global navigation.
- User menu.
- Selected customer/organization display.
- Notifications entry point.
- Responsive desktop and mobile layout.
- Skip links, focus handling, landmarks, and the shared accessibility baseline.
- Shared visual tokens and platform-level design language.

### Platform services

- Global error handling.
- Structured logging and telemetry interfaces.
- Correlation/trace ID management.
- Feature-flag contract.
- Remote registry and remote health/loading state.
- Platform notification contract.
- Cross-remote event contract for the small number of interactions that cannot be expressed cleanly as navigation.

### Route contracts

The Shell owns these stable public mount points:

```text
/                 Shell overview
/accounts/*       Accounts remote
/payments/*       Payments remote
/trading/*        Trading remote
/access-denied    Shell-owned authorization result
/unavailable      Shell-owned remote failure result
```

The internal routes beneath `/accounts`, `/payments`, and `/trading` will belong to their respective remote teams.

---

## What the Shell Explicitly Does NOT Own

The boundary matters more than the number of files. Do not implement any of the following business capabilities in this repository.

### Accounts domain

- Account summary calculations.
- Balances and available funds.
- Transaction history or filtering.
- Statements.
- Account-detail workflows.

The Shell may display clearly labeled **mock overview metrics** to make its landing page believable, but those values must be isolated behind a dashboard facade and documented as temporary composition data, not as the Accounts domain model.

### Payments domain

- ACH or wire-transfer workflows.
- Recipient management.
- Payment validation, limits, cutoffs, or fraud rules.
- Approval workflows.
- Payment history or status.

### Trading domain

- Market-data subscriptions.
- Bond search, pricing, bid/ask, yield, or spread calculations.
- Positions, order entry, execution, blotters, or risk calculations.

### Remote internals

- The Shell must not import a remote component, store, or service directly.
- The Shell must not know a remote’s internal folder structure.
- The Shell must not use another team’s private model types.
- The Shell must not coordinate routine remote releases.
- The Shell must not become a shared dumping ground for code that lacks a clear owner.

Design-system primitives and versioned platform contracts are shared libraries. They are **not** micro-frontends. Buttons, fields, modals, tables, typography, spacing tokens, and accessibility helpers belong in a design-system package, not in separately deployed remotes.

---

## Initial User Experience

Build a professional commercial-banking Shell called **Northstar Commercial Banking**.

The default simulated user context should be easy to find and replace:

```ts
{
  id: 'MG12345',
  displayName: 'Michael Gokey',
  initials: 'MG',
  customerId: 'C98342',
  organization: 'Gokey Farms LLC',
  entitlements: [
    'accounts.read',
    'payments.create',
    'payments.approve',
    'trading.read',
    'trading.execute'
  ]
}
```

The Shell overview should include:

- A greeting and selected organization.
- A small summary area with clearly marked simulated values.
- Three capability cards:
  - **Accounts** — “Understand my money.”
  - **Payments** — “Move money safely.”
  - **Trading** — “React to live markets.”
- Each card should display its stable route and required entitlement.
- An architecture note explaining what the Shell owns.

Selecting a capability should navigate to a local federation-boundary placeholder while that remote is not yet configured. The placeholder should explain:

- which route contract is working;
- which entitlement was checked;
- which exposed module will eventually load;
- that the page is intentionally local until the remote is built;
- that the remote can later deploy without rebuilding the Shell.

This is a teaching feature, not unfinished-product camouflage.

---

## Native Federation Host Strategy

Configure `@angular-architects/native-federation` for Angular 22 as a dynamic host using Angular’s esbuild-based application builder.

The Shell should initialize federation from:

```text
public/federation.manifest.json
```

The initial manifest must be valid but empty:

```json
{}
```

This is deliberate. The Shell must run before any remote exists and must remain useful if no remote is configured.

Document the future manifest shape without enabling unreachable URLs:

```json
{
  "accounts": "http://localhost:4201/remoteEntry.json",
  "payments": "http://localhost:4202/remoteEntry.json",
  "trading": "http://localhost:4203/remoteEntry.json"
}
```

Create a typed `RemoteRegistry` describing each capability:

```ts
type RemoteKey = 'accounts' | 'payments' | 'trading';

interface RemoteDefinition {
  key: RemoteKey;
  displayName: string;
  route: string;
  exposedModule: './Routes';
  requiredEntitlement: BankingEntitlement;
  objective: string;
}
```

The registry is Shell integration metadata. It must not contain remote implementation details.

### Future route conversion

Build the initial routes with local lazy placeholders, but document the exact conversion to a remote route:

```ts
import { loadRemoteModule } from '@angular-architects/native-federation';

{
  path: 'accounts',
  canMatch: [entitlementGuard('accounts.read')],
  loadChildren: () =>
    loadRemoteModule('accounts', './Routes').then(
      (remote) => remote.ACCOUNTS_ROUTES,
    ),
}
```

The route `/accounts` and entitlement `accounts.read` must remain stable when this conversion happens. Only the source of the route implementation should change.

Do not build fake remote applications inside the Shell repository. The future remotes must remain independently buildable and deployable projects.

---

## Shared Runtime Contract

Define a small, versionable platform contract that future remotes can consume without importing Shell internals.

The first iteration may live in a clearly separated `platform-contract` folder for teaching purposes, but document that a production platform would publish it as a versioned workspace/npm package such as:

```text
@northstar/platform-contract
```

The contract should expose abstractions or injection tokens for:

- readonly authentication/session context;
- current customer/organization context;
- entitlement checks;
- structured telemetry;
- correlation ID access;
- feature-flag evaluation;
- platform notifications;
- typed cross-remote events.

Do **not** expose the Shell store itself. Do not give remotes a general-purpose service locator. A narrow contract is safer and easier to version than sharing the host’s implementation.

### Prefer URLs for cross-domain handoffs

Use navigation as the default integration contract because URLs are observable, bookmarkable, testable, and framework-independent.

Example future handoff:

```text
/payments/wires?sourceAccount=84721
```

Accounts may request that navigation, but it must not call a Payments component or service.

Use typed platform events only when navigation is insufficient. Document ownership, versioning, required fields, optional fields, and backward-compatibility expectations for every shared event.

---

## Requirements

### Angular and TypeScript

- Angular 22.x.
- TypeScript version supported by that Angular release.
- Standalone components only; no NgModules.
- Zoneless change detection; no Zone.js dependency.
- `ChangeDetectionStrategy.OnPush` explicitly on presentation and routed components, even though Signals and zoneless Angular already reduce broad checks. This makes component intent visible to readers coming from earlier Angular versions.
- Signals for Shell-owned synchronous state.
- Keep writable Signals private and expose readonly Signals.
- Use `computed()` for derived state such as authentication status, unread notification count, authorized navigation, and remote availability.
- Use `effect()` only for genuine imperative side effects and explain why it is needed.
- Use RxJS only where event streams, cancellation, retries, or asynchronous composition make it the better tool. Do not recreate a `BehaviorSubject` store from older Angular habits.
- Modern template control flow: `@if`, `@for`, and `@switch`; no `*ngIf` or `*ngFor`.
- Lazy-load every routed feature.
- Strict TypeScript and strict Angular template checking.
- No `any` unless an external boundary makes it unavoidable and the reason is documented.

### State ownership

- `AuthStore` owns the mock user/session state.
- `ShellStore` owns global navigation, notifications, and other Shell UI state.
- `RemoteRegistry` owns static remote integration definitions.
- A remote-loading service owns runtime availability/loading/error state.
- Components consume readonly state and send commands back to the owning service.
- Do not create one giant global store containing future Accounts, Payments, and Trading data.

### Authentication and authorization seam

- Implement a `MockAuthSessionProvider` behind a contract that can later be replaced.
- Clearly mark the simulated identity as development-only.
- Model signed-in, loading, expired, and signed-out session states even if the first UI primarily demonstrates signed-in behavior.
- Add typed entitlement guards using `CanMatchFn` so unauthorized bundles are not loaded.
- Provide an accessible `/access-denied` page.
- Document the expected production identity flow and token-handling rules.
- Never log tokens or sensitive banking data.

### Routing

- Shell layout route with lazy child pages.
- Stable routes for overview, Accounts, Payments, Trading, access denied, not found, and remote unavailable.
- Route titles for browser and assistive-technology context.
- Query parameters for cross-domain navigation should be parsed and validated by the receiving domain, not trusted blindly.
- Preserve deep links on refresh once remotes are introduced.

### Resilience and remote failure

The Shell must not become a blank screen because one business capability is unavailable.

- The Shell should bootstrap with an empty manifest.
- A remote-load failure should be isolated to the affected route.
- Show a branded, accessible error state with:
  - the unavailable capability name;
  - a retry action;
  - a return-to-overview action;
  - a correlation/reference ID;
  - language that does not expose stack traces or internal URLs.
- Accounts being unavailable must not prevent Payments or Trading from loading.
- Log remote load start, success, failure, and duration as structured events.
- Do not implement infinite retry loops.

### Observability

Create a structured telemetry contract with a consistent event envelope:

```ts
interface PlatformTelemetryEvent {
  eventName: string;
  timestamp: string;
  correlationId: string;
  application: 'shell' | 'accounts' | 'payments' | 'trading';
  version: string;
  route?: string;
  durationMs?: number;
  outcome?: 'success' | 'failure';
  metadata?: Record<string, string | number | boolean>;
}
```

Implement a console-backed teaching adapter, not a vendor-specific production dependency. Explain where OpenTelemetry, Application Insights, CloudWatch RUM, or another enterprise platform could be connected later.

Never place account numbers, balances, payment details, order details, tokens, or personally identifiable information in telemetry metadata.

### Feature flags

- Define a typed feature-flag contract.
- Include flags for remote visibility, not remote authorization.
- A feature flag may hide an unfinished capability, but it must never grant permission.
- Entitlements and backend authorization remain authoritative.

### Design and accessibility

- Professional commercial-banking visual design, not a generic tutorial screen.
- Responsive desktop, tablet, and mobile layout.
- Semantic `header`, `nav`, `aside`, `main`, and appropriate section landmarks.
- A visible-on-focus skip link.
- Full keyboard access.
- Strong focus indicators.
- Active navigation communicated by more than color.
- Buttons and links must use native elements.
- Notifications and changing status must be announced appropriately without creating noisy live regions.
- Respect reduced-motion preferences.
- Meet WCAG 2.2 AA contrast and interaction expectations.
- Do not use emoji as the only meaning-bearing iconography.
- Keep global CSS small. Future remotes must not depend on accidental Shell selectors.
- Establish design tokens that can later move into `@northstar/design-system`.

### Security

- Treat all remote input, route parameters, runtime configuration, and server responses as untrusted boundaries.
- No secrets in source control.
- Runtime URLs are configuration, not hardcoded environment assumptions.
- Explain Content Security Policy considerations for remote origins.
- Explain how Subresource Integrity or signed deployment metadata could strengthen remote trust where the chosen federation approach supports it.
- Do not inject arbitrary remote URLs from query parameters or local storage.
- Sanitize error reporting.
- Document CSRF/XSS responsibilities and why a BFF can reduce browser token exposure.

### Testing

Use the Angular 22 testing toolchain and Vitest.

Include focused tests for:

- authentication/session derived state;
- entitlement decisions;
- authorized navigation computation;
- Shell notification state;
- feature flags;
- remote registry definitions;
- remote loading success and failure mapping;
- route guard allow and deny paths;
- fallback pages;
- responsive navigation commands;
- telemetry sanitization or metadata allowlisting.

Tests should verify behavior and contracts, not implementation trivia.

### Performance

- Keep the Shell small because every user pays its startup cost.
- Lazy-load all nonessential routed content.
- Do not bundle domain mock data into the initial chunk.
- Share Angular dependencies as strict singletons through Native Federation.
- Establish Angular bundle budgets.
- Document the measured production build sizes.
- Avoid adding a large component library merely to render the Shell.

---

## Suggested Project Structure

The exact filenames may evolve, but the ownership boundaries should remain visible:

```text
bank-shell/
├── public/
│   └── federation.manifest.json
├── src/
│   ├── app/
│   │   ├── features/
│   │   │   ├── access-denied/
│   │   │   ├── home/
│   │   │   ├── not-found/
│   │   │   ├── remote-placeholder/
│   │   │   └── remote-unavailable/
│   │   ├── platform/
│   │   │   ├── auth/
│   │   │   ├── federation/
│   │   │   ├── feature-flags/
│   │   │   ├── notifications/
│   │   │   ├── observability/
│   │   │   └── platform-contract/
│   │   ├── shell/
│   │   │   ├── shell-layout/
│   │   │   └── shell.store.ts
│   │   ├── app.config.ts
│   │   ├── app.routes.ts
│   │   └── app.ts
│   ├── bootstrap.ts
│   ├── main.ts
│   └── styles.scss
├── ARCHITECTURE.md
├── CONTRACT.md
├── DEVELOPMENT.md
├── federation.config.mjs
├── angular.json
├── package.json
└── README.md
```

Avoid generic folders such as `misc`, `common`, or `shared` unless the contents have a precise ownership definition. “Shared” is not an architecture boundary.

---

## Documentation Deliverables

Alongside the code, produce the following.

### 1. `README.md`

Include:

- the project mission;
- prerequisites;
- installation and local startup;
- test and production-build commands;
- a recommended code-reading order;
- a concise explanation of why the manifest starts empty;
- how later remotes will be connected;
- the measured build output;
- the next-step sequence: Accounts, Payments, then Trading.

### 2. `ARCHITECTURE.md`

Explain:

- why the Shell is a platform rather than a banking domain;
- what the Shell owns and refuses to own;
- standalone and zoneless Angular architecture;
- Signal state ownership;
- Native Federation startup and runtime discovery;
- independent deployment and the complexity tradeoff;
- authentication versus authorization;
- remote failure isolation;
- observability and correlation IDs;
- accessibility ownership;
- CSP and remote-origin trust;
- how the design-system and platform-contract packages should evolve.

Write this for an engineer or architect joining the project, not as marketing copy.

### 3. `CONTRACT.md`

Define:

- stable mount routes;
- remote logical names;
- required exposed module names;
- route export names expected from each remote;
- required entitlements;
- the session/customer context contract;
- telemetry envelope;
- feature-flag interface;
- platform notification interface;
- cross-remote navigation conventions;
- event versioning rules;
- compatibility and deprecation policy;
- what a remote may never import from the Shell.

Include a contract table similar to:

| Remote | Mount route | Logical name | Exposed module | Export | Entitlement |
|---|---|---|---|---|---|
| Accounts | `/accounts` | `accounts` | `./Routes` | `ACCOUNTS_ROUTES` | `accounts.read` |
| Payments | `/payments` | `payments` | `./Routes` | `PAYMENTS_ROUTES` | `payments.create` |
| Trading | `/trading` | `trading` | `./Routes` | `TRADING_ROUTES` | `trading.read` |

### 4. `DEVELOPMENT.md`

Include:

- standalone Shell startup;
- how to run with an empty manifest;
- how to connect one local remote at a time;
- expected local ports;
- deep-link testing;
- failure testing by stopping a remote;
- entitlement testing with alternate mock users;
- production build and bundle inspection;
- common Native Federation troubleshooting steps;
- Git workflow suitable for the `bank-shell` repository.

---

## Definition of Done

- [ ] Angular 22 standalone application created.
- [ ] Zoneless change detection enabled.
- [ ] `OnPush` used consistently.
- [ ] Native Federation configured as a dynamic host.
- [ ] Empty runtime federation manifest supported.
- [ ] Shell runs with no remotes online.
- [ ] Signal-based session and Shell state established.
- [ ] Typed user/customer context established.
- [ ] Typed entitlement model and route guards implemented.
- [ ] Global header and responsive navigation implemented.
- [ ] Accessible overview page implemented.
- [ ] Accounts, Payments, and Trading placeholder boundaries implemented.
- [ ] Access-denied page implemented.
- [ ] Remote-unavailable page and finite retry behavior implemented.
- [ ] Not-found page implemented.
- [ ] Remote registry established.
- [ ] Remote load timing and failures logged through structured telemetry.
- [ ] Correlation ID established.
- [ ] Feature-flag contract established.
- [ ] Platform notification contract established.
- [ ] Cross-remote navigation conventions documented.
- [ ] Shared runtime contract separated from Shell internals.
- [ ] Global error handler implemented.
- [ ] Shell CSS does not leak assumptions into remotes.
- [ ] Keyboard navigation and skip link verified.
- [ ] WCAG 2.2 AA expectations documented and checked.
- [ ] Unit tests pass.
- [ ] Production build passes.
- [ ] Bundle budgets pass.
- [ ] `README.md`, `ARCHITECTURE.md`, `CONTRACT.md`, and `DEVELOPMENT.md` completed.
- [ ] The repository contains no Accounts, Payments, or Trading business implementation.

---

## Output Format

- Provide the complete proposed file tree first.
- Then provide the implementation file by file.
- Include substantial inline developer comments at architectural decision points.
- Do not fill simple code with repetitive comments that merely translate syntax into English.
- After the code, provide `README.md`, `ARCHITECTURE.md`, `CONTRACT.md`, and `DEVELOPMENT.md`.
- Include tests as part of the implementation, not as a future recommendation.
- End with:
  1. exact installation and startup commands;
  2. exact test and production-build commands;
  3. how to run the Shell with the empty manifest;
  4. how Accounts will later be connected on port 4201;
  5. a short list of deliberate architectural decisions and tradeoffs;
  6. a checklist of anything requiring human configuration.

Do not silently skip the runtime contract, remote-failure handling, authorization distinction, accessibility baseline, documentation, or tests to save time. Those are central to the exercise, not optional polish.

Most importantly, do not “finish” the platform by moving future remote business logic into the Shell. A clean boundary today is what allows Accounts, Payments, and Trading to become genuinely independent tomorrow.
