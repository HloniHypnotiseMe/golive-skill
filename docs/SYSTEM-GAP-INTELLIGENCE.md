# System Gap Intelligence

## Purpose

Every build must continuously ask:

> **What does this system claim, depend on, assume, expose, trigger, receive, store, charge, deliver, and fail to do yet?**

The objective is not to generate more work for its own sake. It is to prevent a feature from being considered complete when only its visible code exists.

This is a **build-time and go-live intelligence layer**. It should run before implementation, after implementation, before integration, before release, and after deployment.

## The claim-to-proof rule

For every user-visible or operational claim, identify:

1. **Claim** — exactly what the product says it can do.
2. **Capability** — the code/service that performs it.
3. **Authority** — the system that is the source of truth.
4. **Dependencies** — services, credentials, DNS, data, queues, webhooks, storage, permissions and human steps.
5. **Execution path** — the complete request/event lifecycle.
6. **Evidence** — what proves the outcome actually happened.
7. **Failure path** — what happens when a dependency fails, times out, duplicates, is unavailable or returns an unexpected result.
8. **Recovery** — how the system resumes without corrupting state or duplicating consequential actions.
9. **Security boundary** — what must remain server-side and what the browser/agent may see.
10. **Operational owner** — which service, department or agent is responsible for the next action.
11. **Observability** — logs, audit events, metrics, alerts and correlation/reference IDs required to investigate it.
12. **Lifecycle state** — EXISTS, WORKS, VERIFIED or LIVE, with evidence appropriate to that state.

A claim without a complete path and evidence is **not complete**.

## Mandatory gap sweep

At every build/release checkpoint, inspect these dimensions:

### Product
- What does the UI/site promise?
- Are products, services, pricing, eligibility and fulfilment paths actually connected?
- Are all buttons/forms/actions connected to real behaviour?
- Are success states based on authoritative evidence rather than optimistic UI state?

### Money
- Where does money enter?
- Who creates the payment instruction?
- Who is the payment authority?
- How is provider status reconciled?
- Are webhooks/callbacks authenticated and idempotent?
- Where is the transaction ledger?
- What creates the receipt/invoice/entitlement?
- What happens on duplicate, failed, reversed or delayed payment?
- Are prices, currencies, fees and taxes explicit?
- Is any claim being made about escrow, payout or settlement without evidence?

### Data
- What is the source of truth?
- Is there a legacy database still reachable?
- Are writes and reads using the same authoritative store?
- Is tenant isolation enforced server-side?
- Are migrations reversible/recoverable?
- Are backups and restores tested?
- What data is immutable and when?
- Can retries create duplicate records?

### Integrations
- Is every external integration routed through the intended boundary?
- Are provider APIs actually documented/verified rather than inferred?
- Are credentials server-side?
- Are webhooks registered, authenticated, persisted and reconciled?
- What happens when the provider is down?
- Is there a timeout/retry/idempotency strategy?
- Is provider evidence distinguishable from application state?

### Infrastructure
- Can the complete stack actually start?
- Are DNS and TLS correct?
- Are reverse-proxy routes correct?
- Are ports, networks, volumes and health checks correct?
- Do services survive restart?
- Are secrets present by name and in the correct runtime scope?
- Are backups, monitoring and recovery procedures defined?
- Is the deployment path consistent with the chosen production architecture?

### Communications
- Can the system send?
- Can it receive?
- Are SPF/DKIM/DMARC/MX correct where applicable?
- Does the intended mailbox actually receive?
- Are internal notifications connected to business events?
- Are CRM/n8n/email events correlated and auditable?

### Agents and automation
- Which agent/department owns the action?
- Is the agent permitted to execute it?
- Which actions require human approval?
- Can an agent change source-of-truth financial/legal/security state?
- Is there a kill switch/human override where consequential automation exists?
- Are agent actions auditable?
- Are agent claims evidence-bound?

### Security
- Can secrets enter browser bundles, logs, URLs, reports or prompts?
- Can an unauthenticated caller perform a consequential action?
- Is authorization checked server-side?
- Is tenant isolation tested with multiple identities?
- Are webhook signatures verified?
- Are replay/duplicate requests safe?
- Are sensitive KYC/payment values excluded from logs?

### Reliability
- What if the request succeeds but the response is lost?
- What if a webhook arrives twice?
- What if events arrive out of order?
- What if a worker dies halfway through?
- What if a database transaction commits but downstream delivery fails?
- Can the system reconcile itself?
- Can an operator identify and resume the exact failed step?

