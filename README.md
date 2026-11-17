# ACCOUNTIX AI — Engineering Foundation 0.40.0

## Included in 0.27.0

- Added a protected financial-statements API at `/api/reports/financial-statements`, scoped to the authenticated user's company membership.
- Added a financial statements page at `/reports/financial-statements` with a period income statement and an as-of-date balance-sheet summary.
- The report uses posted journal entries only, includes inactive accounts that may still hold historical balances, and visibly lists account types it cannot classify rather than silently including them.
- The report displays the 5,000-entry cap and accounting limitations; it is a preliminary management report, not a Moroccan statutory statement or certified balance sheet.
- Added logic and API contract tests for date-scoped statements, classification and overflow handling.

## Previous releases

ACCOUNTIX AI is a Morocco-first, multilingual accounting platform foundation (Arabic, French and English) built with Next.js, TypeScript, Prisma and PostgreSQL.

### Historical scope: 0.4.0

- Email/password registration and login API routes, with scrypt password hashing and signed HttpOnly session cookies.
- Company creation at registration and owner membership association.
- Arabic login/register screen at `/login`, authenticated account page at `/account`, and contact page at `/contact` using `nexsoai859@gmail.com`.
- Logout API route.

## Previously included in 0.3.0

- Responsive multilingual dashboard shell with Arabic RTL and French/English LTR switching.
- PostgreSQL data model for companies, customers, suppliers, sales/purchase invoices, invoice lines, accounts, journal entries, subscriptions and audit logs.
- Tenant-aware composite foreign keys for invoice customers/suppliers and journal entry/account relations. These database constraints help prevent cross-company references; they do **not** replace authenticated authorization checks in application code.
- Integer minor-unit money utilities with explicit enforcement of the current PostgreSQL `Int` storage ceiling (2,147,483,647 centimes).
- Invoice calculation helper using integer/BigInt arithmetic, fractional quantity scaling (three decimal places), basis-point tax rates and deterministic rounding.
- Double-entry journal validation for account IDs, non-negative integer amounts, balanced totals and supported storage ranges.
- PostgreSQL health endpoint that reports readiness only when the database responds.
- Environment template with server-only secret placeholders; no real credentials are included.

## Verification status

The pure logic suite currently has **10 passing tests** on Node.js 22. ZIP integrity is checked when the release archive is created.

A full dependency installation did not complete in the build environment, so `npm run typecheck`, Prisma schema validation/client generation, and `npm run build` have **not** been verified for this release. Database integration tests were also not run because no PostgreSQL service was available. Do not interpret the passing logic tests as a production certification.

This remains a **developer foundation / non-production MVP**, not a finished statutory accounting product. The dashboard still contains sample figures. Production-grade authentication hardening (email verification, password reset, rate limiting, session revocation and role checks on every data route), persistent CRUD workflows, payment processing, AI document extraction, Morocco tax configuration and statutory reports, backup/restore, monitoring and independent security review are not yet complete. The new login API requires a configured `AUTH_SECRET` and a working PostgreSQL database. Do not enter real customer or financial data or accept paid subscriptions until these controls are implemented and tested.

## Requirements

- Node.js 22 or newer
- npm
- PostgreSQL

## Local setup

1. Extract the ZIP and open the `ACCOUNTIX_AI` folder.
2. Copy `.env.example` to `.env` and set `DATABASE_URL` to a PostgreSQL database you control.
3. Run `npm install`.
4. Run `npx prisma validate` and `npx prisma generate`.
5. For a disposable development database only, run `npx prisma db push`. Use reviewed migrations for production.
6. Run `npm run typecheck` and `npm run test:logic`.
7. Run `npm run dev` and open `http://localhost:3000`.
8. Visit `/api/health`; it returns `200` only when PostgreSQL responds, otherwise `503`.

## Vercel deployment

Import the repository into Vercel and configure `DATABASE_URL` in Project Settings → Environment Variables for the intended deployment environments. A reachable managed PostgreSQL service is required. Redeploy after changing environment variables. Do not deploy this foundation for real financial records until the production milestones below are complete.

Never put database, AI-provider or payment secrets in browser code, Git commits, screenshots or public repositories. Keep secrets in server-side environment variables.

## Morocco-first accounting caution

