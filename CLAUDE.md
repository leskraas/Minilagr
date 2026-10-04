# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project status

No code yet: the repo holds only docs. The product spec lives in the Claude Docs document "Minilagr – produktbeskrivelse" (https://claude.ai/code/artifact/ebcefce2-954e-415a-b0d3-55b53a172a53). Treat it as the source of truth for scope. Domain terms are defined in `GLOSSARY.md`.

When the stack is in place, add the commands to build, lint and test, including how to run a single test.

## Product

Minilagr is a multi-tenant SaaS for running self-storage facilities with no staff on site. An end customer finds a free unit, signs the lease with BankID, pays by card or Vipps and gets an access code on their phone. Operators see occupancy, revenue and outstanding payments in one panel. Access is blocked automatically when payment fails.

It is built for Norway from day one: BankID signing, Vipps, Norwegian lease templates and Norwegian VAT rules for rentals. Our own storage facility is the first customer and the pilot. UI text is Norwegian.

## Architecture (planned)

- **One Next.js app, three surfaces:** the customer site (branded per operator, on its own subdomain or custom domain), the operator panel (must work well on mobile) and platform admin.
- **UI:** shadcn/ui with Base UI as the underlying primitives, and a custom visual style (not any existing brand). Operator branding will replace colours and logo later, so keep them as tokens.
- **Tenant isolation lives in the database.** Every table carries `operator_id`, and Postgres row-level security enforces access. Never rely on application code alone to keep operators apart.
- **Core tables:** `operators`, `locations`, `units`, `customers`, `leases`, `payments`, `access_codes`.
- **Lock adapter layer:** every lock vendor sits behind one interface with three operations: create code, change code, block code. Adding a vendor must not touch the rest of the system. The first vendor is chosen with the pilot facility.
- **MVP integrations:** BankID (Criipto or Signicat), Stripe Connect (first payment, monthly charges, payouts to operators), Vipps ePayment and recurring payments, SMS, transactional e-mail.

## Constraints

- Operators are data controllers for their customers; Minilagr is the data processor.
- Database and files are stored in the EU/EEA.
- Access codes must never appear in plain text in logs. Sensitive fields are encrypted.
- Operators use two-factor auth and role-based access.

## MVP scope

The MVP is done when the pilot facility can rent out a unit from booking to access with no manual help. It covers: locations and units, available units online, BankID signing, card and Vipps, monthly charges, access codes via one lock vendor, a customer page with cancellation, a simple dashboard, and dunning with access blocking. White label for external operators, more lock vendors, waitlist, unit swaps, discount codes, accounting export and staff roles come in version 2. Defer anything outside the MVP.

## Agent skills

### Issue tracker

Issues and specs are tracked in GitHub Issues for leskraas/Minilagr. See `docs/agents/issue-tracker.md`.

### Domain docs

Single-context: one `GLOSSARY.md` and `docs/adr/` at the repo root. See `docs/agents/domain.md`.