### Compliance / governance
- Which actions require evidence?
- Which claims require human review?
- What audit record is retained?
- What is the retention/deletion rule?
- Are legal conclusions being inferred from application state?
- Are provider registrations/licences/certifications being claimed without external evidence?

### Customer journey
- Can a stranger discover the offer?
- Can they understand price and terms?
- Can they submit/register?
- Can they pay?
- Can the system prove payment?
- Can fulfilment start automatically where safe?
- Can the customer see the correct status?
- Can support find the entire record?
- Can the service be completed and evidenced?

### Operations
- Who gets alerted?
- What is the next action?
- What happens outside business hours?
- What happens when the owner is unavailable?
- Is there a runbook?
- Is there a handoff?
- Can the system detect drift after release?

## The hidden-gap questions

Before declaring a feature complete, explicitly ask:

- **What did we assume exists but never verified?**
- **What exists in code but is not connected at runtime?**
- **What works in a happy-path test but has no failure/recovery path?**
- **What can the UI claim that the backend cannot prove?**
- **What can an agent say that the evidence does not support?**
- **What still uses a legacy path?**
- **What integration is implemented on the wrong side of a security boundary?**
- **What happens if the external provider says something different from our local state?**
- **What happens after restart?**
- **What happens after duplicate delivery?**
- **What happens when a dependency is unavailable?**
- **What happens to money/data when the process stops halfway?**
- **What customer-facing action has no real fulfilment path?**
- **What production secret/configuration has only been named, not exercised?**
- **What manual step is being mentally counted as complete?**
- **What evidence would prove this claim to a skeptical third party?**
- **What have we not tested because it was inconvenient?**
- **What can break without anyone being notified?**
- **What changed outside the repository that can create drift?**

## Build loop

The intelligence layer follows:

**DETECT → MAP → BUILD → CONNECT → TEST → VERIFY → OPERATE → RECHECK**

At each transition, produce a gap list.

A gap is not automatically a blocker. Classify it:

- **CRITICAL** — unsafe, misleading, financially consequential, security-breaking, data-loss risk, or prevents the claimed core journey.
- **HIGH** — required for production operation or a material customer journey.
- **MEDIUM** — important operational/reliability/quality gap.
- **LOW** — enhancement that does not invalidate the current claim.
- **HUMAN** — requires an owner action outside the agent's authority.
- **UNKNOWN** — insufficient evidence; never silently treat as complete.

## Claim gate

Use this matrix:

| State | Meaning | Minimum evidence |
|---|---|---|
| EXISTS | code/config/schema is present | repository evidence |
| WORKS | it executes successfully | repeatable execution/test |
| VERIFIED | the intended outcome is demonstrated | outcome evidence + relevant integration checks |
| LIVE | production deployment and runtime behaviour are demonstrated | production runtime evidence + end-to-end evidence |

**UNKNOWN, SKIPPED and HUMAN are not PASS.**

## Evidence chain

For consequential journeys, record:

**intent → request → authorization → execution → provider/source-of-truth result → persistence → downstream event → customer/owner notification → audit evidence**

If any link is missing, surface the missing link rather than filling it with an assumption.

## Regression intelligence

The sweep must also compare the current build against:

- the approved architecture,
- prior release claims,
- environment contracts,
- provider boundaries,
- database migrations,
- security rules,
- known blockers,
- previous verification evidence.

A new implementation must not silently reintroduce a retired provider, legacy runtime, insecure browser credential, duplicate integration, or contradicted business rule.

## Definition of done

A build is not “done” because tests pass.

It is done for a claim only when:

1. the capability exists,
2. the intended runtime path is connected,
3. the authority is identified,
4. consequential state changes are protected,
5. failure/retry/idempotency behaviour is defined,
6. evidence is captured,
7. the customer/owner journey works,
8. monitoring/audit exists where needed,
9. legacy/conflicting paths are removed or explicitly isolated,
10. the claim is labelled with the correct EXISTS/WORKS/VERIFIED/LIVE state.

## What the agent should report

Every build checkpoint should end with:

**BUILT**
- what changed

**PROVEN**
- what evidence was actually obtained

**MISSING**
- known gaps

**OVERLOOKED / NEWLY DISCOVERED**
- gaps found by the sweep

**BLOCKED**
- what cannot proceed and why

**NEXT WIN**
- the smallest concrete action that closes the highest-value remaining gap

This makes gap detection part of the build itself rather than a final audit performed after the system has already accumulated hidden assumptions.