The data model and calculation helpers are engineering primitives, not certification of compliance with Moroccan CGNC, VAT, corporate tax, e-invoicing, withholding, invoice wording, archival or reporting rules. Tax rates, chart-of-account mappings, numbering and statutory reports must be reviewed against current official rules by a qualified Moroccan accountant before production use. Other jurisdictions require separate country packs and review.

## Confirmed product decisions

- Product name: ACCOUNTIX AI.
- Initial market: Morocco.
- Interface languages: Arabic, French and English.
- Subscription billing intent: monthly and annual plans.
- Payment intent: card payments through a compliant provider without storing card data; bank transfers require administrative verification before activation or renewal.

These are product requirements, not integrations already implemented.

## Remaining production milestones

1. Authentication, verified sessions, company membership and role-based access control on every server action/API.
2. Authorization and tenant-isolation tests, CSRF/rate-limit protections, secure file upload and tamper-aware audit trail.
3. Persistent customer/supplier/invoice/expense CRUD and immutable journal posting/reversal workflow.
4. Accountant-reviewed Moroccan CGNC chart, tax rules, VAT reports, invoice numbering and document retention.
5. OCR/AI pipeline with schema validation, human approval and no autonomous posting or payment.
6. Card provider integration with signed webhooks; bank-transfer payments activated only after administrative verification.
7. Subscription entitlements, cancellation/refund handling, privacy policy and terms reviewed for applicable law.
8. Automated tests, backups/restore drills, monitoring, incident response and independent security review.

No software can be guaranteed error-free. A passing unit test suite is not proof of legal/accounting correctness or production security.

## v0.5.0 — Expense register

This release adds a company-scoped expense register:
- `Expense` database model with amount stored in integer minor units (centimes).
- Authenticated `GET /api/expenses` and `POST /api/expenses` endpoints.
- Membership checks ensure a user can only create expenses for a company they belong to.
- Audit log entry is written when an expense is created.
- Arabic expense listing and entry form at `/expenses`.

### Database migration
After configuring `DATABASE_URL` and `AUTH_SECRET`, install dependencies and run:

```bash
npm install
npx prisma validate
npx prisma generate
npx prisma db push
npm run typecheck
npm run build
```

`db push` is suitable for initial development only. For a production deployment, use reviewed Prisma migrations and configure automated backups before storing company records.

### Expense API
`POST /api/expenses` accepts authenticated JSON with `companyId`, `description`, `category`, `amountMinor` (MAD centimes), and ISO `expenseDate`; optional fields are `vendor`, `reference`, and `notes`. The server derives the user identity from the signed session and checks company membership. Do not trust a company ID supplied by a browser without this server-side check.


## Version 0.6.0 — Contacts and expense integrity

- Added authenticated customer and supplier list/create pages and API routes.
- Every create operation checks company membership and writes its audit log in the same database transaction.
- Expense creation and its audit log now run atomically in one transaction.
- Customer and supplier records are scoped to the companies attached to the signed-in user.

Before deployment, run `npm install`, `npx prisma validate`, `npm run typecheck`, `npm run test:logic`, and `npm run build`. Configure `DATABASE_URL` and a strong `AUTH_SECRET` (32+ characters). This version remains a development foundation, not a certified Moroccan accounting system.

## Version 0.7.0 — Invoice drafts

- Added company-scoped sales and purchase invoice listing and creation.
- Invoice lines calculate subtotal, VAT and total on the server; browser totals are never trusted.
- Monetary values are persisted as integer centimes and checked against the current PostgreSQL `Int` ceiling.
- Sales invoices require a customer; purchase invoices require a supplier. The selected party is verified to belong to the same company.
- Invoice creation, line creation and audit logging run in one database transaction.
- New invoices remain `DRAFT`. This release does not issue legally numbered invoices, post accounting entries, send documents, or certify Moroccan VAT treatment.

### Invoice API
`POST /api/invoices` requires an authenticated member and accepts `companyId`, `type` (`SALES` or `PURCHASE`), `number`, the matching `customerId` or `supplierId`, ISO dates, and one to 100 item lines. Each line has `description`, `quantity`, `unitPrice` in MAD, and `taxRate` as a percentage (for example `20`). The server calculates all totals and validates that the party belongs to the requested company.

### v0.7.0 verification
Run `npm install`, `npm run test:logic`, `npm run test:api`, `npx prisma validate`, `npm run typecheck`, and `npm run build`. Database schema validation and a production build must be confirmed in the deployment environment before real company data is entered.

