# Backlog

## What's next

**Complete publishing setup and maintain.** npm already serves v0.1.5. The old GitHub Actions
publish attempt failed because no `NPM_TOKEN` was configured. The replacement workflow uses npm
trusted publishing; configure the owner-side trust entry described in `docs/RELEASING.md` before
pushing the next version tag, then verify its first real publish. A green test run does not prove
that npm trust is configured.

Review the open Dependabot pull requests separately, especially major TypeScript, Jest and Medusa
updates. Do not batch-merge them as repository housekeeping. Community announcements and a Medusa
listing remain optional follow-ups requiring explicit authorization.

## Dependency security review

The 2026-10-07 `npm audit` of the current lockfile reports 175 affected package entries
(46 moderate, 127 high, 2 critical). This repository installs Medusa as development dependencies
and declares it as a peer dependency for consumers; audit counts alone do not prove runtime
exposure in a consuming store. Review that store's actual lockfile separately.

- Critical `@mikro-orm/core`: advisory GHSA-gwhv-j974-6fxm (SQL injection), with related
  GHSA-qpfv-44f3-qqx6 (prototype pollution). npm proposes updating the Medusa development packages
  to 2.21.2. Assess the existing Medusa dependency PR, compatible framework versions, and plugin
  lifecycle tests before updating `package.json` and `package-lock.json`.
- Critical `proxy-addr`: GHSA-jqcg-44mw-7w3h (IP spoofing); affected range >=1.1.0 <2.0.8.
  Review the transitive update and trusted-proxy configuration where this is used.
- Re-run the full audit after remediation. Do not apply `npm audit fix --force` blindly or treat
  passing plugin unit tests as evidence that these dependency advisories are resolved.

## Hardening (LOW, non-blocking, from the verification loop)

All of these were confirmed non-exploitable by the reviewers who raised them; they are
polish, not fixes.

- OAuth token cache has no mutex: two concurrent calls can both fetch a token. Harmless
  (last write wins, both tokens valid) but a single-flight guard would be tidier
  (`src/providers/peach/lib/peach-client.ts`, getToken).
- `safeEq` reveals signature length mismatch by timing (standard practice; HMAC output
  length is public anyway) (`lib/verify-webhook.ts`).
- Malformed-input test matrices for `formatPeachAmount`/`amountsMatch` could be broader
  (NaN, Infinity, objects, exotic coercibles) (`lib/amount.ts` specs).
- `lastRefund.result` stores the raw Peach result object in payment data; consider
  trimming to code + description (`service.ts`, refundPayment).
- OAuth failure error message includes the HTTP status but not the Peach error body;
  helpful detail is logged only at debug level (`lib/peach-client.ts`).
- Declarations tsc pass emits no source maps; fine for a types-only artifact, note if
  debugging support is ever requested (`tsconfig.declarations.json`).
- 3DS challenge path was not exercised in sandbox E2E (the sandbox entity runs in
  Integrator Test Mode and auto-approves). Exercise it if Peach provides a
  challenge-enabled sandbox entity.

## Docs

- Done: README, `docs/WEBHOOKS.md`, and `examples/storefront/` (see CHANGELOG 2026-07-07).
- Possible follow-up: a heavily genericized Apple/Google Pay (Embedded Express) storefront
  example was skipped because the production reference is store-specific; revisit on demand.

## Decisions

- Multi-currency positioning: resolved as "document the limitation" (see the README's Limitations
  section) rather than add per-currency-decimals support. The amount handling works at cent
  granularity for 2-decimal currencies only (`amountsMatch` compares rounded cents,
  `formatPeachAmount` forces 2 dp); zero-decimal (JPY) and three-decimal currencies are
  unsupported. Revisit if a user actually needs one of those currencies.
- Publish identity: personal GitHub + npm, unofficial community plugin, plain-text
  "not affiliated with or endorsed by Peach Payments" disclaimer, no Peach branding.
