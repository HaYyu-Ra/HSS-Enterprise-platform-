# Hayyu Software Solution Enterprise Platform

## Repository baseline

At the initial review, GitHub reported a public repository of size zero, with main configured as the default branch and no branches returned. Source, migrations, APIs, permissions, tests, and deployment configuration could not be reviewed. The paths below are proposed. If the deployed Dala app is the canonical implementation, inspect its source export and migrations before scaffolding a replacement. Identify the deployed version and keep credentials and production data out of this public repository.

## Delivery order

| Priority | Proposed paths | Scope and acceptance criteria |
| --- | --- | --- |
| P0 | README.md, .env.example, .github/workflows/ci.yml, db/migrations/ | Establish the canonical source, document local setup and required environment variable names, and make CI build the app and apply migrations on a fresh PostgreSQL database. |
| P0 | auth/, authorization/, db/migrations/ | Model users, organizations, memberships, roles, permissions, and audit events. Check role and record scope server-side for each protected action. Apply database policies where clients access the database directly. Require privileged staff MFA and test revocation and privilege escalation. |
| P1 | routes/public/, routes/portal/, routes/staff/ | Publish services, approved testimonials, and inquiry forms. Authenticate access to customer projects, quotes, invoices, and tickets. Keep staff entry points out of public navigation and enforce staff access on pages and APIs. Explicitly allowlist public response fields. |
| P1 | modules/crm/, modules/consultations/, modules/quotes/, modules/projects/ | Deliver inquiry → lead → consultation → quote → acceptance → project. Store quote revisions and line items. Record the accepted revision and create or link exactly one project under retries or concurrent acceptance. Audit transitions. |
| P2 | modules/finance/, modules/support/ | Implement invoices, payments, allocations, adjustments, support agreements, tickets, and SLA events. Use fixed-precision amounts and configurable tax rules subject to finance approval before launch. Preserve issued invoice history, audit adjustments, and prevent duplicate payment posting. |
| P2 | .github/workflows/, deploy/, docs/runbooks/ | Separate staging and production credentials and databases. Require CI, staging smoke tests, migration rehearsal, tested database restore, and a rollback or forward-fix procedure. Verify logs exclude secrets and sensitive customer data. |

## Role boundaries

| Role | Scope |
| --- | --- |
| Super Admin | Manage staff identities, privileged roles, platform settings, and security audit access. Audit privilege changes and protect the last active Super Admin account. |
| Admin | Manage leads, consultations, customer records, service content, and assignments. Reserve privileged role grants and finance approvals for explicitly authorized roles. |
| Finance | Issue invoices, record and reconcile payments, and process authorized adjustments. Separate preparation from approval for sensitive actions. |
| Project Manager | Manage assigned projects, milestones, tasks, and delivery updates. Scope customer and quote access to assigned work. |
| Support | Work assigned tickets, record SLA events, and access customer details needed for support. |
| Client | View records belonging to their organization, accept authorized quote revisions, and submit tickets. Validate organization membership on every request. |

## API and release gates

- Start with `POST /api/public/inquiries`. Validate input, rate-limit submissions, and return a receipt without exposing CRM records. Store any uploads privately and authorize downloads.
- Separate `/api/portal/*` and `/api/staff/*`. Validate request schemas, enforce record-level access, and audit sensitive mutations. Use transactions and uniqueness constraints for quote acceptance, invoice issuance, and payment posting.
- Test role permissions and workflow transitions, migrations and authorization against PostgreSQL, and the end-to-end flow from public inquiry through quote acceptance, project creation, and client invoice visibility. Test direct protected API calls, cross-customer ID substitution, role escalation, and revoked sessions.
- Record the reviewed commit SHA, migration versions, staging smoke-test results, and restore result before production deployment.