## Version 0.8.0 — Chart of accounts and journal workflow

- Added authenticated, company-scoped account listing and account creation at `/api/accounting/accounts`.
- Added authenticated journal draft creation and listing at `/api/accounting/journals`.
- Journal creation validates at least two lines, integer centime amounts, non-negative debit/credit values, balanced totals, and active accounts belonging to the selected company.
- Added a draft-only posting endpoint at `/api/accounting/journals/:id/post`. Posting rechecks balance and uses a conditional state transition to avoid posting the same draft twice; audit logging is part of the transaction.
- Added an Arabic accounting workspace at `/accounting` for account setup, simple two-line journal drafts, and posting.

### Accounting limitations
The current screen creates simple two-line entries only; compound multi-line entries, reversal workflow, trial balance, general ledger reports, automatic posting from invoices/expenses, and period closing are not implemented. The account type labels are generic engineering classifications, not an approved Moroccan CGNC chart. Have a qualified Moroccan accountant validate account codes, mappings, tax handling and reporting before professional use. Posted journals are not yet immutable against all administrative/database-level changes and there is no reversal workflow, so this is not a production ledger.

### v0.8.0 verification
Run `npm install`, `npm run test:logic`, `npm run test:api`, `npx prisma validate`, `npm run typecheck`, and `npm run build` after configuring the environment. No successful full build or PostgreSQL integration run is claimed for this archive.


## v0.9.0 — Trial balance reports
- Added an authenticated `/reports` page with a trial balance based on posted journal entries only.
- Reports are scoped to the signed-in user's company memberships.
- Added account-balance aggregation validation tests.
- Current limitations: no fiscal-period filters, opening balances, closing entries, reconciliation workflow, or automated invoice/expense posting. The report is an early-stage operational view, not a certified Moroccan statutory report.


### v0.9.0 verification
- Logic tests: 12 passed, 0 failed.
- API contract tests: 12 passed, 0 failed.
- ZIP archive integrity: verified after packaging.
- TypeScript, Prisma validation/generation, Next.js production build, and live PostgreSQL integration remain unverified because dependencies/database are not available in this build environment.

## v0.10.0 — General ledger and preliminary statements
- Added authenticated `/ledger` view of posted journal-line movements, grouped by company and account, with a running debit-minus-credit balance.
- Added preliminary per-company income and financial-position summaries to `/reports`, based on account classifications and posted journal activity.
- Added navigation between the accounting workspace, ledger and reports.
- The ledger is limited to the latest 3,000 posted journal lines and has no date/account filters or export yet.
- Financial summaries are provisional and depend entirely on the account types assigned by the user. They do not implement fiscal-year closing, opening balances, depreciation, provisions, tax reconciliation, cash-flow statement rules, or official CGNC mappings. They must not be used as certified statutory statements.

### v0.10.0 verification
- Logic tests: 12 passed, 0 failed.
- API contract tests: 12 passed, 0 failed.
- TypeScript, Prisma validation/generation, production build and live database integration have not been run in this environment because dependencies and a PostgreSQL instance are unavailable. Run `npm install`, `npx prisma validate`, `npm run typecheck`, `npm run test:logic`, `npm run test:api`, and `npm run build` before deployment.

## v0.11.0 — Secure ledger CSV export
- Added an authenticated CSV export endpoint for posted general-ledger entries.
- Exports only companies the signed-in user belongs to; an optional company ID is checked against memberships before querying.
- Supports inclusive `from` and `to` date filters in `YYYY-MM-DD` format and rejects invalid or reversed date ranges.
- Exports journal-line detail with company, date, reference, description, account, debit and credit columns; capped at 5,000 journal entries per export.
- Escapes CSV quotes and prefixes untrusted spreadsheet-formula-like text to reduce CSV injection risk. Responses are marked private/no-store and served as UTF-8 CSV with BOM.
- Added a download link to the general ledger page.

Example: `/api/exports/ledger?from=2026-01-01&to=2026-12-31` or `/api/exports/ledger?companyId=YOUR_COMPANY_ID&from=2026-01-01`.

The export contains posted journal entries only; it is not an official accounting filing. Verify exported totals against the ledger and retain access controls when sharing CSV files.

