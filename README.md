# n8n-nodes-erpnext-selling

[![Live tested](https://img.shields.io/badge/live--tested-ERPNext%20v16%20%2F%20n8n%20self--hosted%20%2F%20LXD%20%2F%20API%20v2-00A6A6)](#live-tested-status)
[![Runtime audit](https://img.shields.io/badge/runtime%20audit-0%20vulnerabilities-2E7D5F)](#development)
[![Node checks](https://img.shields.io/badge/n8n%20node%20lint%20%2B%20build-passing-2E7D5F)](#development)
[![Package](https://img.shields.io/badge/package-0.1.1-2490EF)](./package.json)

Community n8n node package for ERPNext/Frappe Selling v15-v16.

This package is part of the `n8n2erpnext` ecosystem. It focuses on customer-facing sales workflows and keeps generic escape hatches for custom DocTypes and whitelisted Frappe methods.

## Live-Tested Status

This package has been live-tested end to end on the project ERPNext/Frappe test environment:

| Area | Status |
| --- | --- |
| ERPNext/Frappe target | Live-tested on ERPNext v16/Frappe v16 behavior |
| n8n runtime | Live-tested on self-hosted n8n `2.20.7-exp.0` |
| Infrastructure | Live-tested through LXD ERPNext container at `http://10.192.135.2:8001` with host header `erp.thaiduy.digital` |
| API coverage | Live-tested with Frappe API v1 read workflow and API v2 document workflows |
| Core selling lifecycle | Customer -> Lead -> Opportunity -> Quotation submit -> Sales Order submit |
| Adjacent document coverage | Sales Invoice submit and Payment Entry submit through `Custom DocType` |
| Amendment coverage | Sales Order cancel/amend/submit flow |
| Negative coverage | Duplicate Item, missing Customer, delete submitted Sales Order, fake Sales Order lookup |
| Security response policy | Public webhook responses were allowlisted summaries; `securityFindings: []` |
| Cleanup | Temporary workflows were deactivated and verified as `404 Active version not found` |

The live verification used traceable demo records in the ERPNext LXD test instance. The README intentionally includes test infrastructure routing values and document IDs, but no API keys, API secrets, Authorization headers, database passwords, npm tokens, or credential material.

## Who This Is For

This package is built for teams that run ERPNext Selling and want controlled sales automations in n8n.

Typical users:

- ERP administrators who maintain ERPNext/Frappe.
- Sales, customer success, finance, or operations teams that need lead, opportunity, quotation, or sales order workflows.
- Integration teams that want repeatable n8n automations without writing custom Frappe client code for every selling process.

The node is intentionally conservative: it exposes standard Selling document operations, supports Frappe API v1 and v2, and allows controlled fallback access to custom DocTypes and whitelisted Frappe methods.

## Architecture At A Glance

Read workflow from left to right:

```text
ERPNext / Frappe Selling  <---- API token ---->  n8n ERPNext Selling node  <---- webhook/API ---->  Client / App / Report
```

Common read pattern:

```text
Client
  -> n8n Webhook
  -> ERPNext Selling node
  -> Frappe REST API
  -> ERPNext Selling DocType
  -> filtered JSON response
```

Common selling lifecycle pattern:

```text
n8n Webhook / Schedule / App Event
  -> validation / mapping / approval logic
  -> ERPNext Selling node
  -> Customer, Lead, Opportunity, Quotation, or Sales Order
  -> safe summary response or downstream system
```

## Supported Resources

- Customer
- Lead
- Opportunity
- Quotation
- Sales Order
- Sales Partner
- Sales Person
- Item
- Contact
- Address
- Custom DocType
- Frappe Method

`Custom DocType` is used when a selling workflow crosses into another ERPNext module, such as `Delivery Note`, `Sales Invoice`, or `Payment Entry`.

## Node Identity

All `n8n2erpnext` module nodes use the same ERPNext-style logo shape. Each module changes only the main background color.

| Module | Color | Hex | Reason |
| --- | --- | --- | --- |
| Core | ERPNext blue | `#2490EF` | Foundation package, closest to the ERPNext brand color. |
| HRMS | People green | `#2E7D5F` | Human operations, employees, attendance, leave, payroll. |
| Accounting | Finance orange-red | `#D94A2B` | Ledger, journals, invoices, financial control. |
| Buying | Procurement amber | `#C47F00` | Purchase flow, suppliers, RFQs, purchase orders, receipts, spend. |
| Selling | Commerce teal | `#00A6A6` | Customer-facing pipeline, quotations, sales orders, revenue. |
| Stock | Frappe black | `#171717` | Warehouses, items, inventory movement; aligned with Frappe black. |

## Operations

For Selling doctypes:

- Create
- Get
- Get Many
- Update
- Delete
- Submit
- Cancel

For Frappe methods:

- Run Method

## API Versions

The node supports both ERPNext/Frappe document API styles:

- `v1`: `/api/resource/:doctype`
- `v2`: `/api/v2/document/:doctype`

Use `v1` for broad compatibility. Use `v2` when your ERPNext/Frappe v16 environment is ready for the newer document API behavior.

Submit and cancel use the shared `n8n2erpnext` helper rule:

- Submit fetches the latest document and sends `{ doc }` to `frappe.client.submit`.
- Cancel fetches the latest document and sends `{ doctype, name }` to `frappe.client.cancel`.

Reference:

- [Frappe REST API](https://docs.frappe.io/framework/user/en/api/rest)

## Credentials

Create an API key and secret in ERPNext/Frappe, then configure:

- Site URL: `https://erp.example.com`
- Site Host Header, optional: `erp.example.com`
- API Key
- API Secret
- Ignore SSL Issues, optional

The node authenticates with:

```http
Authorization: token api_key:api_secret
```

Credential fields are marked as password fields where appropriate. Do not expose API keys, API secrets, Authorization headers, tokens, or passwords in webhook responses, logs, README examples, or package artifacts.

### Internal URL With Public Host Header

When n8n and ERPNext run on the same VPS, you can point n8n at the internal ERPNext address and still send the public ERPNext host header:

- Site URL: `http://erpnext.internal:8001`
- Site Host Header: `erp.example.com`

For the current VPS/LXD test setup:

```text
Site URL: http://10.192.135.2:8001
Site Host Header: erp.thaiduy.digital
```

This is infrastructure routing information for the project test environment, not credential material. API keys and API secrets are not included in this README.

## Examples

Get recent customers:

```json
{
  "resource": "customer",
  "operation": "getMany",
  "apiVersion": "v1",
  "fields": "name,customer_name,customer_group,territory,disabled",
  "filtersJson": "[]",
  "returnAll": false,
  "limit": 20,
  "orderBy": "modified desc"
}
```

Get submitted sales orders:

```json
{
  "resource": "salesOrder",
  "operation": "getMany",
  "apiVersion": "v2",
  "fields": "name,customer,transaction_date,grand_total,status,per_delivered,per_billed",
  "filtersJson": "[[\"docstatus\",\"=\",1]]",
  "returnAll": false,
  "limit": 20,
  "orderBy": "transaction_date desc"
}
```

Run a whitelisted Frappe method:

```json
{
  "resource": "frappeMethod",
  "operation": "runMethod",
  "methodName": "frappe.client.get_value",
  "argumentsJson": {
    "doctype": "Sales Order",
    "filters": { "name": "SAL-ORD-2026-00001" },
    "fieldname": ["name", "customer", "grand_total", "status"]
  }
}
```

Use `Custom DocType` for selling-adjacent documents:

```json
{
  "resource": "customDocType",
  "customDocType": "Sales Invoice",
  "operation": "get",
  "documentName": "ACC-SINV-2026-00001",
  "apiVersion": "v2"
}
```

## Webhook From n8n To ERPNext Selling

Use this pattern when you want an HTTP endpoint in n8n that reads or writes Selling data in ERPNext.

```text
Client / Browser / App
  -> n8n webhook URL
  -> ERPNext Selling node
  -> Frappe REST API
  -> ERPNext Selling DocType
  -> JSON response or safe summary
```

### 1. Configure The ERPNext Credential

In n8n, create or edit an `ERPNext API` credential:

- Site URL: `http://erpnext.internal:8001`
- Site Host Header: `erp.example.com`
- API Key: your ERPNext API key
- API Secret: your ERPNext API secret
- Ignore SSL Issues: `false`

For the current VPS/LXD test setup:

```text
Site URL: http://10.192.135.2:8001
Site Host Header: erp.thaiduy.digital
```

### 2. Create A Read Workflow

Create a workflow with these nodes:

```text
GET Webhook -> ERPNext Selling
```

Webhook node:

- HTTP Method: `GET`
- Path: `erpnext-selling-get-customers`
- Respond: `When Last Node Finishes`
- Response Data: `All Entries`

ERPNext Selling node:

- Credential: your `ERPNext API` credential
- Resource: `Customer`
- Operation: `Get Many`
- API Version: `v1`
- Fields: `name,customer_name,customer_group,territory,disabled`
- Filters JSON: `[]`
- Return All: `false`
- Limit: `20`
- Order By: `modified desc`

### 3. Activate And Test

Activate the workflow, then call:

```bash
curl -i https://n8n.example.com/webhook/erpnext-selling-get-customers
```

On the local VPS, you can test without going through the public proxy:

```bash
curl -i http://127.0.0.1:5678/webhook/erpnext-selling-get-customers
```

The tested workflow artifact is included in this repository:

```text
n8n-webhook-erpnext-selling-get-customers.workflow.json
```

## Webhook From ERPNext v16 To n8n

Use this pattern when ERPNext should call n8n automatically after a Selling document is created or updated. For example, ERPNext can call a n8n workflow whenever a `Lead`, `Opportunity`, `Quotation`, `Sales Order`, or `Customer` changes.

```text
ERPNext Doc Event
  -> Frappe Webhook
  -> POST n8n webhook URL
  -> n8n workflow
  -> validation, notification, CRM sync, approval, audit, or downstream automation
```

### 1. Create The n8n Webhook Receiver

Create a workflow in n8n with a Webhook trigger:

```text
Webhook -> your processing nodes
```

Webhook node:

- HTTP Method: `POST`
- Path: `erpnext-selling-event`
- Authentication: `None` for a private/internal test, or `Header Auth` for production
- Respond: `Immediately` or `When Last Node Finishes`

The production webhook URL will look like:

```text
https://n8n.example.com/webhook/erpnext-selling-event
```

### 2. Add The Webhook In ERPNext/Frappe v16

In ERPNext/Frappe Desk:

1. Open the global search bar.
2. Search for `Webhook`.
3. Open `Webhook` from the Integrations area.
4. Click `New`.

Configure the Webhook:

- Enabled: checked
- Webhook Doctype: `Sales Order`, `Quotation`, `Opportunity`, `Lead`, `Customer`, or another Selling DocType
- Doc Event: `on_submit`, `on_cancel`, `after_insert`, or `on_update` depending on the workflow
- Request URL: your n8n production webhook URL
- Request Method: `POST`
- Request Structure: `JSON`
- Webhook JSON: use an allowlisted body like the example below

Example JSON body:

```json
{
  "event": "sales_order_submitted",
  "doctype": "{{ doc.doctype }}",
  "name": "{{ doc.name }}",
  "customer": "{{ doc.customer }}",
  "company": "{{ doc.company }}",
  "transaction_date": "{{ doc.transaction_date }}",
  "grand_total": "{{ doc.grand_total }}",
  "status": "{{ doc.status }}",
  "modified": "{{ doc.modified }}"
}
```

For production, add a shared secret header and validate it in n8n:

```text
X-ERPNext-Webhook-Secret: your-long-random-secret
```

If you use Frappe's Webhook Secret field, Frappe adds an `X-Frappe-Webhook-Signature` header generated from the payload and secret. You can verify this signature in n8n with a Code node if needed.

Official Frappe reference:

- [Frappe Webhooks](https://docs.frappe.io/framework/user/en/guides/integration/webhooks)

## Development

```bash
npm install
npm run lint
npm audit --omit=dev
npm run build
npx @n8n/node-cli lint
npx @n8n/node-cli build
npm pack --dry-run
```

Current verification for `0.1.1`:

```text
npm run lint            passed
npm audit --omit=dev    found 0 vulnerabilities
npm run build           passed
npx @n8n/node-cli lint  passed
npx @n8n/node-cli build passed
npm pack --dry-run      passed
```

Expected runtime dependency policy:

- `n8n-workflow` stays in `devDependencies` for local TypeScript/lint/build tooling.
- `n8n-workflow` is declared in `peerDependencies` so the host n8n instance provides it at runtime.
- `form-data` is pinned through `overrides` to avoid vulnerable transitive versions.

## Current Status

Initial package scaffold and live n8n verification are complete.

Tested workflow artifacts:

- `n8n-webhook-erpnext-selling-get-customers.workflow.json`
- `n8n-webhook-erpnext-selling-v2-real-sales-flow-test.workflow.json`
- `n8n-webhook-erpnext-selling-v2-sales-order-amend-test.workflow.json`
- `n8n-webhook-erpnext-selling-v2-negative-cases-test.workflow.json`

Live tested results:

- GET Customers returned allowlisted Customer fields with no credential material.
- Real sales flow passed: Customer, disabled Customer, Lead, Items, Opportunity, submitted Quotation, submitted Sales Order, submitted Sales Invoice, and submitted Payment Entry.
- Sales Order amend passed: original `SAL-ORD-2026-00002` cancelled, amended `SAL-ORD-2026-00002-1` submitted.
- Negative cases passed: duplicate Item, Sales Order missing Customer, delete submitted Sales Order, and fake Sales Order lookup failed as expected.
- All temporary workflows were deactivated after testing and confirmed to return `404 Active version not found`.

### Live Test Evidence

GET Customers:

```text
Workflow id: sellGetCustomers01
Path: /webhook/erpnext-selling-get-customers
HTTP result: 200 OK
Fields returned: name, customer_name, customer_group, territory, disabled
Credential scan: no API key, API secret, Authorization header, token, or password returned
```

Real sales flow:

```text
Workflow id: sellRealSalesFlowV2Test01
Path: /webhook/erpnext-selling-v2-real-sales-flow-test
Run id: N8N-SELL-REAL-1779071479090
Status: passed
Customer: N8N-SELL-REAL-1779071479090 KH Đặc biệt & Space / Test
Disabled Customer: N8N-SELL-REAL-1779071479090 Disabled Customer
Lead: CRM-LEAD-2026-00016
Opportunity: CRM-OPP-2026-00002
Quotation: SAL-QTN-2026-00003
Sales Order: SAL-ORD-2026-00001
Sales Invoice: ACC-SINV-2026-00003
Payment Entry: ACC-PAY-2026-00005
Security findings: []
```

ERPNext database verification:

```text
Quotation SAL-QTN-2026-00003: docstatus 1, party Customer, status Open, grand_total 585.
Opportunity CRM-OPP-2026-00002: party Customer, status Open, base_total 150.
Sales Order SAL-ORD-2026-00001: docstatus 1, status To Deliver and Bill, grand_total 575, per_billed 0.
Sales Invoice ACC-SINV-2026-00003: docstatus 1, Paid, grand_total 150, outstanding_amount 0.
Payment Entry ACC-PAY-2026-00005: docstatus 1, Receive, paid_amount 150, received_amount 150.
GL Entry:
  Sales Invoice ACC-SINV-2026-00003 | 2 rows | debit 150 | credit 150 | cancelled 0
  Payment Entry ACC-PAY-2026-00005 | 2 rows | debit 150 | credit 150 | cancelled 0
```

Sales Order amend:

```text
Workflow id: sellSalesOrderAmendV2Test01
Path: /webhook/erpnext-selling-v2-sales-order-amend-test
Run id: N8N-SELL-AMEND-1779071568003
Status: passed
Original Sales Order: SAL-ORD-2026-00002
Original docstatus: 2
Amended Sales Order: SAL-ORD-2026-00002-1
Amended from: SAL-ORD-2026-00002
Amended docstatus: 1
Amended qty: 2
```

Negative cases:

```text
Workflow id: sellNegativeCasesV2Test01
Path: /webhook/erpnext-selling-v2-negative-cases-test
Status: passed
Sales Order under test: SAL-ORD-2026-00001
Failed as expected:
  duplicateItem
  salesOrderMissingCustomer
  deleteSubmittedSalesOrder
  getFakeSalesOrder
Security findings: []
```

Cleanup verification:

```text
/webhook/erpnext-selling-get-customers -> 404 Active version not found
/webhook/erpnext-selling-v2-real-sales-flow-test -> 404 Active version not found
/webhook/erpnext-selling-v2-sales-order-amend-test -> 404 Active version not found
/webhook/erpnext-selling-v2-negative-cases-test -> 404 Active version not found
```

## Release Checklist

Before tagging a release:

```bash
npm ci
npm run lint
npx @n8n/node-cli lint
npm run build
npx @n8n/node-cli build
npm audit --omit=dev
npm pack --dry-run
```

For provenance publishing, push a semver tag that matches `package.json`:

```bash
git tag v0.1.x
git push origin main
git push origin v0.1.x
```

The GitHub Actions workflow publishes to npm with provenance:

```text
npm publish --access public --provenance
```

## Test Policy

The ERPNext LXD environment used by this project is allowed to receive realistic demo/test sales data. Selling workflow tests should use traceable test prefixes and should avoid leaking raw ERPNext documents or credential-like fields in public webhook responses.

Temporary write-test workflows should be deactivated after verification. Current live tests confirmed these endpoints returned `404 Active version not found` after cleanup:

```text
/webhook/erpnext-selling-get-customers
/webhook/erpnext-selling-v2-real-sales-flow-test
/webhook/erpnext-selling-v2-sales-order-amend-test
/webhook/erpnext-selling-v2-negative-cases-test
```

## Security Notes

Recommended baseline:

- Use a dedicated ERPNext API user for n8n integrations.
- Avoid using a full Administrator API key in production.
- Scope ERPNext roles to the exact DocTypes and actions needed by the workflow.
- Prefer `Get Many` with explicit `Fields` over `Get` when exposing webhook responses, because `Get` can return full documents including comments, owners, child tables, totals, accounting metadata, and contact details.
- Keep n8n webhook URLs private unless they are meant to be public.
- Add authentication to public n8n webhooks, such as header auth, reverse proxy auth, VPN, IP allowlisting, or a shared secret.
- Do not log API keys, API secrets, Authorization headers, raw upstream error bodies, or full sales documents into external systems unless there is a clear retention policy.
- Use HTTPS for public traffic.
- If n8n and ERPNext are on the same VPS or private network, prefer the internal ERPNext URL plus `Site Host Header`.
- Rotate API keys after testing, after staff changes, and after any suspected exposure.
- Review n8n execution data retention for customer records, quotations, sales orders, invoices, payment allocations, and CRM data.
- Deactivate temporary write-test workflows after verification.

## Security Notice

This package pins transitive `form-data` resolution with an npm `overrides` entry so `npm audit --omit=dev` reports no known vulnerabilities at release time. Keep this override in place until upstream dependencies resolve to a safe version without assistance.

In this package's tested deployment model, security risk is also reduced by:

- Internal network access between n8n and ERPNext.
- Reverse proxy or VPN controls for public endpoints.
- Dedicated ERPNext API credentials with scoped roles.
- Explicit field selection for public webhook responses.
- Safe summary nodes for write-heavy lifecycle tests.
- Avoiding public exposure of generic Custom DocType and Frappe Method workflows.

Do not treat this mitigation as a permanent substitute for dependency maintenance. Re-run `npm audit --omit=dev` before publishing a new package version and upgrade compatible n8n dependencies when the upstream dependency chain allows it without breaking n8n node compatibility.

## Deployment Checklist For SME And Mid-Market Teams

Before going live:

- Confirm ERPNext/Frappe version and choose API `v1` or `v2`.
- Create a dedicated ERPNext integration user.
- Assign only the ERPNext roles needed for Customer, Lead, Opportunity, Quotation, Sales Order, Item, and any Custom DocTypes used by your workflow.
- Configure n8n credentials with the ERPNext internal URL when available.
- Set `Site Host Header` if ERPNext is served by a named Frappe site.
- Use allowlisted fields in public read workflows.
- Add shared-secret or proxy authentication for public write webhooks.
- Test write operations in a staging or dedicated ERPNext test site before production.
- Deactivate temporary write-test workflows after verification.
- Review n8n execution data retention for customer and financial documents.

Suggested production approach:

- Keep read-only customer, opportunity, quotation, and sales-order reporting workflows separate from write-heavy workflows.
- Put quotation and sales-order creation behind validation or approval logic.
- Use explicit request validation before creating or submitting ERPNext documents.
- Return safe summaries instead of raw ERPNext documents from public webhooks.
- Keep audit trails for submitted, cancelled, and amended Quotations, Sales Orders, Invoices, and Payment Entries.

## Troubleshooting

- `401` or `403`: verify API key, API secret, user roles, and DocType permissions in ERPNext.
- TLS `EPROTO` or `tlsv1 alert internal error`: use the internal ERPNext HTTP URL from n8n when the public domain is protected by a reverse proxy or VPN layer.
- Frappe site not found or wrong site: set `Site Host Header` to the public ERPNext site name.
- `Customer` create fails: verify `customer_group` and `territory` exist in the ERPNext site. The live test site uses `Individual` and `Vietnam`.
- `Sales Partner` create fails: verify the required Sales Partner setup fields for your ERPNext version. The core live-test flow does not require Sales Partner.
- `Quotation` submit fails: verify `quotation_to`, `party_name`, valid item rows, price list/currency, and company.
- `Sales Order` submit fails: verify customer, item sales flags, delivery date, company, UOM, and any pricing/tax rules.
- `Sales Invoice` or `Payment Entry` fails through `Custom DocType`: verify receivable account, cash/bank account, income account, linked invoice, fiscal period, and party.
- Cancel/amend fails after downstream documents exist: ERPNext may correctly block cancellation of linked documents until downstream effects are cancelled.
- Delete fails for submitted documents: ERPNext normally requires cancel first and may still block deletion based on links and permissions.

## Scope Boundaries

This package is intentionally Selling-focused. Other ERPNext modules should live in separate packages so each module can evolve independently:

- `n8n-nodes-frappe-core`
- `n8n-nodes-erpnext-hrms`
- `n8n-nodes-erpnext-accounting`
- `n8n-nodes-erpnext-buying`
- `n8n-nodes-erpnext-selling`
- `n8n-nodes-erpnext-stock`

Use `Custom DocType` for adjacent documents when a Selling workflow needs to cross into another module, such as `Sales Invoice`, `Payment Entry`, and `Delivery Note`. If a workflow is mostly financial, prefer the Accounting package.

## References

Frappe / ERPNext:

- [Frappe REST API](https://docs.frappe.io/framework/user/en/api/rest)
- [Frappe token based authentication](https://docs.frappe.io/framework/v15/user/en/guides/integration/rest_api/token_based_authentication)
- [Frappe Webhooks](https://docs.frappe.io/framework/user/en/guides/integration/webhooks)
- [ERPNext Selling](https://docs.frappe.io/erpnext/user/manual/en/selling)
- [Customer](https://docs.frappe.io/erpnext/user/manual/en/customer)
- [Lead](https://docs.frappe.io/erpnext/user/manual/en/lead)
- [Opportunity](https://docs.frappe.io/erpnext/user/manual/en/opportunity)
- [Quotation](https://docs.frappe.io/erpnext/user/manual/en/quotation)
- [Sales Order](https://docs.frappe.io/erpnext/user/manual/en/sales-order)
- [Sales Invoice](https://docs.frappe.io/erpnext/user/manual/en/sales-invoice)
- [Payment Entry](https://docs.frappe.io/erpnext/user/manual/en/payment-entry)

n8n:

- [Community nodes](https://docs.n8n.io/integrations/community-nodes/)
- [Install community nodes](https://docs.n8n.io/integrations/community-nodes/installation/)
- [Creating nodes](https://docs.n8n.io/integrations/creating-nodes/)
- [Declarative-style nodes](https://docs.n8n.io/integrations/creating-nodes/build/declarative-style-node/)
- [Node linter](https://docs.n8n.io/integrations/creating-nodes/test/node-linter/)
- [Submit community nodes](https://docs.n8n.io/integrations/creating-nodes/deploy/submit-community-nodes/)

## Maintainer Notes

Prepared and reviewed with care by Codex for the `n8n2erpnext` ERPNext Selling integration work.
