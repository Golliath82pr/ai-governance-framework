# Circuit Breaker Examples by Domain

Real-world circuit breakers across common domains. Use these as starting points — your best CBs will come from your own mistakes.

## Web Development

| # | TRIGGER | STOP | USE INSTEAD |
|---|---|---|---|
| CB-1 | `"use client"` on a component that fetches data | Disables SSR, kills SEO, causes waterfall | Keep as Server Component. Extract interactive parts into client child components. |
| CB-2 | Client-side `disabled` attribute as security enforcement | Bypassed by API calls, browser devtools, automated tools | Server-side validation. Client-side is UX hint only. |
| CB-3 | `float` / `number` for monetary display | Rounding errors: 0.1 + 0.2 != 0.3 | `Intl.NumberFormat` for display, integer cents for storage, `Decimal` for calculation. |
| CB-4 | API endpoint without rate limiting | Burst traffic triggers bans, costs spiral, DDoS vector | Rate limit at API gateway or middleware. Per-IP and per-user limits. |
| CB-5 | Sensitive data in URL parameters | Logged in server logs, browser history, referrer headers | POST body or encrypted session storage. |

## Python / Data Science

| # | TRIGGER | STOP | USE INSTEAD |
|---|---|---|---|
| CB-1 | `float` for monetary calculation | Drift accumulates: $0.10 + $0.20 != $0.30 | `from decimal import Decimal` with explicit precision. |
| CB-2 | Random train/test split on time-series data | Future data leaks into training set | Walk-forward validation or expanding window. Never random split on temporal data. |
| CB-3 | Feature that uses target-derived information | Model learns the answer, not the pattern (leakage) | Trace every feature to raw source. If data path crosses the target variable, remove. |
| CB-4 | `pip install` / upgrade on production data tools | Migration may be destructive (e.g., database schema changes) | Pin versions. Test upgrades on disposable copy first. |
| CB-5 | Model validated on same distribution as training | Overfitting invisible in metrics | Leave-group-out or walk-forward. Different distribution for validation. |

## Financial Modeling

| # | TRIGGER | STOP | USE INSTEAD |
|---|---|---|---|
| CB-1 | IRR/NPV presented without scenario qualification | Single-scenario misleads investment decisions | Always: Base + Optimistic + Conservative. Label assumptions. |
| CB-2 | Discount rate without sourcing | WACC varies wildly by context | Source: comparable transactions, CAPM derivation, or explicitly state assumption. |
| CB-3 | Revenue projection without capacity constraint | Unconstrained growth is fiction | Model capacity limits: headcount, infrastructure, market size ceiling. |
| CB-4 | 20-year projection table in portrait layout | Clips in PDF, auto-sizes unpredictably, unreadable | Landscape section break, or split into 2x10-year tables. |
| CB-5 | Negative values without formatting convention | Ambiguous: is (100) negative or a note reference? | Parentheses for negatives: (100.00). Consistent through entire document. Currency with M/B suffixes. |

## DevOps / Infrastructure

| # | TRIGGER | STOP | USE INSTEAD |
|---|---|---|---|
| CB-1 | Governance/quota budget tested on dev data volume | Passes QA, fails production at 10-100x volume | Budget formula: `(limit / production_ceiling) * 0.7` safety margin. |
| CB-2 | Database migration without rollback plan | Forward-only migrations become production incidents | Write up + down migrations. Test rollback on staging before production. |
| CB-3 | Secret in environment variable without rotation plan | Leaked secrets have infinite exposure window | Secret manager (Vault, AWS SM) with rotation. Expire old secrets. |
| CB-4 | Search/query without result limit | Returns unbounded results, memory blow, governance exhaustion | Always paginate. Set explicit page size. Use streaming callbacks for large result sets. |
| CB-5 | `rm -rf` or `DROP TABLE` in automated script | No confirmation, no backup check, irreversible | Soft delete first. Backup verification before destructive ops. Manual confirmation gate. |

## ERP / Business Systems

| # | TRIGGER | STOP | USE INSTEAD |
|---|---|---|---|
| CB-1 | Submitting the triggering record inside its own event handler | Re-entrancy cascade — infinite loop of saves | Use built-in field-setting mechanisms. Cross-record submits are safe. |
| CB-2 | Lock Record as access control | Blocks ALL edits including system processes, CSV imports, API calls | Field-level display rules + server-side validation on save. |
| CB-3 | Custom record type with wrong prefix | Runtime crash — platform expects specific prefix format | Check documentation for record type vs. field prefixes. They're different. |

## How to Write Good Circuit Breakers

1. **Start from pain.** Each CB should trace back to a real mistake or near-miss.
2. **Be concrete.** "Don't use floats for money" is actionable. "Be careful with numbers" is not.
3. **Include the why.** The failure mode helps the AI catch variants.
4. **Keep the alternative specific.** "Use Decimal" not "use a better approach."
5. **Prune ruthlessly.** Max 15 active. If a CB hasn't fired in 6 months, retire it.