### v0.11.0 verification
Run `npm install`, `npm run test:logic`, `npm run test:api`, `npx prisma validate`, `npm run typecheck`, and `npm run build`. Full TypeScript/build/database checks are not claimed unless executed successfully in the target environment.


## v0.12.0 — Receivables and payables aging
- Added an authenticated `/aging` report for issued and overdue sales and purchase invoices with a due date.
- Separates customer receivables from supplier payables and groups invoice totals into not-yet-due, 1–30, 31–60, and over-60-day buckets.
- Uses UTC calendar-day comparisons to avoid time-of-day boundary drift.
- Report is company-scoped using the signed-in user's memberships and capped at 1,000 invoices.
- Important limitation: the application does not yet record payments or outstanding balances. The report therefore excludes partially paid invoices and shows full invoice totals for ISSUED/OVERDUE only; it is not a true residual-balance aging report.

### v0.12.0 verification
- Added pure-function tests for aging bucket boundaries and sales/purchase separation.
- Run `npm run test:logic`, `npm run test:api`, `npx prisma validate`, `npm run typecheck`, and `npm run build` in an environment with dependencies and PostgreSQL configuration before deployment. No successful production build or database integration is claimed for this archive.


## v0.13.0 — Invoice payments and balances

- Added `InvoicePayment` records with integer minor-unit amounts, payment date, method, reference, notes, creator, and a company-scoped idempotency key.
- Added `POST /api/invoices/:id/payments` to record receipts/payments for issued invoices. It rejects draft/cancelled invoices, dates before issue, duplicate-key conflicts, and amounts above the remaining balance.
- Payment creation, invoice status update (`PARTIALLY_PAID` / `PAID`), and audit event are committed in one serializable database transaction.
- Added `GET /api/invoices/:id/payments` for payment history and computed paid/remaining totals. Invoice list now displays paid and remaining amounts.
- Apply the updated Prisma schema to the configured database with `npm run db:push`, then regenerate the Prisma client with `npm run db:generate`. Back up the database and review schema changes before applying them to an existing production database.
- Clients must send a stable `idempotencyKey` per payment attempt (at least 12 characters) and store it until the request outcome is known.
- This records payment evidence only; it does not create or post accounting journal entries. Journal mapping and accountant review remain separate.


## v0.14.0 — Payment-aware aging report
- Corrected the aging report to include partially paid invoices and subtract recorded payments from invoice totals.
- Fully paid invoices and zero-balance documents are excluded from outstanding balances.
- The detail table now shows invoice total, recorded payments and remaining balance; summary buckets use remaining balances only.
- Added tests for partial payments, fully paid invoices and protection against negative balances.

### v0.14.0 verification
- Logic tests: 15 passed, 0 failed.
- API contract tests: 17 passed, 0 failed.
- ZIP integrity: checked after packaging.
- Prisma validation, TypeScript typecheck, Next.js production build and live PostgreSQL integration remain unverified in this environment.

## v0.15.0 — Invoice payment entry workflow
- Added an inline payment form to the invoice register for invoices that can accept payments; draft and cancelled invoices show a non-payable state instead.
- Form captures amount, date/time, payment method and optional reference, validates against the currently displayed remaining balance, and submits a fresh idempotency key.
- Successful submissions refresh the invoice register so paid and remaining balances/statuses are recalculated from persisted payment records.
- Hardened idempotent replay: reusing a key with a different invoice, amount, method or payment timestamp is rejected as a conflict.
- This UI is an operational entry point, not a bank/card processor. Card entries are manual records only; no card credentials are collected or stored.

### v0.15.0 verification
- Added static contract tests for the payment form, payable-state gating, and idempotency replay checks.
- Run `npm run test:logic` and `npm run test:api` locally. TypeScript, Prisma, production build and database integration still require dependency installation and a configured PostgreSQL database; do not treat this archive as production-certified.

## v0.16.0 — invoice-to-journal draft workflow

A new authenticated endpoint can create a **DRAFT** journal from an issued invoice:

`POST /api/accounting/invoices/{invoiceId}/draft`

JSON body:

```json
{
  "companyId": "company_id",
  "controlAccountId": "customer_or_supplier_control_account_id",
  "baseAccountId": "revenue_or_purchase_account_id",
  "taxAccountId": "tax_account_id"
}
```

For invoices with zero tax, `taxAccountId` may be omitted. All selected accounts must be active and belong to the selected company. The endpoint only accepts issued, partially paid, paid, or overdue invoices; it creates a balanced draft and records an audit event. Each invoice can have at most one linked journal entry, preventing duplicate source entries. Existing drafts are returned on repeat requests.

Sales draft: debit receivable for the gross total; credit the selected base account and (when applicable) tax account. Purchase draft: debit the selected base and tax accounts; credit the supplier control account. These are configurable account selections, **not a certified Moroccan CGNC chart or tax opinion**. A qualified Moroccan accountant must confirm the account mapping and tax deductibility before posting. Payments are not separately journalized in this release.

After updating the source, regenerate and apply the Prisma schema before using the new endpoint:

```bash
npm install
npm run db:generate
npm run db:push
npm run typecheck
npm run build
```

Database migration and production build were not executed in the packaging environment; do not deploy this schema change to production without a reviewed backup and migration plan.


## v0.17.0 — Payment accounting drafts

- Create a balanced, reviewable journal draft from an existing invoice payment.
- Sales receipts debit the selected cash/bank account and credit the selected customer-control account.
- Purchase payments debit the selected supplier-control account and credit the selected cash/bank account.
- The payment-to-journal relationship is unique per company to prevent duplicate drafts.
- Journal creation and audit logging run in one database transaction.
- This is a configurable draft workflow, not a certified Moroccan chart-of-accounts mapping; review account selection and tax treatment with a qualified accountant before posting.

API: `POST /api/accounting/payments/:id/draft` with `companyId`, `controlAccountId`, and `cashBankAccountId`.

### Date-filtered trial balance API (v0.19.0)

`GET /api/reports/trial-balance?companyId=<id>&from=YYYY-MM-DD&to=YYYY-MM-DD` returns posted journal activity grouped by account for a company the signed-in user belongs to. Date bounds are optional and inclusive (UTC calendar dates). The response includes debit/credit totals, a balance flag, entry count, and a `truncated` flag when the 5,000-entry safety limit is reached. Draft and reversed journals are excluded. This is a management report, not a certified Moroccan statutory statement.


## v0.21.0 — واجهة كشف حركة الحساب

أضيفت صفحة `/account-statement` لتمكين المستخدم من اختيار الشركة والحساب والفترة الزمنية، وعرض الرصيد الافتتاحي وإجماليات المدين والدائن والرصيد الجاري والختامي. تستعمل الصفحة endpoint المحمي `/api/reports/account-statement`، وتعرض تنبيهاً عند بلوغ حد 5,000 حركة. الواجهة لا تنشئ قيوداً ولا ترحلها.


## v0.22.0 — تصدير ميزان المراجعة

أضيف endpoint `GET /api/exports/trial-balance` لتصدير ميزان المراجعة بصيغة CSV، مع اختيار اختياري للشركة والفترة (`companyId`, `from`, `to`). يقتصر على القيود المرحلة، ويتحقق من عضوية الشركة وصحة التواريخ، ويعالج النصوص لمنع CSV formula injection. أضيف رابط التصدير إلى صفحة التقارير. التصدير يعرض حركة القيود المرحلة ضمن النطاق المحدد ولا يحسب الأرصدة الافتتاحية خارج الفترة.


## v0.23.0 — مسودة قيد المصروف

أضيف `POST /api/accounting/expenses/[id]/draft` لإنشاء قيد يومية بحالة `DRAFT` من مصروف مسجل. يتطلب حساب مصروف وحساب صندوق/بنك نشطين ومن نفس الشركة، ويمنع تطابق الحسابين، ويتحقق من توازن القيد ويسجل العملية في AuditLog. تمت إضافة علاقة اختيارية وفريدة لكل شركة بين JournalEntry وExpense عبر `sourceExpenseId` لمنع إنشاء أكثر من قيد مصدر لنفس المصروف.

**تنبيه محاسبي:** القيد المقترح يفترض أن المصروف مدفوع نقداً/بنكياً. إذا كان المصروف غير مدفوع، يجب استعمال حساب مورد/دائن بدل الصندوق أو البنك. القيد يبقى مسودة ولا يُرحّل تلقائياً، كما أن ضريبة المصروفات غير مفصّلة في نموذج Expense الحالي.

## Company accounting settings (v0.24.0)

`GET /api/accounting/settings?companyId=<id>` returns the authenticated member's company-level account mappings. `PUT /api/accounting/settings` accepts `{ companyId, mappings }` and is restricted to OWNER/ADMIN members. Supported mapping keys are `CUSTOMER_CONTROL`, `SUPPLIER_CONTROL`, `CASH`, `BANK`, `SALES_REVENUE`, `PURCHASE_EXPENSE`, `OUTPUT_VAT`, and `INPUT_VAT`. Every mapped account must be active and belong to the selected company; duplicate account IDs are rejected. Changes are written to the audit log.

The settings endpoint stores mappings for future workflow integration. Existing invoice/payment/expense draft endpoints still accept account IDs explicitly in this release; they do not silently consume these settings yet. Apply the Prisma schema through a reviewed migration in a backed-up development database before deployment; do not run schema-changing commands against production without a migration and backup plan.


## v0.25.0 — واجهة إعدادات الربط المحاسبي

- إضافة صفحة `/accounting/settings` لإدارة ربط الحسابات الافتراضية لكل شركة.
- تعرض الصفحة الحسابات النشطة التابعة للشركات التي ينتمي إليها المستخدم فقط.
- تدعم وظائف العملاء والموردين والصندوق والبنك والمبيعات والمشتريات وTVA المدخلات والمخرجات.
- الحفظ عبر API المحمي، مع السماح بالتعديل للمالك والمدير فقط وفق عضوية الشركة.
- إضافة رابط مباشر من مساحة المحاسبة إلى إعدادات الحسابات.
- هذه الصفحة تحفظ الإعدادات ولا تعني أن جميع مولدات القيود تستخدمها تلقائياً بعد؛ يجب ربطها والتحقق منها في مرحلة لاحقة.


## v0.26.0 — automatic use of company accounting mappings

- Invoice journal drafts now resolve customer/supplier control, sales/purchase base, and output/input tax accounts from the saved company mappings. Explicit legacy account IDs remain accepted for compatibility.
- Payment journal drafts now resolve customer/supplier control plus cash or bank based on the recorded payment method. `OTHER` requires an explicit account selection.
- Expense journal drafts use `PURCHASE_EXPENSE` and the new `EXPENSE_CONTROL` mapping. Expenses have no payment-status field, so the draft uses a payable/control counterpart instead of assuming cash or bank payment.
- All generated entries remain `DRAFT`; users must review and post them explicitly.
- Moroccan tax/account classification still requires review by a qualified accountant; this release does not certify CGNC or tax compliance.

## v0.28.0
- Added a company-scoped VAT operational summary grouped by the tax rates actually stored on issued sales and purchase invoice lines.
- The report separates sales/output VAT and purchase/input VAT and displays an indicative arithmetic difference only.
- No Moroccan statutory VAT rate, exemption, deduction eligibility, filing rule, or tax adjustment is inferred. The report is not a tax return and requires review by a qualified Moroccan accountant.

## Version 0.29.0
- Added an authenticated, company-scoped CSV export for the VAT operational summary.
- Export uses the same saved invoice-line tax rates and issued-invoice statuses as the VAT report; it does not infer statutory Moroccan tax rules.
- CSV includes a UTF-8 BOM and semicolon delimiters for spreadsheet compatibility, and is marked no-store.
- Added an export link that carries the selected company and date range.


## v0.30.0 — subscription status overview

- Added an authenticated `/subscriptions` page showing the current plan, status, optional period end, and recorded provider for each company belonging to the signed-in user.
- Company records are queried only through the authenticated user's membership company IDs.
- Added an entry point from the dashboard.
- This is a read-only status overview. Card processing, bank-transfer verification, paid-plan activation, plan enforcement, and subscription prices are not implemented or configured in this release. No payment is accepted or represented as completed.

## Version 0.31.0

- Added CSV export for the financial statements report (income statement, balance sheet, and unclassified accounts).
- Export is scoped to a company the signed-in user belongs to and includes only posted journal entries.
- CSV uses UTF-8 BOM and semicolon separators for spreadsheet compatibility, with escaped cell values.
- Export repeats important limitations: opening balances, closing entries, accruals, depreciation, inventory adjustments and tax calculations are not automatically generated; Moroccan CGNC treatment requires qualified review.
- The report remains a management aid and is not a certified statutory financial statement.

## Version 0.32.0 — bank-transfer review workflow

- Added authenticated bank-transfer payment requests for company owners and admins.
- Requests include plan, monthly/annual cycle, MAD amount, bank reference, and optional payer name.
- A company can have only one pending request at a time; bank references are unique per company.
- Platform review is restricted to login emails listed in `ADMIN_EMAILS` (comma-separated). An approval activates the requested subscription only after a human reviewer confirms receipt of funds; rejection leaves the subscription unchanged.
- All request and review actions create audit log entries. The application does not collect card details and does not claim to process card payments.

### Deployment notes for 0.32.0

1. Set `ADMIN_EMAILS` in the deployment environment to the exact email address(es) used by trusted platform administrators. Do not use the public support email unless that address is actually the administrator login.
2. Apply the additive database migration with `npx prisma migrate deploy` after taking a database backup. Do not use `prisma db push` as a production migration substitute.
3. Set the four `ACCOUNTIX_PRICE_..._MAD` variables documented in `.env.example` before enabling bank-transfer requests. Leave them unset to keep paid-plan requests safely blocked.
4. Regenerate the Prisma client and deploy the application (`npx prisma generate`, then the normal production build).
5. Verify a test transfer request end-to-end before accepting real payments. A reviewer must independently verify the bank statement/reference and amount; the form itself is not proof of payment.

In v0.32.0 the request amount was entered by the company user and therefore was not a trusted price. v0.33.0 replaces that behavior: the server now checks the amount against the configured plan price and blocks requests if the price is missing. Card processing is not implemented in this release.


## v0.33.0 — Subscription price integrity

- Bank-transfer requests now use server-side configured prices instead of accepting a user-entered amount.
- Configure `ACCOUNTIX_PRICE_PREMIUM_MONTHLY_MAD`, `ACCOUNTIX_PRICE_PREMIUM_ANNUAL_MAD`, `ACCOUNTIX_PRICE_ENTERPRISE_MONTHLY_MAD`, and `ACCOUNTIX_PRICE_ENTERPRISE_ANNUAL_MAD` in the deployment environment before accepting paid-plan requests. Values are decimal MAD amounts (up to two decimal places).
- If a price is missing, requests are blocked; no official prices are assumed or invented.


## v0.34.0 — Server-authoritative transfer amount

The bank-transfer request endpoint no longer accepts a client-supplied amount. The request schema is strict, and the server derives the amount in minor currency units from the configured plan price before inserting the payment request and audit record. The browser sends only company, plan, billing cycle, bank reference, and optional payer name. This reduces price-tampering risk. It does not process card payments or independently verify bank deposits; an authorized platform administrator must still verify receipt before approval.


## v0.35.0 — إلغاء طلب التحويل البنكي
- يمكن لمالك الشركة أو مديرها إلغاء طلب تحويل ما زال قيد المراجعة.
- لا يمكن إلغاء طلب مقبول أو مرفوض أو طلب تابع لشركة أخرى.
- يسجل الإلغاء في سجل التدقيق، مع الحفاظ على سجل الطلب بدلاً من حذفه.
- تمت إضافة حالة `CANCELLED` عبر migration مستقلة.
- أصبحت بيانات مراجعة الإدارة ترفض الحقول غير المعروفة.

### ترحيل قاعدة البيانات لهذا الإصدار
نفّذ `npx prisma migrate deploy` بعد أخذ نسخة احتياطية من PostgreSQL، وبعد التأكد من أن اتصال `DATABASE_URL` يشير إلى قاعدة البيانات الصحيحة. لا تستخدم `prisma db push` لهذا التحديث.

## v0.36.0 — عكس القيود المرحلة بأثر محاسبي واضح
- إضافة API لعكس القيد المرحّل دون حذف أو تعديل تفاصيله الأصلية.
- يتم تحويل القيد الأصلي إلى `REVERSED` وإنشاء قيد معاكس `POSTED` بتاريخ تنفيذ العكس، داخل معاملة قاعدة بيانات واحدة.
- إضافة سجل تدقيق للعملية وربط القيدين في بيانات التدقيق.
- تحديث التقارير والتصديرات لتحتسب القيد الأصلي المعكوس والقيد العكسي معاً، حتى يكون الأثر الصافي صفراً مع الاحتفاظ بالأثر التاريخي.
- لا يمكن عكس مسودة أو قيد سبق عكسه.

## ملاحظة بوابة الدفع بالبطاقة في المغرب
لم يتم تفعيل الدفع بالبطاقة في هذا الإصدار. قائمة Stripe الرسمية للدول المدعومة لا تتضمن المغرب حالياً. صفحة CMI الرسمية تعرض حل الدفع الإلكتروني وتوجّه التجار إلى جهات الاستحواذ الشريكة، بينما توضح Payzone أن الوثائق التقنية التفصيلية تُتاح بعد التواصل مع فريقها. لذلك لا نضع تكاملاً تخمينياً أو مفاتيح وهمية؛ يلزم عقد تاجر ووثائق API وبيانات sandbox رسمية قبل بناء موصل إنتاجي.
راجع `docs/PAYMENT_PROVIDER_RESEARCH.md` للمراجع الرسمية وخطة الدمج الآمن.


## v0.38.0 — صلاحيات أدوار القيود المحاسبية
- إضافة دالة مركزية للتحقق من دور العضو داخل الشركة المطلوبة، وليس الاكتفاء بوجود العضوية.
- إنشاء القيود اليدوية وترحيلها وعكسها أصبح متاحاً فقط للأدوار `OWNER` و`ADMIN` و`ACCOUNTANT`.
- الأعضاء ذوو الأدوار الأخرى يحصلون على `403 INSUFFICIENT_ROLE` حتى لو كانت لديهم عضوية في الشركة.
- يظل التحقق من الشركة منفصلاً عن التحقق من الدور، مع استمرار سجل التدقيق والتحقق من توازن القيد.
- هذه الحماية تخص مسارات القيود الثلاثة المذكورة فقط؛ لا تعني أن جميع مسارات المنصة اكتملت فيها صلاحيات RBAC. يجب إكمال مراجعة الأدوار لكل API قبل الإنتاج.

### v0.38.0 verification
- API contract tests: 65 passed, 0 failed.
- Logic tests: 23 passed, 0 failed.
- ZIP archive integrity: verified after packaging.
- لم يتم تشغيل `npm run typecheck` أو `npm run build` أو Prisma validation أو تكامل PostgreSQL في بيئة إعداد هذا الإصدار؛ يلزم تشغيلها بعد تثبيت الاعتماديات وربط قاعدة اختبار.
## v0.38.0 security update

Financial write endpoints now enforce company-scoped role checks: chart-of-account maintenance is OWNER/ADMIN only, while invoice and expense creation, payment recording, and source journal-draft creation require OWNER/ADMIN/ACCOUNTANT. See `docs/RELEASE_NOTES_v0.38.0.md`.

## v0.39.0 — Company team invitations

Owners and admins can create seven-day invitations for ACCOUNTANT and VIEWER roles, revoke pending invitations, and manage members at `/team`. Invitees must be signed in with the exact invited email to accept at `/invite`. Tokens are stored as SHA-256 hashes. Email delivery and independent email verification are not configured: the inviter must share the generated link manually and only with the intended recipient. The login flow preserves the invite URL for post-login acceptance. Apply the `20261117000100_company_invitations` migration before deploying.

The VIEWER role is read-only for customer and supplier creation endpoints. Continue reviewing every API route before production; this is not a completed platform-wide RBAC audit.


## v0.39.0 — دعوات أعضاء الشركات

- صفحة `/team` لإدارة الأعضاء والدعوات المعلقة.
- روابط دعوة محددة الدور (ACCOUNTANT أو VIEWER) صالحة سبعة أيام، مع تخزين hash فقط للرمز.
- قبول الدعوة يتطلب تسجيل الدخول بالبريد المطابق، وتتم إضافة العضوية وقبول الدعوة وسجل التدقيق في معاملة واحدة.
- إلغاء الدعوات وتسجيل ذلك في AuditLog.
- دور VIEWER لا يستطيع إنشاء العملاء أو الموردين.
- لا يوجد إرسال بريد آلي أو تحقق بريد مستقل؛ مشاركة الرابط يدوياً فقط.


## v0.40.0 — Invitation onboarding completion
- Added an invitation acceptance page and authenticated invite flow.
- New invited users can register directly into the invited company using the single-use invitation token, without creating a separate company.
- Registration checks invited email, token hash, expiry and revocation, and atomically claims the invitation.
- Added contract coverage for invited-user registration and acceptance routing.
- Email delivery remains manual; production build and database integration tests remain required.
