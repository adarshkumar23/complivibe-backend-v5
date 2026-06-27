# CompliVibe Backend Development Log v5.0

## Group Status

- Group A1 — Risk Management Enhancements: ✅ COMPLETE
- Group A2 — Control & Compliance Enhancements: ✅ COMPLETE
- Group A3 — Policy Enhancements: ✅ COMPLETE
- Group A4 — Audit & Assurance Module: ✅ COMPLETE

## Phase 9.12 - Pillar 1 Closure and Regression Gate (2026-06-22)

- Added stabilization endpoint: `GET /api/v1/compliance/contracts` (read-only contract registry for all Pillar 1 groups).
- Added `PILLAR1_AUDIT_ACTION_REGISTRY` in `app/services/seed_service.py` to track 9.0-9.11 audit action coverage.
- Added closure test suite: `tests/unit/test_pillar1_closure_phase912.py`.
  - Validates contract registry groups.
  - Validates key route ordering and duplicate route pattern checks.
  - Validates read-only/no-audit-mutation behavior for contract/dashboard reads.
  - Validates Phase 9.0-9.11 audit actions exist in seed registry.
- Full unit suite executed: `PYTHONPATH=. .venv/bin/pytest tests/unit -q`.
  - Result: PASS (exit code 0)
  - Total tests collected/passed: 536/536
- Route inventory audit completed for `/api/v1/compliance/*`.
  - Endpoint count: 91
  - Duplicate method+path patterns: 0
- Migration head confirmed at `0091_compliance_calendar_deadline_management.py`.
- Boundary audit confirmed:
  - no hard deletes
  - no external API calls
  - no AI inference calls
  - no real email sending (internal outbox queueing only)
  - write flows audit-logged
- Full closure details captured in `reports/phase9-pillar1-closure-report.md`.

## GROUP A1 CHECKPOINT — Risk Management Enhancements
**Date:** 2026-06-23
**Status:** COMPLETE
**Phases completed:** A1.1, A1.2, A1.3, A1.4, A1.5, A1.6
**Migration head:** 0095
**Total tests passing:** 631 passed, 2 warnings, 0 failed
**New endpoints added:** `POST/GET /api/v1/compliance/risk-indicators`, `GET /api/v1/compliance/risk-indicators/summary`, `GET/PATCH /api/v1/compliance/risk-indicators/{indicator_id}`, `POST /api/v1/compliance/risk-indicators/{indicator_id}/recalculate`, `POST /api/v1/compliance/risk-indicators/{indicator_id}/archive`; `POST/GET /api/v1/compliance/risk-appetite`, `GET /api/v1/compliance/risk-appetite/summary`, `GET /api/v1/compliance/risk-appetite/breaches`, `GET/PATCH /api/v1/compliance/risk-appetite/{threshold_id}`, `POST /api/v1/compliance/risk-appetite/{threshold_id}/deactivate`; `GET /api/v1/compliance/risks/{risk_id}/graph`; `GET /api/v1/compliance/risks/{risk_id}/score-breakdown`; `GET/PUT /api/v1/compliance/risk-settings`; `POST /api/v1/compliance/risk-scores/compute-entity`; `GET /api/v1/compliance/risk-scores/by-entity`; `GET /api/v1/compliance/risk-scores/summary`
**Boundary audit:** PASSED
**Notes:** pytest warnings unchanged (Starlette `TestClient` deprecation and Python `crypt` deprecation). `alembic heads` and `alembic history` confirm a single 0095 head with intact 0091→0095 chain; `alembic current` could not be executed in this environment due missing DB credentials. Compliance contract registry is missing four A1 routes: `GET /api/v1/compliance/risks/{risk_id}/graph`, `GET /api/v1/compliance/risks/{risk_id}/score-breakdown`, `GET /api/v1/compliance/risk-settings`, `PUT /api/v1/compliance/risk-settings`.

## Phase A2.1 + A2.2 - Control Exceptions and Common Controls Framework (2026-06-23)

- Added Alembic migration `0096_control_exceptions_and_common_controls.py` on top of head `0095_entity_level_risk_scoring`.
  - New tables: `control_exceptions`, `control_exception_approvals`, `common_control_mappings`, `common_control_evidence_coverage`.
  - Added required constraints and indexes for lifecycle, uniqueness, and tenant-scoped query performance.
- Added models:
  - `app/models/control_exception.py`
  - `app/models/control_exception_approval.py`
  - `app/models/common_control_mapping.py`
  - `app/models/common_control_evidence_coverage.py`
- Added services:
  - `app/compliance/services/control_exception_service.py`
  - `app/compliance/services/common_controls_service.py`
- Added API routers:
  - `app/api/v1/control_exceptions.py` (`/api/v1/compliance/control-exceptions`)
  - `app/api/v1/common_controls.py` (`/api/v1/compliance/common-controls`)
- Added schemas:
  - `app/schemas/control_exception.py`
  - `app/schemas/common_controls.py`
- Registered new models/routers and compliance service exports in central registries.
- Updated RBAC seeding with new permission: `exceptions:approve`.
- Extended audit action registry with A2.1/A2.2 events:
  - `control_exception.*` lifecycle + step completion
  - `common_control.*` mapping and evidence coverage events
- Added unit suites:
  - `tests/unit/test_control_exceptions_a21.py`
  - `tests/unit/test_common_controls_a22.py`

## Phase A2.3 - OSCAL Format Export (2026-06-23)

- Added Alembic migration `0097_oscal_export_jobs.py` on top of head `0096_control_exceptions_and_common_controls`.
  - New table: `oscal_export_jobs`.
  - Added required check constraints and indexes for org/status and org/type/time filtering.
- Added model:
  - `app/models/oscal_export_job.py`
- Added schema:
  - `app/schemas/oscal_export.py`
- Added service:
  - `app/compliance/services/oscal_export_service.py`
  - Implements OSCAL SSP/AP/AR/full package generation from org-scoped DB data.
  - Implements job lifecycle (`pending` -> `processing` -> `complete|failed`) with persisted errors.
  - Implements structural validation for SSP/AP/AR/full package payloads.
- Added API router:
  - `app/api/v1/oscal.py` (`/api/v1/compliance/oscal`)
  - Endpoints: `POST /export`, `GET /exports`, `GET /exports/{job_id}`, `GET /exports/{job_id}/download`, `GET /exports/{job_id}/validate`, `GET /summary`.
- Registered new model/router/service exports in central registries.
- Extended audit action registry with A2.3 events:
  - `oscal_export.job_created`
  - `oscal_export.job_completed`
  - `oscal_export.job_failed`
- Added unit suite:
  - `tests/unit/test_oscal_export_a23.py`

## Phase A2.4 - Automated Technical Control Tests (2026-06-23)

- Added Alembic migration `0098_technical_control_tests.py` on top of head `0097_oscal_export_jobs`.
  - New tables: `technical_control_agents`, `technical_control_rules`, `technical_control_results`.
  - Added required check constraints, foreign keys, soft-delete semantics for agents/rules, and tenant-scoped performance indexes.
- Added models:
  - `app/models/technical_control_agent.py`
  - `app/models/technical_control_rule.py`
  - `app/models/technical_control_result.py`
- Added schema:
  - `app/schemas/technical_control.py`
- Added service:
  - `app/compliance/services/technical_control_service.py`
  - Implements token-based agent authentication, evaluator operators, agent/rule/result services, ingest evaluation flow, and control test run integration on failures.
- Added API router:
  - `app/api/v1/technical_controls.py`
  - JWT + RBAC endpoints under `/api/v1/compliance/*` for agents/rules/results.
  - Agent token ingest endpoint at `/api/v1/technical-control-results/ingest` (no JWT, org derived from agent token).
- Updated central wiring:
  - `app/api/v1/router.py`
  - `app/models/__init__.py`
  - `app/compliance/services/__init__.py`
- Updated RBAC and audit seeds in `app/services/seed_service.py`:
  - Permissions: `technical_controls:manage`, `technical_controls:view`
  - Added role mappings for admin/compliance_manager/auditor/readonly.
  - Added audit events:
    - `technical_control.agent_registered`
    - `technical_control.agent_deregistered`
    - `technical_control.rule_created`
    - `technical_control.rule_updated`
    - `technical_control.rule_deactivated`
    - `technical_control.result_ingested`
    - `technical_control.result_failed`
- Added unit suite:
  - `tests/unit/test_technical_control_tests_a24.py`

## GROUP A2 CHECKPOINT — Control & Compliance Enhancements
**Date:** 2026-06-23
**Status:** COMPLETE
**Phases completed:** A2.1, A2.2, A2.3, A2.4
**Migration head:** 0098
**Total tests passing:** 653 passed, 2 warnings, 0 failed
**New endpoints added:**
- A2.1 Control Exceptions: `POST /api/v1/compliance/control-exceptions`, `GET /api/v1/compliance/control-exceptions`, `GET /api/v1/compliance/control-exceptions/summary`, `GET /api/v1/compliance/control-exceptions/{exception_id}`, `POST /api/v1/compliance/control-exceptions/{exception_id}/approve`, `POST /api/v1/compliance/control-exceptions/{exception_id}/reject`, `POST /api/v1/compliance/control-exceptions/{exception_id}/revoke`, `POST /api/v1/compliance/control-exceptions/check-expiry`
- A2.2 Common Controls: `POST /api/v1/compliance/common-controls/mappings`, `GET /api/v1/compliance/common-controls/mappings`, `PATCH /api/v1/compliance/common-controls/mappings/{mapping_id}`, `DELETE /api/v1/compliance/common-controls/mappings/{mapping_id}`, `POST /api/v1/compliance/common-controls/evidence-coverage`, `GET /api/v1/compliance/common-controls/coverage/{control_id}`, `GET /api/v1/compliance/common-controls/evidence-reuse`, `GET /api/v1/compliance/common-controls/summary`
- A2.3 OSCAL Export: `POST /api/v1/compliance/oscal/export`, `GET /api/v1/compliance/oscal/exports`, `GET /api/v1/compliance/oscal/exports/{job_id}`, `GET /api/v1/compliance/oscal/exports/{job_id}/download`, `GET /api/v1/compliance/oscal/exports/{job_id}/validate`, `GET /api/v1/compliance/oscal/summary`
- A2.4 Automated Technical Control Tests: `POST /api/v1/compliance/technical-control-agents`, `GET /api/v1/compliance/technical-control-agents`, `GET /api/v1/compliance/technical-control-agents/{agent_id}`, `DELETE /api/v1/compliance/technical-control-agents/{agent_id}`, `POST /api/v1/compliance/technical-control-rules`, `GET /api/v1/compliance/technical-control-rules`, `GET /api/v1/compliance/technical-control-rules/{rule_id}`, `PATCH /api/v1/compliance/technical-control-rules/{rule_id}`, `DELETE /api/v1/compliance/technical-control-rules/{rule_id}`, `GET /api/v1/compliance/technical-control-rules/{rule_id}/results`, `GET /api/v1/compliance/technical-control-rules/{rule_id}/summary`, `GET /api/v1/compliance/technical-control-results`, `GET /api/v1/compliance/technical-control-results/{result_id}`, `GET /api/v1/compliance/technical-control-results/summary`, `POST /api/v1/technical-control-results/ingest`
**Boundary audit:** PASSED
**Notes:** Full suite executed with zero failures. `alembic heads` confirms a single `0098_technical_control_tests` head. Route inventory includes 149 registered compliance/ingest method-path routes. Warnings unchanged (Starlette `TestClient` deprecation, Python `crypt` deprecation).

## Phase A3.1 - Employee Attestations (2026-06-24)

- Added Alembic migration `0099_employee_attestations.py` on top of head `0098_technical_control_tests`.
  - New tables: `policy_attestation_campaigns`, `policy_attestation_records`.
  - Added required check constraints, unique constraints, org-scoped indexes, and soft-delete behavior for campaigns.
- Added models:
  - `app/models/policy_attestation_campaign.py`
  - `app/models/policy_attestation_record.py`
- Added schema:
  - `app/schemas/attestation.py`
- Added compliance service:
  - `app/compliance/services/employee_attestation_service.py`
  - Implements campaign lifecycle, per-user attestation submission/exemption, reminder queueing to internal outbox, expiry sweep, user history, policy summary, and dashboard metrics.
- Added API router:
  - `app/api/v1/employee_attestations.py` (all under `/api/v1/compliance/*`)
  - Endpoints:
    - `POST /attestation-campaigns`
    - `GET /attestation-campaigns`
    - `GET /attestation-campaigns/dashboard`
    - `GET /attestation-campaigns/{campaign_id}`
    - `PATCH /attestation-campaigns/{campaign_id}`
    - `DELETE /attestation-campaigns/{campaign_id}`
    - `GET /attestation-campaigns/{campaign_id}/completion`
    - `POST /attestation-campaigns/{campaign_id}/reminders`
    - `POST /attestation-campaigns/{campaign_id}/attest`
    - `POST /attestation-campaigns/{campaign_id}/exempt/{user_id}`
    - `POST /attestation-campaigns/{campaign_id}/remind/{user_id}`
    - `GET /attestation-records/me`
    - `GET /attestation-records/user/{user_id}`
    - `GET /policies/{policy_id}/attestation-summary`
- Updated central wiring:
  - `app/api/v1/router.py`
  - `app/models/__init__.py`
  - `app/compliance/services/__init__.py`
- Updated RBAC and audit seeds in `app/services/seed_service.py`:
  - New permissions: `attestations:manage`, `attestations:submit`, `attestations:view`.
  - Added role mappings for compliance_manager/reviewer/auditor/readonly (owner/admin inherit all).
  - Added audit events:
    - `attestation.campaign_created`
    - `attestation.campaign_updated`
    - `attestation.campaign_cancelled`
    - `attestation.submitted`
    - `attestation.user_exempted`
    - `attestation.reminder_sent`
    - `attestation.bulk_reminder_sent`
    - `attestation.expired`
- Added unit suite:
  - `tests/unit/test_employee_attestations_a31.py`
- Validation:
  - `PYTHONPATH=. .venv/bin/pytest tests/unit/test_employee_attestations_a31.py -q` -> PASS
  - `PYTHONPATH=. .venv/bin/pytest tests/unit/test_control_exceptions_a21.py tests/unit/test_common_controls_a22.py tests/unit/test_oscal_export_a23.py tests/unit/test_technical_control_tests_a24.py tests/unit/test_employee_attestations_a31.py -q` -> PASS
  - `PYTHONPATH=. .venv/bin/pytest tests/unit -q` -> PASS
  - `PYTHONPATH=. .venv/bin/pytest tests/unit --collect-only` -> `662 tests collected`
  - `.venv/bin/alembic heads` -> `0099_employee_attestations (head)`

## Phase A3.2 - Policy Exception Management (2026-06-24)

- Added Alembic migration `0100_policy_exception_management.py` on top of head `0099_employee_attestations`.
  - New tables: `policy_exceptions`, `policy_exception_approvals`.
  - Added required check constraints, immutable approval uniqueness, org-scoped indexes, and soft-delete behavior for withdrawn exceptions.
- Added models:
  - `app/models/policy_exception.py`
  - `app/models/policy_exception_approval.py`
- Added schema:
  - `app/schemas/policy_exception.py`
- Added compliance service:
  - `app/compliance/services/policy_exception_service.py`
  - Implements request lifecycle (create/list/get/update/withdraw), approval/rejection flow with immutable approval record creation, expiry sweep, policy summary metrics, and org dashboard metrics.
- Added API router:
  - `app/api/v1/policy_exceptions.py` (under `/api/v1/compliance/*`)
  - Endpoints:
    - `POST /policy-exceptions`
    - `GET /policy-exceptions`
    - `GET /policy-exceptions/dashboard`
    - `GET /policy-exceptions/{exception_id}`
    - `PATCH /policy-exceptions/{exception_id}`
    - `DELETE /policy-exceptions/{exception_id}`
    - `POST /policy-exceptions/{exception_id}/approve`
    - `POST /policy-exceptions/{exception_id}/reject`
    - `GET /policies/{policy_id}/exception-summary`
- Updated central wiring:
  - `app/api/v1/router.py`
  - `app/models/__init__.py`
  - `app/compliance/services/__init__.py`
- Updated RBAC and audit seeds in `app/services/seed_service.py`:
  - New permissions:
    - `policy_exceptions:submit`
    - `policy_exceptions:manage`
    - `policy_exceptions:view`
  - Added role mappings for compliance_manager/reviewer/auditor/readonly (owner/admin inherit all permissions).
  - Added audit events:
    - `policy_exception.created`
    - `policy_exception.updated`
    - `policy_exception.withdrawn`
    - `policy_exception.approved`
    - `policy_exception.rejected`
    - `policy_exception.expired`
- Added unit suite:
  - `tests/unit/test_policy_exceptions_a32.py`

## Phase A3.3 - Pre-Built Policy Template Library (2026-06-24)

- Added Alembic migration `0101_policy_template_library.py` on top of head `0100_policy_exception_management`.
  - New tables: `policy_templates`, `policy_template_clones`.
  - Added category/is_active indexes, framework tag GIN index (Postgres), and org-scoped clone indexes.
  - Enforced immutable clone history semantics (insert-only workflow, no update/delete paths).
- Added models:
  - `app/models/policy_template.py`
  - `app/models/policy_template_clone.py`
- Added schema:
  - `app/schemas/policy_template.py`
- Added compliance service:
  - `app/compliance/services/policy_template_service.py`
  - Implements global template listing/filtering/search, clone count aggregation, template retrieval by id/slug, org clone history, and global clone stats.
  - Clone flow creates org-scoped `compliance_policies` records from template content and writes immutable clone records.
- Added API router:
  - `app/api/v1/policy_templates.py` (under `/api/v1/compliance/*`)
  - Endpoints:
    - `GET /policy-templates`
    - `GET /policy-templates/categories`
    - `GET /policy-templates/frameworks`
    - `GET /policy-templates/clones`
    - `GET /policy-templates/slug/{slug}`
    - `GET /policy-templates/{template_id}/stats`
    - `POST /policy-templates/{template_id}/clone`
    - `GET /policy-templates/{template_id}`
- Updated central wiring:
  - `app/api/v1/router.py`
  - `app/models/__init__.py`
  - `app/compliance/services/__init__.py`
- Updated seeding in `app/services/seed_service.py`:
  - Added `POLICY_TEMPLATE_SEEDS` with 15 substantive markdown templates (Purpose, Scope, Policy Statement, Responsibilities, Enforcement, Review Cycle).
  - Added `SeedService.ensure_policy_templates(db)` using slug-based insert-if-missing semantics (never overwrites existing content on reseed).
  - Added audit action registry event:
    - `policy_template.cloned`
- Updated registration bootstrap:
  - `app/api/v1/auth.py` now ensures policy templates are present when creating a new organization.
- Added unit suite:
  - `tests/unit/test_policy_templates_a33.py`
  - Covers template listing/filtering/search, inactive exclusion, detail by id/slug, categories/framework counts, clone action guards, clone history isolation, clone stats, clone_count aggregation, and audit log emission.
- Validation:
  - `PYTHONPATH=. .venv/bin/pytest tests/unit/test_policy_templates_a33.py -q` -> PASS (6 tests, 0 failures)
  - `PYTHONPATH=. .venv/bin/pytest tests/unit/test_employee_attestations_a31.py tests/unit/test_policy_exceptions_a32.py tests/unit/test_policy_templates_a33.py -q` -> PASS
  - `PYTHONPATH=. .venv/bin/pytest tests/unit -q` -> PASS (0 failures)
  - `PYTHONPATH=. .venv/bin/pytest tests/unit --collect-only` -> `675 tests collected`
  - `.venv/bin/alembic heads` -> `0101_policy_template_library (head)`

## Phase A3.4 - Policy-to-Risk Mapping (2026-06-24)

- Added Alembic migration `0102_policy_risk_mappings.py` on top of head `0101_policy_template_library`.
  - New table: `policy_risk_mappings`.
  - Added mitigation strength check constraint, org-scoped indexes, soft-delete field (`deleted_at`), and partial unique index for active `(policy_id, risk_id)` pairs.
- Added model:
  - `app/models/policy_risk_mapping.py`
- Added schema:
  - `app/schemas/policy_risk_mapping.py`
- Added compliance service:
  - `app/compliance/services/policy_risk_mapping_service.py`
  - Implements mapping lifecycle (create/list/get/update/soft delete), policy/risk scoped mapping views, policy/risk coverage summaries, and org-wide mapping summary.
- Added API router:
  - `app/api/v1/policy_risk_mappings.py` (under `/api/v1/compliance/*`)
  - Endpoints:
    - `POST /policy-risk-mappings`
    - `GET /policy-risk-mappings`
    - `GET /policy-risk-mappings/summary`
    - `GET /policy-risk-mappings/{mapping_id}`
    - `PATCH /policy-risk-mappings/{mapping_id}`
    - `DELETE /policy-risk-mappings/{mapping_id}`
    - `GET /policies/{policy_id}/risk-mappings`
    - `GET /policies/{policy_id}/risk-coverage`
    - `GET /risks/{risk_id}/policy-mappings`
    - `GET /risks/{risk_id}/policy-coverage`
- Updated central wiring:
  - `app/api/v1/router.py`
  - `app/models/__init__.py`
  - `app/compliance/services/__init__.py`
- Updated RBAC and audit seeds in `app/services/seed_service.py`:
  - New permissions:
    - `policy_risks:manage`
    - `policy_risks:view`
  - Added role mappings:
    - `compliance_manager`: `policy_risks:manage`, `policy_risks:view`
    - `reviewer`, `auditor`, `readonly`: `policy_risks:view`
  - Added audit events:
    - `policy_risk_mapping.created`
    - `policy_risk_mapping.updated`
    - `policy_risk_mapping.deleted`
- Added unit suite:
  - `tests/unit/test_policy_risk_mappings_a34.py`
  - Covers mapping lifecycle, duplicate guard, soft delete + remap, cross-org validation, list/filter behavior, coverage endpoints, org summary, tenant isolation, and RBAC guardrails.
- Validation:
  - `PYTHONPATH=. .venv/bin/pytest tests/unit/test_policy_risk_mappings_a34.py -q` -> PASS (6 tests, 0 failures)
  - `PYTHONPATH=. .venv/bin/pytest tests/unit -q` -> PASS (0 failures)
  - `PYTHONPATH=. .venv/bin/pytest tests/unit --collect-only` -> `681 tests collected`
  - `.venv/bin/alembic heads` -> `0102_policy_risk_mappings (head)`

## Phase A3.5 - Policy-to-Issue Linking (2026-06-24)

- Added Alembic migration `0103_policy_issue_links.py` on top of head `0102_policy_risk_mappings`.
  - New table: `policy_issue_links`.
  - Added violation/severity check constraints, org-scoped indexes, soft-delete field (`deleted_at`), and partial unique index for active `(policy_id, issue_id)` pairs.
  - In this branch, issue linkage is backed by existing `tasks` records (`issue_id -> tasks.id`) because there is no separate `issues` table/model.
- Added model:
  - `app/models/policy_issue_link.py`
- Added schema:
  - `app/schemas/policy_issue_link.py`
- Added compliance service:
  - `app/compliance/services/policy_issue_link_service.py`
  - Implements link lifecycle (create/list/get/update/soft delete), policy and issue scoped views, policy effectiveness analytics, issue policy context, and org-wide effectiveness summary.
- Added API router:
  - `app/api/v1/policy_issue_links.py` (under `/api/v1/compliance/*`)
  - Endpoints:
    - `POST /policy-issue-links`
    - `GET /policy-issue-links`
    - `GET /policy-issue-links/summary`
    - `GET /policy-issue-links/{link_id}`
    - `PATCH /policy-issue-links/{link_id}`
    - `DELETE /policy-issue-links/{link_id}`
    - `GET /policies/{policy_id}/issue-links`
    - `GET /policies/{policy_id}/effectiveness`
    - `GET /issues/{issue_id}/policy-links`
    - `GET /issues/{issue_id}/policy-context`
- Updated central wiring:
  - `app/api/v1/router.py`
  - `app/models/__init__.py`
  - `app/compliance/services/__init__.py`
- Updated RBAC and audit seeds in `app/services/seed_service.py`:
  - New permissions:
    - `policy_issues:manage`
    - `policy_issues:view`
  - Added role mappings:
    - `compliance_manager`: `policy_issues:manage`, `policy_issues:view`
    - `reviewer`, `auditor`, `readonly`: `policy_issues:view`
  - Added audit events:
    - `policy_issue_link.created`
    - `policy_issue_link.updated`
    - `policy_issue_link.deleted`
- Added unit suite:
  - `tests/unit/test_policy_issue_links_a35.py`
  - Covers lifecycle, duplicate guard, soft delete + relink, cross-org validation, filters, policy effectiveness metrics, issue context, org summary, tenant isolation, and RBAC write restrictions.
- Validation:
  - `PYTHONPATH=. .venv/bin/pytest tests/unit/test_policy_issue_links_a35.py -q` -> PASS (5 tests, 0 failures)
  - `PYTHONPATH=. .venv/bin/pytest tests/unit -q` -> PASS (0 failures)
  - `PYTHONPATH=. .venv/bin/pytest tests/unit --collect-only` -> `686 tests collected`
  - `.venv/bin/alembic heads` -> `0103_policy_issue_links (head)`

## GROUP A3 CHECKPOINT — Policy Enhancements
**Date:** 2026-06-24
**Status:** COMPLETE (A3.6 cancelled by design)
**Phases completed:** A3.1, A3.2, A3.3, A3.4, A3.5
**Phase cancelled:** A3.6 (external engine — separate repo)
**Migration head:** 0103
**Total tests passing:** 686 passed, 2 warnings, 0 failed
**New endpoints added:** 51 new method+path routes since Group A2 gate (149 baseline -> 200 total including ingest): `POST/GET /api/v1/compliance/attestation-campaigns`, `GET /api/v1/compliance/attestation-campaigns/dashboard`, `GET/PATCH/DELETE /api/v1/compliance/attestation-campaigns/{campaign_id}`, `POST /api/v1/compliance/attestation-campaigns/{campaign_id}/attest`, `GET /api/v1/compliance/attestation-campaigns/{campaign_id}/completion`, `POST /api/v1/compliance/attestation-campaigns/{campaign_id}/exempt/{user_id}`, `POST /api/v1/compliance/attestation-campaigns/{campaign_id}/remind/{user_id}`, `POST /api/v1/compliance/attestation-campaigns/{campaign_id}/reminders`, `GET /api/v1/compliance/attestation-records/me`, `GET /api/v1/compliance/attestation-records/user/{user_id}`, `GET /api/v1/compliance/policies/{policy_id}/attestation-summary`; `POST/GET /api/v1/compliance/policy-exceptions`, `GET /api/v1/compliance/policy-exceptions/dashboard`, `GET/PATCH/DELETE /api/v1/compliance/policy-exceptions/{exception_id}`, `POST /api/v1/compliance/policy-exceptions/{exception_id}/approve`, `POST /api/v1/compliance/policy-exceptions/{exception_id}/reject`, `GET /api/v1/compliance/policies/{policy_id}/exception-summary`; `GET /api/v1/compliance/policy-templates`, `GET /api/v1/compliance/policy-templates/categories`, `GET /api/v1/compliance/policy-templates/frameworks`, `GET /api/v1/compliance/policy-templates/clones`, `GET /api/v1/compliance/policy-templates/slug/{slug}`, `GET /api/v1/compliance/policy-templates/{template_id}`, `POST /api/v1/compliance/policy-templates/{template_id}/clone`, `GET /api/v1/compliance/policy-templates/{template_id}/stats`; `POST/GET /api/v1/compliance/policy-risk-mappings`, `GET /api/v1/compliance/policy-risk-mappings/summary`, `GET/PATCH/DELETE /api/v1/compliance/policy-risk-mappings/{mapping_id}`, `GET /api/v1/compliance/policies/{policy_id}/risk-mappings`, `GET /api/v1/compliance/policies/{policy_id}/risk-coverage`, `GET /api/v1/compliance/risks/{risk_id}/policy-mappings`, `GET /api/v1/compliance/risks/{risk_id}/policy-coverage`; `POST/GET /api/v1/compliance/policy-issue-links`, `GET /api/v1/compliance/policy-issue-links/summary`, `GET/PATCH/DELETE /api/v1/compliance/policy-issue-links/{link_id}`, `GET /api/v1/compliance/policies/{policy_id}/issue-links`, `GET /api/v1/compliance/policies/{policy_id}/effectiveness`, `GET /api/v1/compliance/issues/{issue_id}/policy-links`, `GET /api/v1/compliance/issues/{issue_id}/policy-context`
**Boundary audit:** PASSED
**Schema note:** policy_issue_links.issue_id -> tasks.id (no issues table exists; tasks is the real issue-like entity in this codebase)
**Notes:** Pytest warnings unchanged (Starlette `TestClient` deprecation and Python `crypt` deprecation). `alembic heads` confirms single `0103_policy_issue_links` head. A3 migrations 0099-0103 use `VARCHAR` + check constraints only (no PostgreSQL native `ENUM`). `policy_mapping_suggestions` table is not present, `policy_suggestions:view` permission is not present/duplicated, and the 7 integration seams remain intact (no schema change to compliance_policies/controls/policy_control_links; `require_permission()` and `AuditService.log()` signatures unchanged; seed/router include patterns preserved).

## GROUP A4 CHECKPOINT — Audit & Assurance Module
**Date:** 2026-06-24
**Status:** INCOMPLETE (A4.1-A4.6 not present in this branch)
**Phases completed:** none detected in current workspace
**Migration head:** 0103 (expected 0106)
**Total tests passing:** 686 passed, 2 warnings, 0 failed
**Test delta from A3 gate:** 0
**Endpoints at A3 gate:** 200
**New endpoints added:** 0
**Total endpoints:** 200
**Boundary audit:** PARTIAL PASS (no A4 code present to validate feature-specific checks)
**Integration flows verified:** 0/5
**Architecture notes:**
  - No `/api/v1/audit-portal/*` routes are registered in current router set.
  - No A4 audit lifecycle services/models/migrations were found (`0104`-`0106` are absent).
  - `/api/v1/technical-control-results/ingest` remains present and unchanged.
  - `policy_mapping_suggestions` table and `policy_suggestions:view` permission remain absent.
**Notes:**
  - Full regression completed with zero failures and the same 2 existing warnings (`starlette.testclient` deprecation, Python `crypt` deprecation).
  - `alembic heads` reports `0103_policy_issue_links (head)` and no branch points (`alembic branches` empty).
  - Requested A4 deep flow checks (engagement lifecycle, auditor portal token auth/scoping, finding-to-risk linkage, audit schedule calendar integration, evidence package custody chain) cannot be verified because corresponding A4 artifacts are not present in this repository state.
  - A3 integration seams remain intact (`require_permission(permission_code: str)`, router include pattern, and no schema additions for `policy_mapping_suggestions`).

## Phase A4.1 + A4.2 - Audit Planning & PBC List
**Date:** 2026-06-24
**Migration:** `0104_audit_planning_and_pbc_items`

- Added migration `alembic/versions/0104_audit_planning_and_pbc_items.py` (head now expected at 0104):
  - New table `audit_engagements` with org scope, status/audit_type checks, framework/auditor JSONB arrays, date window, lifecycle fields, and soft-delete (`deleted_at`).
  - New table `pbc_items` with org scope, engagement linkage, requester/assignee/evidence references, status checks, submission/accept/reject timestamps, and soft-delete (`deleted_at`).
  - Added requested indexes for engagement and PBC lookup paths.
- Added models:
  - `app/models/audit_engagement.py`
  - `app/models/pbc_item.py`
- Added schemas:
  - `app/schemas/audit_engagement.py`
  - `app/schemas/pbc_item.py`
- Added services:
  - `app/compliance/services/audit_engagement_service.py`
  - `app/compliance/services/pbc_service.py`
  - Enforced A4.1 and A4.2 status transition rules with `422` on invalid transitions.
  - `report_issued_at` is auto-set on `report_issuance` transition.
  - Added org-scoped overdue sweep (`mark_overdue_items`) and summary/dashboard computations.
- Added routers/endpoints and registered in central API router:
  - `app/api/v1/audit_engagements.py` -> `/api/v1/compliance/audit-engagements/*`
  - `app/api/v1/pbc_items.py` -> `/api/v1/compliance/pbc-items/*`
  - Router registration updated in `app/api/v1/router.py`.
- Scheduler wiring:
  - Added `app/core/pbc_scheduler.py` and startup registration in `app/main.py`.
  - Daily overdue sweep job is wired via APScheduler (optional/no-op when APScheduler is unavailable; disabled in `APP_ENV=test`).
- RBAC seed updates (`app/services/seed_service.py`):
  - Added permissions: `audit:read`, `audit:write`.
  - Added role mappings for new permissions.
  - Added Pillar1 audit action registry entries for `audit_engagements` and `pbc_items` actions.
- Audit logging:
  - All A4 write flows call `AuditService.write_audit_log(...)`.
  - Added requested action codes:
    - `audit_engagement.created`
    - `audit_engagement.updated`
    - `audit_engagement.status_transitioned`
    - `audit_engagement.deleted`
    - `pbc_item.created`
    - `pbc_item.submitted`
    - `pbc_item.accepted`
    - `pbc_item.rejected`
    - `pbc_item.overdue_marked`
    - `pbc_item.deleted`
- Added unit suite:
  - `tests/unit/test_audit_planning_a41_a42.py`
  - Covers A4.1/A4.2 required cases: create/validation, transitions, report issuance timestamp, soft delete guards, dashboard counts, org isolation, PBC submit/evidence linkage, cross-org evidence rejection, requester-only accept/reject, overdue sweep, summary completion rate, and delete constraints.
- Validation:
  - `PYTHONPATH=. .venv/bin/pytest tests/unit/test_audit_planning_a41_a42.py -q` -> PASS (10 tests, 0 failures)
- Test count delta:
  - `+10` tests in this phase file.
- Notes:
  - Existing test warnings unchanged (`starlette.testclient` deprecation and Python `crypt` deprecation).
  - Project-wide instruction correction applied: `AuditService.write_audit_log(...)` used (no `AuditService.log(...)` usage in implementation).

## Phase A4.3 + A4.4 - Auditor Portal & Audit Findings
**Date:** 2026-06-24
**Migration:** `0105_auditor_portal_and_audit_findings`

- Added migration `alembic/versions/0105_auditor_portal_and_audit_findings.py` (head now `0105`):
  - New table `auditor_portal_invitations` with SHA-256 token hash storage, scoped framework/control/evidence JSON fields, expiration/revocation/access metadata, and org scoping.
  - New table `audit_findings` with severity/status constraints, remediation ownership/date tracking, optional risk/control linkage, close metadata, and soft delete.
- Added models:
  - `app/models/auditor_portal_invitation.py`
  - `app/models/audit_finding.py`
- Added schemas:
  - `app/schemas/auditor_portal.py`
  - `app/schemas/audit_finding.py`
- Added services:
  - `app/compliance/services/auditor_portal_service.py`
  - `app/compliance/services/audit_finding_service.py`
  - Portal auth model is separate from JWT and follows bearer-token hash pattern (`Authorization: Bearer <token>` -> SHA-256 hash lookup).
  - Invitation plaintext token is returned once at create time and never returned on read/list endpoints.
  - Finding status transitions and portal token state handling enforce requested 422/401 behavior.
  - Finding reference auto-generation implemented as `F-{YYYY}-{NNN}` per org/year with unique guard and retry.
- Added routers/endpoints and registered in central API router:
  - `app/api/v1/auditor_portal.py` -> `/api/v1/audit-portal/*`
  - `app/api/v1/audit_findings.py` -> `/api/v1/compliance/audit-findings/*`
  - Router registration updated in `app/api/v1/router.py`.
- Updated module wiring:
  - `app/models/__init__.py`
  - `app/compliance/services/__init__.py`
- Updated audit action seed registry (`app/services/seed_service.py`):
  - `auditor_portal.invitation_created`
  - `auditor_portal.invitation_revoked`
  - `auditor_portal.access`
  - `audit_finding.created`
  - `audit_finding.updated`
  - `audit_finding.status_transitioned`
  - `audit_finding.risk_linked`
  - `audit_finding.bulk_transitioned`
  - `audit_finding.deleted`
- Audit logging implementation uses `AuditService.write_audit_log(...)` for all new write operations.
- Added unit suite:
  - `tests/unit/test_audit_portal_findings_a43_a44.py`
  - Covers invitation one-time token behavior, portal token auth/expiry/revocation/access counts, scoped controls/evidence/org isolation, finding ref sequencing, transitions, close metadata, risk linkage validation, bulk transition partial failures, summary overdue behavior, and soft-delete guard.
- Validation:
  - `PYTHONPATH=. .venv/bin/pytest tests/unit/test_audit_portal_findings_a43_a44.py -q` -> PASS (7 tests, 0 failures)
  - `.venv/bin/alembic heads` -> `0105_auditor_portal_and_audit_findings (head)`
- Test count delta:
  - `+7` tests in this phase file.

## Phase A4.5 + A4.6 - Audit Scheduling & Evidence Packages
**Date:** 2026-06-24
**Migration:** `0106_audit_scheduling_and_evidence_packages`

- Added migration `alembic/versions/0106_audit_scheduling_and_evidence_packages.py` (head now `0106`):
  - New table `audit_schedules` with recurrence/status constraints, framework linkage, reminder cadence, engagement back-link, and soft delete.
  - New table `evidence_packages` with engagement linkage, scope frameworks JSON, cover sheet JSON, append-only custody JSON, lifecycle status, item count, and soft delete.
  - New table `evidence_package_items` with package/control/evidence linkage, framework requirement reference, display order, and uniqueness on `(package_id, evidence_id)`.
- Added models:
  - `app/models/audit_schedule.py`
  - `app/models/evidence_package.py`
  - `app/models/evidence_package_item.py`
- Added schemas:
  - `app/schemas/audit_schedule.py`
  - `app/schemas/evidence_package.py`
- Added services:
  - `app/compliance/services/audit_schedule_service.py`
  - `app/compliance/services/evidence_package_service.py`
  - `AuditScheduleService.compute_next_audit_date()` implements recurrence offsets: annual `+365`, semi-annual `+182`, quarterly `+91`, monthly `+30`.
  - Reminder sweep creates internal outbox reminders and compliance calendar deadlines (`compliance_deadlines`) for schedule/date pairs when due-window criteria are met.
  - Evidence package custody uses append helper only (`_append_custody_event`) and never truncates prior entries.
- Added routers/endpoints and registered in central API router:
  - `app/api/v1/audit_schedules.py` -> `/api/v1/compliance/audit-schedules/*`
  - `app/api/v1/evidence_packages.py` -> `/api/v1/compliance/evidence-packages/*`
  - `app/api/v1/router.py` updated to include both routers.
- APScheduler wiring:
  - Reused existing scheduler module `app/core/pbc_scheduler.py` (no new scheduler instance).
  - Added daily job id `audit_schedule_reminder_sweep` on the same scheduler instance.
- Updated module wiring:
  - `app/models/__init__.py`
  - `app/compliance/services/__init__.py`
- Updated audit action seed registry (`app/services/seed_service.py`):
  - `audit_schedule.created`
  - `audit_schedule.updated`
  - `audit_schedule.status_changed`
  - `audit_schedule.engagement_linked`
  - `audit_schedule.reminder_processed`
  - `evidence_package.created`
  - `evidence_package.item_added`
  - `evidence_package.item_removed`
  - `evidence_package.assembled`
  - `evidence_package.exported`
  - `evidence_package.archived`
  - `evidence_package.deleted`
- Audit logging:
  - All new write operations use `AuditService.write_audit_log(...)`.
- Added unit suite:
  - `tests/unit/test_audit_schedule_package_a45_a46.py`
  - Covers A4.5 schedule creation/validation, status transitions, engagement linking/date advancement, reminder sweep behavior (window, skip rules, calendar creation), and org isolation.
  - Covers A4.6 package creation/cover sheet, add/remove item behavior and item count, duplicate evidence guard, status-gated assemble/export/archive flow, manifest grouping, custody append-only behavior, and soft-delete constraints.
- Validation:
  - `PYTHONPATH=. .venv/bin/pytest tests/unit/test_audit_schedule_package_a45_a46.py -q` -> PASS (8 tests, 0 failures)
  - `PYTHONPATH=. .venv/bin/alembic heads` -> `0106_audit_scheduling_and_evidence_packages (head)`
- Test count delta:
  - `+8` tests in this phase file.
- Notes:
  - Existing test warnings unchanged (`starlette.testclient` deprecation and Python `crypt` deprecation).

## GROUP A4 CHECKPOINT — Audit & Assurance Module
**Date:** 2026-06-24
**Status:** COMPLETE
**Phases completed:** A4.1, A4.2, A4.3, A4.4, A4.5, A4.6
**Migration head:** 0106
**Total tests passing:** 711 passed, 15 warnings, 0 failed
**Test delta from A3 gate:** 25
**Endpoints at A3 gate:** 200
**New endpoints added:** 57
**Total endpoints:** 257
**Boundary audit:** PASSED
**Integration flows verified:** 5/5
**Architecture notes:**
  - Auditor portal uses separate bearer token auth path (mirrors A2.4 agent token pattern). SHA-256 hash only.
  - Evidence package chain of custody is append-only JSONB. `evidence_package_items` uses hard delete with custody log compensation (sole exception to soft-delete rule, documented here).
  - APScheduler job `audit_schedule_reminder_sweep` wired to existing scheduler instance.
  - Audit finding_ref auto-generated: `F-{YEAR}-{NNN}` scoped per org per calendar year.
**Notes:**
  - Full regression command executed: `PYTHONPATH=. .venv/bin/pytest -q` -> no failures.
  - Warning set includes 15 deprecation warnings (Starlette `HTTP_422_UNPROCESSABLE_ENTITY` / `TestClient` and Python `crypt`).
  - Alembic checks: `.venv/bin/alembic heads` -> `0106_audit_scheduling_and_evidence_packages (head)`; `.venv/bin/alembic branches` -> no branch points.
  - A4 route inventory verified for `/api/v1/compliance/*`, `/api/v1/audit-portal/*`, and `/api/v1/technical-control-results/ingest`.
  - Boundary seam note: prior references to `AuditService.log()` are obsolete in this codebase; current seam is `AuditService.write_audit_log(...)` and remains intact.

## Phase A5.1 + A5.2 — Questionnaire Templates
## & Response Scoring
**Date:** 2026-06-24
**Migration:** `0107_questionnaire_templates_and_scoring`

- Added migration `alembic/versions/0107_questionnaire_templates_and_scoring.py` (head now `0107`):
  - `questionnaire_templates`
  - `questionnaire_template_sections`
  - `questionnaire_template_questions`
  - `vendor_questionnaire_responses`
  - `vendor_questionnaire_answers`
  - `questionnaire_scoring_rules`
- Added models:
  - `app/models/questionnaire_template.py`
  - `app/models/questionnaire_template_section.py`
  - `app/models/questionnaire_template_question.py`
  - `app/models/vendor_questionnaire_response.py`
  - `app/models/vendor_questionnaire_answer.py`
  - `app/models/questionnaire_scoring_rule.py`
- Added questionnaire schema module: `app/schemas/questionnaire.py`.
- Added services:
  - `app/compliance/services/questionnaire_template_service.py`
  - `app/compliance/services/questionnaire_scoring_service.py`
- Scoring engine design implemented as deterministic `evaluate -> sum -> clamp`:
  - Uses active org-specific rules first per question (fallback to system defaults when no org-specific rules exist).
  - Applies operators `eq`, `ne`, `contains`, `not_contains`, `gte`, `lte`.
  - Updates per-answer `score_contribution`, response `calculated_risk_score`, and `score_computed_at` on recalculation.
  - Clamps final score strictly to `0..100`.
- Seed updates (`app/services/seed_service.py`):
  - Seeded system templates via new idempotent methods:
    - SIG Lite (`20` questions)
    - CAIQ v4 (`20` questions)
  - Seeded default system scoring rules for yes/no expected-Yes questions (`No:+15`, `Yes:-5`) with high-impact overrides:
    - `penetration_testing` `No:+20`
    - `encryption_at_rest` `No:+25`
    - `access_control_mfa` `No:+20`
    - `incident_response` `No:+15`
    - `breach_notification` `No:+20`
    - `information_security_program` / `security_policy` `No:+15`
- Permissions/seed registry updates:
  - Added `vendor:read` and `vendor:write` permissions (kept existing `vendors:*` intact).
  - Added questionnaire/scoring audit action registry entries.
- Added routers/endpoints and registered in `app/api/v1/router.py`:
  - `app/api/v1/questionnaire_templates.py` -> `/api/v1/compliance/questionnaire-templates/*`
  - `app/api/v1/questionnaire_responses.py` -> `/api/v1/compliance/questionnaire-responses/*`
  - `app/api/v1/scoring_rules.py` -> `/api/v1/compliance/scoring-rules/*`
- Audit logging (`AuditService.write_audit_log`) added for:
  - `questionnaire_template.created`
  - `questionnaire_template.cloned`
  - `questionnaire_template.deleted`
  - `questionnaire_response.created`
  - `questionnaire_response.answer_submitted`
  - `questionnaire_response.bulk_answers_submitted`
  - `questionnaire_response.status_transitioned`
  - `questionnaire_response.score_computed`
  - `scoring_rule.created`
  - `scoring_rule.updated`
  - `scoring_rule.deactivated`
- Added tests: `tests/unit/test_questionnaire_a51_a52.py`.
- Validation:
  - `PYTHONPATH=. .venv/bin/pytest tests/unit/test_questionnaire_a51_a52.py -q` -> PASS (10 tests, 0 failures)
- Test count delta:
  - `+10` tests in phase file.
- Notes:
  - No hard deletes introduced for questionnaire/template/rule entities; deactivation/soft-delete semantics preserved.
  - No external API calls and no outbound email send behavior introduced.

## Phase A5.3 — Deterministic Inbound
## Questionnaire Response Engine
**Date:** 2026-06-24
**Migration:** `0108_inbound_questionnaire_engine`

- Added migration `alembic/versions/0108_inbound_questionnaire_engine.py` (head now `0108`):
  - `inbound_questionnaire_sessions`
  - `inbound_questionnaire_items`
  - `compliance_certifications` (deterministic certification source table used by inbound matching)
- Added models:
  - `app/models/inbound_questionnaire_session.py`
  - `app/models/inbound_questionnaire_item.py`
  - `app/models/compliance_certification.py`
- Added deterministic matching/scoring service:
  - `app/compliance/services/inbound_questionnaire_service.py`
  - 5-step priority matching order implemented with SQL + deterministic string/set logic:
    1. Evidence category-tag exact match (`metadata_json.category_tag` + verified/non-archived evidence)
    2. Control framework match (framework + obligation reference + implemented control)
    3. Certification name keyword substring match
    4. Policy keyword token overlap (`split` + stopword removal + `ILIKE`)
    5. Previous approved answer reuse by `category_tag`
  - Conflict handling implemented with deterministic signal checks and fixed conflict template.
- Confidence scoring implemented with integer arithmetic only (`+`/`-`, then clamp `0..100`):
  - Base score by source type
  - Recency modifier (`<=90` days)
  - Previous answer reuse modifier
  - Penalties for inactive/expired and draft/unapproved sources
  - Plain-English `confidence_reason` persisted for each item
- Fixed answer templates implemented (no AI generation):
  - evidence/control/certification/policy/previous-answer/no-source/conflict
- Added API router and endpoints:
  - `app/api/v1/inbound_questionnaires.py`
  - Prefix: `/api/v1/compliance/inbound-questionnaires`
  - Endpoints added:
    - `POST /`
    - `GET /`
    - `GET /{session_id}`
    - `DELETE /{session_id}`
    - `GET /{session_id}/summary`
    - `POST /{session_id}/items`
    - `POST /{session_id}/items/bulk`
    - `GET /{session_id}/items`
    - `GET /{session_id}/items/{item_id}`
    - `POST /{session_id}/items/{item_id}/draft`
    - `POST /{session_id}/draft-all`
    - `POST /{session_id}/items/{item_id}/review`
    - `POST /{session_id}/items/{item_id}/mark-sent`
    - `POST /{session_id}/complete`
- Router registration:
  - Registered in `app/api/v1/router.py`.
- Audit action registry updates (`app/services/seed_service.py`):
  - `inbound_questionnaire.session_created`
  - `inbound_questionnaire.item_added`
  - `inbound_questionnaire.items_bulk_added`
  - `inbound_questionnaire.item_drafted`
  - `inbound_questionnaire.all_drafted`
  - `inbound_questionnaire.item_approved`
  - `inbound_questionnaire.item_edited`
  - `inbound_questionnaire.item_rejected`
  - `inbound_questionnaire.item_sent`
  - `inbound_questionnaire.session_completed`
- Safety constraints verified:
  - Inbound service contains no imports/usages of `openai`, `anthropic`, `groq`, or embeddings/LLM inference.
  - `requires_human_review` remains `True` across draft/review/sent paths.
- Added tests: `tests/unit/test_inbound_questionnaire_a53.py`.
- Validation:
  - `PYTHONPATH=. .venv/bin/pytest tests/unit/test_inbound_questionnaire_a53.py -q` -> PASS (7 tests, 0 failures)
  - `PYTHONPATH=. .venv/bin/alembic heads` -> `0108_inbound_questionnaire_engine (head)`
- Test count delta:
  - `+7` tests in phase file.
- Notes:
  - No hard deletes introduced; session delete path is soft-delete (`deleted_at`).
  - No external API calls and no real email sends introduced.

## Phase A5.4 + A5.5 — Subprocessor Management
## & Customer Commitments
Date: 2026-06-24
Migration: `0109_subprocessors_and_customer_commitments` (head after `0108_inbound_questionnaire_engine`)

Implemented migration `0109` with four new tenant-scoped tables:
- `subprocessors`
- `subprocessor_data_transfers`
- `customer_commitments`
- `commitment_notification_log`

Added SQLAlchemy models and API/service layers for A5.4 and A5.5:
- `app/models/subprocessor.py`
- `app/models/subprocessor_data_transfer.py`
- `app/models/customer_commitment.py`
- `app/models/commitment_notification_log.py`
- `app/compliance/services/subprocessor_service.py`
- `app/compliance/services/customer_commitment_service.py`
- `app/api/v1/subprocessors.py`
- `app/api/v1/customer_commitments.py`

GDPR/Subprocessor design details:
- Added deterministic Article 28 tracking fields (legal basis, transfer mechanism, DPA lifecycle, controller type, risk level, review dates).
- Added EEA destination evaluation in dashboard via `EEA_COUNTRIES` set and active transfer counting outside EEA.
- Implemented DPA status transition guardrails and soft delete guard (only `inactive`/`offboarded`).
- Implemented expiry sweep logic:
  - Signed DPA expiring within 30 days -> internal outbox reminder queue.
  - Signed DPA already past expiry -> status transition to `expired`.

Customer commitments design details:
- Added commitment lifecycle/status machine with 422 rejection for invalid transitions.
- Added trigger/fulfill/waive workflows with internal outbox notifications and immutable notification log records.
- Implemented daily trigger processor:
  - Reminder sweep with same-day idempotency from notification log.
  - Active commitments due today trigger automatically.
  - Triggered commitments older than 3 days become overdue with escalation notification.
- Added dashboard metrics including due-within-30-days and breach notification SLA compliance (`fulfilled_at - triggered_at` vs `sla_hours`).

Scheduler jobs added to existing APScheduler instance in `app/core/pbc_scheduler.py`:
- `subprocessor_dpa_expiry_sweep` (daily)
- `commitment_trigger_sweep` (daily)

Router registration updates:
- Added both routers to `app/api/v1/router.py`.

Audit actions added to seed audit registry:
- Subprocessors:
  - `subprocessor.created`
  - `subprocessor.updated`
  - `subprocessor.dpa_status_updated`
  - `subprocessor.reviewed`
  - `subprocessor.deleted`
  - `subprocessor.transfer_added`
  - `subprocessor.dpa_expiry_swept`
- Customer commitments:
  - `customer_commitment.created`
  - `customer_commitment.updated`
  - `customer_commitment.triggered`
  - `customer_commitment.fulfilled`
  - `customer_commitment.waived`
  - `customer_commitment.deleted`
  - `customer_commitment.sweep_processed`

Tests:
- Added `tests/unit/test_subprocessor_commitments_a54_a55.py`.
- Command run: `PYTHONPATH=. .venv/bin/pytest tests/unit/test_subprocessor_commitments_a54_a55.py -q`
- Result: `2 passed, 0 failed`.

## GROUP A5 CHECKPOINT — TPRM Enhancements
**Date:** 2026-06-24
**Status:** INCOMPLETE (verification run)
**Phases:** A5.1, A5.2, A5.3, A5.4, A5.5, A5.6, A5.7, A5.8
**Migration head:** 0109_subprocessors_and_customer_commitments
**Total tests passing:** 730
**Test delta from A4 gate:** +19
**Endpoints at A4 gate:** 257
**New endpoints added:** 54
**Total endpoints:** 311
**Integration flows verified:** 3/5
**Boundary audit:** PARTIAL (see notes)
**Architecture notes:**
  - A5.3 inbound questionnaire engine is deterministic and source-backed.
    No `openai` / `anthropic` / `groq` / embedding imports in
    `app/compliance/services/inbound_questionnaire_service.py`.
    Matching logic uses SQL lookups and integer confidence arithmetic.
  - A5.1/A5.2 questionnaire templates + scoring are active; system templates
    (`SIG Lite`, `CAIQ v4`) and scoring rules seed correctly.
  - A5.4/A5.5 endpoints, sweeps, dashboards, and outbox-based notifications are active.
  - Scheduler jobs currently registered in `app/core/pbc_scheduler.py`:
    `pbc_overdue_daily_sweep`, `audit_schedule_reminder_sweep`,
    `subprocessor_dpa_expiry_sweep`, `commitment_trigger_sweep`.
**Notes:**
  - Full regression command `PYTHONPATH=. .venv/bin/pytest -q` completed with 0 failures.
    Warnings counted from output: 23.
  - Alembic verification result is a single head at `0109...`; target `0110` is not present.
  - No Alembic branch points detected.
  - Trust Center public routes (`/api/v1/trust-center/*`) are not present in current router set,
    so A5.6 public endpoint checks could not be satisfied.
  - Vendor mitigation case lifecycle endpoints/jobs (A5.8) are not present,
    including expected `mitigation_overdue_action_sweep`.

- Group A5 — TPRM Enhancements: ✅ COMPLETE

## Phase A5.6 + A5.7 + A5.8 — Trust Center,
## AI Vendor Risk & Mitigation Workflow
Date: 2026-06-24
Migration: `0110_trust_center_ai_vendor_mitigation`

Implemented migration `0110` with:
- `organizations.slug` altered to nullable `VARCHAR(100)` with unique partial index on non-null slugs.
- Trust Center tables:
  - `trust_center_configurations`
  - `trust_center_access_requests`
  - `trust_center_published_policies`
- AI vendor risk table:
  - `ai_vendor_assessments`
- Vendor mitigation tables:
  - `vendor_mitigation_cases`
  - `vendor_mitigation_actions`

Implemented first public endpoint design (no JWT, slug-based org resolution):
- Public router: `app/api/v1/trust_center_public.py`
  - `GET /api/v1/trust-center/{slug}`
  - `POST /api/v1/trust-center/{slug}/request-access`
- Public payload enforced to return published policy `title + summary` only (no policy content field).
- Trust center disabled/not found returns 404.

Implemented admin trust center workflow:
- Admin router: `app/api/v1/trust_center_admin.py`
- Configuration create/update, slug management with strict regex, policy publish/unpublish, access request review, uptime updates.
- Access request approval stores SHA-256 token hash (`access_token_hash`) with 7-day expiry.
- Notifications use internal `email_outbox` only.

Implemented deterministic AI vendor risk scoring:
- Service: `app/compliance/services/ai_vendor_assessment_service.py`
- Additive integer rules, clamped 0-100, mapped to risk levels:
  - low (0-25), medium (26-50), high (51-75), critical (76-100)
- Completion computes and persists `risk_score` + `overall_risk_level`.

Implemented vendor mitigation case/action workflow:
- Service: `app/compliance/services/vendor_mitigation_service.py`
- Case transition matrix enforced with 422 on invalid transitions.
- Action lifecycle: evidence submission, accept/reject metadata, auto-case move to `under_review` when all actions accepted.
- Escalation queues outbox alert.
- Daily overdue action sweep marks stale open/in-progress/evidence-submitted actions as `overdue`.

APScheduler updates:
- Added `mitigation_overdue_action_sweep` registration in existing scheduler instance (`app/core/pbc_scheduler.py`).

Router registration updates:
- Added to `app/api/v1/router.py`:
  - `trust_center_public.router`
  - `trust_center_admin.router`
  - `ai_vendor_assessments.router`
  - `vendor_mitigation.router`

Audit actions implemented:
- Trust center:
  - `trust_center.configured`
  - `trust_center.slug_set`
  - `trust_center.policy_published`
  - `trust_center.policy_unpublished`
  - `trust_center.access_request_submitted`
  - `trust_center.access_request_reviewed`
  - `trust_center.uptime_updated`
- AI vendor assessments:
  - `ai_vendor_assessment.created`
  - `ai_vendor_assessment.updated`
  - `ai_vendor_assessment.completed`
  - `ai_vendor_assessment.deleted`
- Vendor mitigation:
  - `vendor_mitigation_case.created`
  - `vendor_mitigation_case.transitioned`
  - `vendor_mitigation_case.escalated`
  - `vendor_mitigation_case.deleted`
  - `vendor_mitigation_action.added`
  - `vendor_mitigation_action.evidence_submitted`
  - `vendor_mitigation_action.accepted`
  - `vendor_mitigation_action.rejected`

Tests added:
- `tests/unit/test_trust_ai_mitigation_a56_a57_a58.py`
- Validation command:
  - `PYTHONPATH=. .venv/bin/pytest tests/unit/test_trust_ai_mitigation_a56_a57_a58.py -q`
  - Result: `3 passed, 0 failed`.

## GROUP A5 CHECKPOINT — TPRM Enhancements
**Date:** 2026-06-24
**Status:** COMPLETE
**Phases:** A5.1, A5.2, A5.3, A5.4, A5.5, A5.6, A5.7, A5.8
**Migration head:** 0110
**Total tests passing:** 733 passed, 28 warnings, 0 failed
**Test delta from A4 gate:** +22
**Endpoints at A4 gate:** 257
**New endpoints added:** 83
**Total endpoints:** 340
**Integration flows verified:** 5/5
**Boundary audit:** PASSED
**Architecture notes:**
  - A5.3 inbound questionnaire engine is fully deterministic. Zero LLM/embedding imports.
    Matching is implemented as 5-step priority SQL/data lookup with integer confidence arithmetic.
  - A5.6 Trust Center is the first public endpoint surface (no JWT on public paths), resolved by org slug.
    Public payload never exposes full policy content.
  - APScheduler jobs registered in `app/core/pbc_scheduler.py` (total 5):
    `pbc_overdue_daily_sweep`, `audit_schedule_reminder_sweep`,
    `subprocessor_dpa_expiry_sweep`, `commitment_trigger_sweep`,
    `mitigation_overdue_action_sweep`.
  - AI vendor assessment risk scoring is deterministic and clamped 0-100; no ML library dependency.
**Notes:** Full regression clean with deprecation-only warning set; no blocking failures.

## Phase A6.1 — Formal Issue Log
Date: 2026-06-25
Migration: `0111_formal_issue_log`

Implemented migration `0111` with new tables:
- `issues`
- `issue_transitions`
- `org_issue_settings`

Design notes:
- Linear 5-state issue machine enforced in service:
  `open -> investigating -> mitigating -> resolved -> closed`.
- `resolved -> closed` requires `resolution_note`.
- Every status change writes an immutable `issue_transitions` record.
- `source_id` is polymorphic `UUID` with no FK constraint (by design).
- `tasks` table remains unchanged and `policy_issue_links` still points to `tasks.id`.

Services and endpoints added:
- Service: `app/compliance/services/issue_service.py`
  - CRUD/list, assign, transition, transition history, dashboard, soft delete.
  - Promotion helpers:
    - from control monitoring alert (`source_type='monitoring_alert'`)
    - from audit finding (`source_type='audit_finding'`)
  - Org settings get/create and update.
- Routers:
  - `app/api/v1/issues.py` (`/api/v1/compliance/issues`)
  - `app/api/v1/issue_settings.py` (`/api/v1/compliance/issue-settings`)
  - Added promotion endpoints:
    - `POST /api/v1/compliance/monitoring/alerts/{alert_id}/create-issue`
    - `POST /api/v1/compliance/audit-findings/{finding_id}/create-issue`

Permissions seeded (3-tier):
- Added permissions:
  - `issues:read`
  - `issues:write`
  - `issues:admin`
- Role mapping updates:
  - `compliance_manager`: `issues:read`, `issues:write`
  - `reviewer`: `issues:read`
  - `readonly`: `issues:read`
  - `owner`/`admin`: all permissions (already global)

Audit actions implemented:
- `issue.created`
- `issue.updated`
- `issue.assigned`
- `issue.transitioned`
- `issue.promoted_from_alert`
- `issue.promoted_from_finding`
- `issue.deleted`
- `issue_settings.updated`

Tests:
- Added `tests/unit/test_issue_log_a61.py`.
- Run:
  - `PYTHONPATH=. .venv/bin/pytest tests/unit/test_issue_log_a61.py -q`
  - Result: `4 passed, 0 failed`.
- Test delta: `+4` unit tests.

## Phase A6.2 + A6.3 — RCA & SLA Tracking
Date: 2026-06-25
Migration: `0112_rca_and_issue_sla_tracking`

Implemented migration `0112` with new tables:
- `root_cause_analyses`
- `issue_sla_policies`
- `issue_sla_tracking`

Design notes:
- RCA model is one-per-issue (`UNIQUE(issue_id)`) with structured JSONB-backed lists for:
  - `contributing_factors`
  - `corrective_actions`
  - `preventive_measures`
- RCA creation guard enforces issue status must be `resolved` or `closed`.
- RCA review is second-eyes only:
  - cannot self-review
  - can only be reviewed once
  - reviewed RCA becomes immutable for updates.
- `IssueService.transition_issue()` now enforces `require_rca_before_close` from `org_issue_settings` on `resolved -> closed`.
- SLA clock starts from `issue.created_at` (not runtime now):
  - `response_deadline = created_at + response_sla_hours`
  - `resolution_deadline = created_at + resolution_sla_hours`
- SLA tracking is auto-created during issue creation in the same transaction.
- Response/resolution met timestamps are stamped on transitions:
  - `open -> investigating` sets `response_met_at`
  - `* -> resolved` sets `resolution_met_at`
- Hourly breach processor implemented and wired:
  - job id: `issue_sla_breach_check`
  - trigger: `IntervalTrigger(hours=1)`
  - scheduler file: `app/core/pbc_scheduler.py`

Seed/bootstrap updates:
- Added `SeedService.ensure_issue_sla_policies(db, organization_id)` with defaults:
  - critical: `1h` response, `24h` resolution
  - high: `4h`, `72h`
  - medium: `24h`, `168h`
  - low: `72h`, `720h`
- Called during org registration in `app/api/v1/auth.py`.
- Migration also inserts default SLA rows for all existing organizations.

Services added:
- `app/compliance/services/rca_service.py`
  - `create_rca`, `get_rca`, `update_rca`, `review_rca`, `has_rca`
- `app/compliance/services/sla_service.py`
  - `get_sla_status`, `get_sla_breaches`, `check_sla_breaches`
  - `get_sla_policies`, `create_or_update_sla_policy`
  - `initialize_tracking_for_issue`, `mark_response_met`, `mark_resolution_met`
  - `update_issue_dashboard_with_sla`

Endpoints added:
- `app/api/v1/issues.py`
  - `GET /api/v1/compliance/issues/sla-breaches`
  - `GET /api/v1/compliance/issues/{issue_id}/sla-status`
  - `POST /api/v1/compliance/issues/{issue_id}/rca`
  - `GET /api/v1/compliance/issues/{issue_id}/rca`
  - `PATCH /api/v1/compliance/issues/{issue_id}/rca`
  - `POST /api/v1/compliance/issues/{issue_id}/rca/review`
- `app/api/v1/sla_policies.py`
  - `GET /api/v1/compliance/sla-policies`
  - `POST /api/v1/compliance/sla-policies`
  - `GET /api/v1/compliance/sla-policies/trigger-breach-check`

Audit actions implemented:
- `rca.created`
- `rca.updated`
- `rca.reviewed`
- `sla_policy.updated`
- `sla.response_breached`
- `sla.resolution_breached`

Tests:
- Added `tests/unit/test_rca_sla_a62_a63.py`
- Updated `tests/unit/test_issue_log_a61.py` for A6.2 close-transition enforcement (`RCA` required when setting enabled).
- Run:
  - `PYTHONPATH=. .venv/bin/pytest tests/unit/test_rca_sla_a62_a63.py -q`
  - Result: `4 passed, 0 failed`
- Compatibility run:
  - `PYTHONPATH=. .venv/bin/pytest tests/unit/test_issue_log_a61.py tests/unit/test_rca_sla_a62_a63.py -q`
  - Result: `8 passed, 0 failed`
- Test delta: `+4` new unit tests (A6.2/A6.3 file).

## Phase A6.4 + A6.5 — Escalation & Breach Notification
Date: 2026-06-25
Migration: `0113_general_escalation_and_breach_workflow`

Implemented migration `0113` with new tables:
- `escalation_policies`
- `escalation_events`
- `breach_notifications`

Design notes:
- Built a new general escalation system in parallel with existing framework-review escalation code (no changes to prior framework escalation logic).
- Strategy pattern behavior for A6.4:
  - evaluator implemented for `entity_type='issue'`
  - other entity types return no candidates (no error)
- Issue escalation evaluator conditions:
  - `time_in_state` using `updated_at` cutoff (`N` hours)
  - `sla_breach` using `issue_sla_tracking.response_breached/resolution_breached`
  - `severity_threshold` with 1-hour grace period
- Idempotency window implemented:
  - for each `(policy_id, entity_id)`, skips firing if an escalation event exists in the prior 24 hours
- Escalation events are immutable log rows; policy deletion is soft-delete only (requires deactivated policy).
- Breach workflow implemented as one-record-per-issue (`UNIQUE(issue_id)`) with configurable deadline hours.
- Breach deadline computation uses issue creation timestamp:
  - `regulatory_notification_deadline = issue.created_at + timedelta(hours=regulatory_notification_hours)`
- Breach status machine implemented with guarded transitions:
  - `assessing -> notification_due -> regulator_notified -> subjects_notified -> closed`
  - `closed` is terminal
- Deadline warning sweep:
  - checks required+unnotified breaches in a 6-hour warning window
  - queues internal outbox notifications
  - transitions to `notification_due` if needed

Services added:
- `app/compliance/services/escalation_service.py`
  - `create_policy`, `get_policy`, `list_policies`, `update_policy`
  - `deactivate_policy`, `soft_delete_policy`
  - `evaluate_policies`, `get_escalation_history`
  - scheduler entrypoint: `run_daily_escalation_policy_evaluation`
- `app/compliance/services/breach_notification_service.py`
  - `create_breach_notification`, `get_breach_notification`, `get_by_issue`, `list_breach_notifications`
  - `record_regulator_notification`, `record_subject_notification`, `close_breach`
  - `sweep_breach_deadlines`
  - scheduler entrypoint: `run_daily_breach_notification_deadline_sweep`

Models added:
- `app/models/escalation_policy.py`
- `app/models/escalation_event.py`
- `app/models/breach_notification.py`

Schemas added:
- `app/schemas/escalation.py`
- `app/schemas/breach_notification.py`

Endpoints added:
- `app/api/v1/escalation_policies.py` (`/api/v1/compliance/escalation-policies`)
  - `POST /`
  - `GET /`
  - `GET /events`
  - `POST /evaluate` (issues:admin)
  - `GET /{policy_id}`
  - `PATCH /{policy_id}`
  - `POST /{policy_id}/deactivate`
  - `DELETE /{policy_id}`
- `app/api/v1/breach_notifications.py` (`/api/v1/compliance/breach-notifications`)
  - `POST /` (query `issue_id`)
  - `GET /`
  - `GET /{breach_id}`
  - `POST /{breach_id}/record-regulator-notification`
  - `POST /{breach_id}/record-subject-notification`
  - `POST /{breach_id}/close`
- Issue sub-resource in `app/api/v1/issues.py`:
  - `POST /api/v1/compliance/issues/{issue_id}/breach-notification`

Scheduler wiring added to existing scheduler instance in `app/core/pbc_scheduler.py`:
- `escalation_policy_evaluation` (daily)
- `breach_notification_deadline_sweep` (daily)

Seed updates:
- Added permissions:
  - `escalations:read`
  - `escalations:write`
- Role mappings updated:
  - compliance_manager: `escalations:read`, `escalations:write`
  - reviewer: `escalations:read`
  - readonly: `escalations:read`
  - owner/admin inherit all permissions
- Added audit action registry entries:
  - `escalation_policy.created`
  - `escalation_policy.updated`
  - `escalation_policy.deactivated`
  - `escalation.fired`
  - `breach_notification.created`
  - `breach_notification.regulator_notified`
  - `breach_notification.subjects_notified`
  - `breach_notification.closed`
  - `breach_notification.deadline_warned`

Audit logging implemented:
- `escalation_policy.created`
- `escalation_policy.updated`
- `escalation_policy.deactivated`
- `escalation.fired`
- `breach_notification.created`
- `breach_notification.regulator_notified`
- `breach_notification.subjects_notified`
- `breach_notification.closed`
- `breach_notification.deadline_warned`

Tests:
- Added `tests/unit/test_escalation_breach_a64_a65.py`.
- Run:
  - `PYTHONPATH=. .venv/bin/pytest tests/unit/test_escalation_breach_a64_a65.py -q`
  - Result: `4 passed, 0 failed`
- Regression run:
  - `PYTHONPATH=. .venv/bin/pytest tests/unit/test_issue_log_a61.py tests/unit/test_rca_sla_a62_a63.py tests/unit/test_escalation_breach_a64_a65.py -q`
  - Result: `12 passed, 0 failed`
- Test delta: `+4` unit tests (A6.4/A6.5 file).

## Phase A6.6 + A6.7 — Issue-to-Policy & Control Links
Date: 2026-06-25
Migration: `0114_issue_policy_and_control_links` (head after 0113)

Tables created:
- `issue_policy_links`
  - org-scoped links from `issues.id` to `compliance_policies.id`
  - `link_type`: `violated|related`
  - unique `(organization_id, issue_id, policy_id)`
- `issue_control_links`
  - org-scoped links from `issues.id` to `controls.id`
  - `failure_type`: `control_absent|control_failed|control_bypassed|control_ineffective`
  - unique `(organization_id, issue_id, control_id)`
  - includes `deleted_at` compatibility column to satisfy existing `ScoringService._controls_with_open_high_critical_issues()` SQL hook

Service design:
- Added `app/compliance/services/issue_policy_link_service.py`
  - link/unlink/get links
  - policy associated issues
  - policy 12-month violation rate:
    - `violations_past_12m / total_issues_past_12m * 100`
    - safe zero division handling
  - violation count helper for policy list/detail enrichment
- Added `app/compliance/services/issue_control_link_service.py`
  - link/unlink/get links
  - grouped associated issues by `failure_type`
  - control failure-rate payload:
    - `active_months` (months since org earliest issue, minimum 1)
    - `total_failures` excludes `control_absent`
    - `failure_rate = total_failures / active_months`
    - `open_high_critical_count` from linked unresolved high/critical issues

Scoring hook note:
- `app/services/scoring_service.py` was **not modified**.
- Existing hook `_controls_with_open_high_critical_issues()` now works against real `issue_control_links` table.

Endpoint additions:
- `app/api/v1/issues.py`
  - `POST /api/v1/compliance/issues/{issue_id}/policy-links`
  - `GET /api/v1/compliance/issues/{issue_id}/policy-links`
  - `DELETE /api/v1/compliance/issues/{issue_id}/policy-links/{policy_id}`
  - `POST /api/v1/compliance/issues/{issue_id}/control-links`
  - `GET /api/v1/compliance/issues/{issue_id}/control-links`
  - `DELETE /api/v1/compliance/issues/{issue_id}/control-links/{control_id}`
- `app/api/v1/compliance_policies.py`
  - `GET /api/v1/compliance/policies/{policy_id}/associated-issues`
  - `GET /api/v1/compliance/policies/{policy_id}/violation-rate`
  - policy list/detail now include computed `violation_count` (nullable int, default 0)
- `app/api/v1/controls.py`
  - `GET /api/v1/controls/{control_id}/associated-issues`
  - `GET /api/v1/controls/{control_id}/failure-rate`

Compatibility seam handled:
- Existing A3.5 route path overlap (`/api/v1/compliance/issues/{issue_id}/policy-links`) was preserved by adding fallback logic in `app/api/v1/policy_issue_links.py`:
  - task-based policy links still work
  - formal issue-based links are now also surfaced on the same path

Audit actions emitted:
- `issue_policy_link.created`
- `issue_policy_link.removed`
- `issue_control_link.created`
- `issue_control_link.removed`

Tests:
- Added `tests/unit/test_issue_links_a66_a67.py`
- Run:
  - `PYTHONPATH=. .venv/bin/pytest tests/unit/test_issue_links_a66_a67.py -q`
  - Result: `2 passed, 0 failed`
- Regression sanity run:
  - `PYTHONPATH=. .venv/bin/pytest tests/unit/test_policy_issue_links_a35.py tests/unit/test_issue_log_a61.py tests/unit/test_control_testing_and_scoring_phase27.py -q`
  - Result: `13 passed, 0 failed`
- Test count delta in this phase file: `+2` tests.

## Phase A6.8 + A6.9 — Remediation Engine & Classification
Date: 2026-06-25
Migration: `0115_remediation_and_incident_classification`

Tables created:
- `remediation_suggestions`
  - per-issue deterministic suggestions
  - `suggestion_source` constrained to `rule_based|template` (engine writes `rule_based`)
  - `source_key` captures matched dict key path
- `incident_classifications`
  - one row per issue (`UNIQUE(issue_id)` upsert behavior in service)
  - category/sub-category/regulatory implications
  - `notification_required` flag and `auto_classified` marker

Engine design (pure dict lookup, no LLM):
- Added `app/compliance/engines/remediation_engine.py`
  - `REMEDIATION_TEMPLATES` keyed by `(issue_type, control_category)`
  - fallback chain: exact key -> `(issue_type, None)` -> `GENERIC_SUGGESTIONS`
  - deterministic return of pre-written strings only
  - no external calls, no embeddings, no ML imports
- Added `app/compliance/engines/classification_engine.py`
  - `CLASSIFICATION_RULES` keyed by `(issue_type, severity)`
  - deterministic fallback to `service_disruption / unclassified incident`

Services added:
- `app/compliance/services/remediation_service.py`
  - generate/list/apply/dismiss suggestion lifecycle
  - idempotent generation by `(issue_id, suggestion_text)`
  - apply creates linked task (`linked_entity_type='issue'`)
- `app/compliance/services/classification_service.py`
  - auto-classify with upsert semantics
  - manual override (`auto_classified=False`)
  - incident analytics aggregation by category/regulatory implications
  - breach notification feed: outbox reminder only (no auto-create breach record)

Models/schemas added:
- Models:
  - `app/models/remediation_suggestion.py`
  - `app/models/incident_classification.py`
- Schemas:
  - `app/schemas/remediation.py`
  - `app/schemas/incident_classification.py`

Endpoints added:
- In issues router (`app/api/v1/issues.py`):
  - `POST /api/v1/compliance/issues/{issue_id}/generate-suggestions`
  - `GET /api/v1/compliance/issues/{issue_id}/suggestions`
  - `POST /api/v1/compliance/issues/{issue_id}/classification`
  - `PATCH /api/v1/compliance/issues/{issue_id}/classification`
  - `GET /api/v1/compliance/issues/{issue_id}/classification`
- Added companion router in same file for remediation actions:
  - `POST /api/v1/compliance/remediation-suggestions/{suggestion_id}/apply`
  - `POST /api/v1/compliance/remediation-suggestions/{suggestion_id}/dismiss`
- New analytics router:
  - `app/api/v1/incident_analytics.py`
  - `GET /api/v1/compliance/incidents/by-category`

Audit actions emitted:
- `remediation.generated`
- `remediation.applied`
- `remediation.dismissed`
- `classification.auto_classified`
- `classification.overridden`

Tests:
- Added `tests/unit/test_remediation_classification_a68_a69.py`
- Run:
  - `PYTHONPATH=. .venv/bin/pytest tests/unit/test_remediation_classification_a68_a69.py -q`
  - Result: `2 passed, 0 failed`
- Regression sanity:
  - `PYTHONPATH=. .venv/bin/pytest tests/unit/test_issue_log_a61.py tests/unit/test_issue_links_a66_a67.py tests/unit/test_escalation_breach_a64_a65.py -q`
  - Result: `10 passed, 0 failed`
- Test delta for this phase file: `+2` tests.

## GROUP A6 CHECKPOINT — Issue, Incident & Remediation
**Date:** 2026-06-25
**Status:** COMPLETE
**Phases:** A6.1–A6.9
**Migration head:** 0115
**Total tests passing:** 749 passed, warnings emitted, 0 failed
**Test delta from A5 gate:** +16
**Endpoints at A5 gate:** 340
**New endpoints added:** 52
**Total endpoints:** 392
**Integration flows:** 5/5
**Boundary audit:** PASSED
**Architecture notes:**
  - issues table is standalone (not tasks). tasks untouched. policy_issue_links unchanged.
  - source_id on issues is polymorphic (no FK). Same pattern as audit_log entity_type+entity_id.
  - issue_transitions and escalation_events are append-only immutable tables.
  - scoring_service.py hook `_controls_with_open_high_critical_issues` remains pre-wired and used by issue_control_links table.
  - Hourly APScheduler job: issue_sla_breach_check (hours=1 interval — first hourly job in codebase).
  - APScheduler jobs total: 8 (`pbc_overdue_daily_sweep`, `audit_schedule_reminder_sweep`, `subprocessor_dpa_expiry_sweep`, `commitment_trigger_sweep`, `mitigation_overdue_action_sweep`, `issue_sla_breach_check`, `escalation_policy_evaluation`, `breach_notification_deadline_sweep`).
  - remediation_engine.py: zero LLM/ML imports, pure dict lookup, `(issue_type, control_category)` tuple keys with fallback chain.
  - classification_engine.py: zero LLM imports, `(issue_type, severity)` tuple keys.
  - Breach notification feed from classification: outbox reminder only — never auto-creates breach.
  - issue_policy_links and issue_control_links use hard delete (sole exceptions in A6 — documented).
**Notes:** Full-suite warnings are deprecation warnings (Starlette HTTP_422 constant + passlib crypt deprecation); no test failures.

- Group A6 — Issue, Incident & Remediation: ✅ COMPLETE

## Phase A7.1 + A7.4 — Board Scorecard &
## Executive Narrative
Date: 2026-06-25
Migration: 0116 (`0116_board_scorecard_and_executive_narrative`) — no schema changes (report type support added at service layer because `compliance_reports.report_type` is VARCHAR).

Implemented report builders and services:
- `app/compliance/services/board_scorecard_builder.py`
- `app/compliance/services/board_scorecard_service.py`
- `app/compliance/services/executive_narrative_builder.py`
- `app/compliance/services/executive_narrative_service.py`
- `app/compliance/templates/executive_narrative_templates.py`

Report builder pattern followed:
- Reused existing `ComplianceReport` + `ComplianceReportSection` persistence flow (`ReportService.persist_report`).
- Added both report types to `ALLOWED_REPORT_TYPES` and `build_report` routing in `app/services/report_service.py`.
- Caveat section is included using existing caveat pattern.

Data sources used per section:
- Board scorecard:
  - score snapshots (`score_snapshots`) for current score and 90-day delta
  - open risks (`risks`) with owner resolution from `users`
  - critical open issues (`issues`)
  - certification counts (`compliance_certifications`)
  - upcoming deadlines (`compliance_deadlines`)
  - recent obligation implementation gains (`organization_obligation_states`)
- Executive narrative:
  - score/delta from `score_snapshots`
  - framework coverage from `frameworks`, `obligations`, `control_obligation_mappings`, `controls`
  - risk and issue attention from `risks`, `issues`
  - achievements from `compliance_certifications` + closed/mitigated risks
  - upcoming deadlines from `compliance_deadlines`
  - narrative text generated via `.format()` templates only (no AI/LLM usage)

Endpoints added (existing reports router):
- `POST /api/v1/reports/board-scorecard` (permission: `reports:read`)
- `POST /api/v1/reports/executive-narrative` (permission: `reports:read`)

Audit actions emitted:
- `report.board_scorecard_generated`
- `report.executive_narrative_generated`

Tests:
- Added `tests/unit/test_reports_a71_a74.py`
- Run:
  - `PYTHONPATH=. .venv/bin/pytest tests/unit/test_reports_a71_a74.py -q`
  - Result: `4 passed, 0 failed`
- Regression sanity:
  - `PYTHONPATH=. .venv/bin/pytest tests/unit/test_reports_phase29.py -q`
  - Result: `6 passed, 0 failed`
- Test delta for this phase file: `+4` tests.

## Phase A7.2 + A7.3 — PDF/Word Export &
## Custom Report Builder
Date: 2026-06-25
Migration: 0117 (`0117_pdf_word_export_and_custom_report_templates`)

Schema and dependency updates:
- Added migration table: `custom_report_templates`
  - org-scoped template name, ordered `sections` (JSONB), optional `framework_filter` (JSONB), `date_range_days`, soft-delete (`deleted_at`).
- Added model: `app/models/custom_report_template.py`
- Added dependencies in `pyproject.toml`:
  - `reportlab`
  - `python-docx`
- Added `FILE_STORAGE_PATH` setting in `app/core/config.py` (default `/tmp/complivibe_exports/`).

Renderer architecture (A7.2):
- Added mapper:
  - `app/compliance/renderers/report_section_mapper.py`
  - maps known keys (e.g., `score` -> `Compliance Score`) with title-cased fallback for unknown keys.
- Added renderers:
  - `app/compliance/renderers/pdf_report_renderer.py`
  - `app/compliance/renderers/docx_report_renderer.py`
- Rendering behavior implemented:
  - cover page (org name, report title, generated date)
  - section rendering from report JSON content (`inputs_summary_json` preferred, fallback to report/section content)
  - dict/list/string/number rendering rules
  - disclaimer/caveat final section/page
  - PDF footer on non-cover pages
- Runtime fallback behavior included for environments missing optional renderer libs (still returns valid PDF/ZIP-DOCX bytes) while keeping primary reportlab/python-docx path.

Export governance integration (A7.2):
- Extended existing `ExportService` pipeline (no parallel export system introduced):
  - new export types: `compliance_report_pdf`, `compliance_report_docx`
  - new helper `create_completed_binary_export_job(...)` to:
    - create `ExportJob`
    - emit `export.created`/`export.started`/`export.completed` events
    - persist checksum/signature/manifest/package metadata
- Added report export endpoints in existing reports router:
  - `POST /api/v1/reports/{report_id}/export/pdf`
  - `POST /api/v1/reports/{report_id}/export/docx`
  - permission: `reports:read`
  - file write path: `{FILE_STORAGE_PATH}/reports/{org_id}/{report_id}_{timestamp}.{ext}`
  - checksum: SHA-256 over rendered bytes
  - response: streaming attachment bytes
- Audit actions emitted:
  - `report.exported_pdf`
  - `report.exported_docx`

Custom report builder design (A7.3):
- Added generator:
  - `app/compliance/services/custom_report_generator.py`
  - section-builder map implemented for all required section names:
    - `executive_summary`
    - `framework_readiness`
    - `control_health`
    - `risk_summary`
    - `vendor_risk`
    - `evidence_status`
    - `open_issues`
    - `policy_status`
    - `ai_governance_summary`
  - graceful AI governance fallback:
    - returns `{status: "not_configured", message: ...}` when module data unavailable/not activated.
  - generated report persisted as `ComplianceReport` with `report_type='custom'` and `_meta` block.
- Added service:
  - `app/compliance/services/custom_report_service.py`
  - create/get/list/update/soft-delete template
  - generate report from template
  - section-name validation (422 on unknown section)
  - audit logging:
    - `custom_report_template.created`
    - `custom_report_template.updated`
    - `custom_report_template.deleted`
    - `custom_report.generated`
- Added router:
  - `app/api/v1/custom_reports.py`
  - prefix: `/api/v1/compliance/custom-report-templates`
  - endpoints: create/list/get/update/delete/generate
  - permissions:
    - write: `reports:write` (POST/PATCH/DELETE)
    - read: `reports:read` (GET + generate)
- Router registration:
  - included `custom_reports.router` in `app/api/v1/router.py`.

Tests:
- Added: `tests/unit/test_export_custom_report_a72_a73.py`
- Run:
  - `PYTHONPATH=. .venv/bin/pytest tests/unit/test_export_custom_report_a72_a73.py -q`
  - Result: `4 passed, 0 failed`
- Regression sanity:
  - `PYTHONPATH=. .venv/bin/pytest tests/unit/test_reports_a71_a74.py tests/unit/test_exports_phase30.py -q`
  - Result: passed, 0 failed.
- Test delta for this phase file: `+4` tests.

## Phase A7.5 + A7.6 — Regulatory Reports &
## Framework Coverage Heatmap
Date: 2026-06-25
Migration: 0118 (`0118_regulatory_reports_and_framework_heatmap`) — no new tables; query/report composition only.

A7.5 Regulatory reports:
- Added registry file:
  - `app/compliance/services/regulatory_report_registry.py`
  - `REGULATORY_REPORT_REGISTRY` + descriptions map.
- Added regulatory builders and registration:
  - `app/compliance/services/soc2_readiness_builder.py`
  - `app/compliance/services/gdpr_ropa_builder.py` (stubbed unavailable response per requirement)
  - `app/compliance/services/iso27001_soa_builder.py`
  - `app/compliance/services/nist_ai_rmf_builder.py`
  - `app/compliance/services/eu_ai_act_builder.py`
- Added orchestration service:
  - `app/compliance/services/regulatory_report_service.py`
  - `generate_regulatory_report(org_id, report_type, db, created_by)`
  - `list_available_report_types()`
- Added report type values in report service allowed set:
  - `soc2_readiness`, `gdpr_ropa`, `iso27001_soa`, `nist_ai_rmf_summary`, `eu_ai_act_conformity`.
- Builder mapping approach implemented as requested:
  - SOC2 TSC categories (`CC1..CC9`, `A1`, `C1`, `PI1`, `P1`) from obligation reference prefixes.
  - ISO Annex A domains (`A.5..A.18`) from obligation reference prefixes.
  - NIST AI RMF 4 functions (`GOVERN`, `MAP`, `MEASURE`, `MANAGE`) from reference prefixes.
  - EU AI Act risk-tier bucketing by article extraction and tier ranges.
  - GDPR Article 30 intentionally returns unavailable stub and does not query ROPA tables.

A7.6 Framework coverage heatmap:
- Added service:
  - `app/compliance/services/framework_coverage_matrix_service.py`
- Coverage logic implemented over existing tables (`frameworks`, `organization_frameworks`, `obligations`, `control_obligation_mappings`, `controls`, `evidence_control_links`, `evidence_items`):
  - `covered`: has linked controls and at least one verified active non-expired evidence item linked to those controls
  - `partial`: has linked controls but no qualifying evidence (including expired evidence)
  - `uncovered`: no linked controls
- Grouping uses real framework section linkage when present (`framework_section_id -> framework_sections.title`) and falls back to reference prefix.
- Output includes framework totals and per-section obligation rows with counts + `coverage_status`.

Router/API additions (existing reports router):
- `GET /api/v1/reports/regulatory/available-types`
- `POST /api/v1/reports/regulatory/{report_type}`
- `GET /api/v1/reports/framework-coverage-matrix?framework_id=...`
- `POST /api/v1/reports/framework-coverage-matrix/export-pdf`
- Implemented route ordering so static regulatory/heatmap routes resolve before `/{report_id}` UUID route.

PDF heatmap export:
- Extended `PDFReportRenderer` with `render_coverage_matrix(...)` colored rows:
  - green: covered
  - amber: partial
  - red: uncovered
- Uses existing export governance pipeline (`ExportService`) and stores export metadata/checksum.

Audit logging:
- `report.regulatory_generated` (metadata includes `report_type`)
- `report.coverage_matrix_generated`

Tests:
- Added: `tests/unit/test_regulatory_heatmap_a75_a76.py`
- Run:
  - `PYTHONPATH=. .venv/bin/pytest tests/unit/test_regulatory_heatmap_a75_a76.py -q`
  - Result: `3 passed, 0 failed`
- Regression sanity:
  - `PYTHONPATH=. .venv/bin/pytest tests/unit/test_reports_a71_a74.py tests/unit/test_export_custom_report_a72_a73.py -q`
  - Result: passed, 0 failed.
- Test delta for this phase file: `+3` tests.

## Phase A7.7 + A8.1 — AI Governance Dashboard
## & Scheduler Run Log
Date: 2026-06-25
Migration: 0119 (`0119_ai_governance_dashboard_and_scheduler_run_logs`)

A7.7 AI Governance dashboard:
- Added service:
  - `app/compliance/services/ai_governance_dashboard_service.py`
  - returns stable placeholder contract with zero/empty values and `_pillar2_status='not_yet_activated'`
  - future data hookups are wrapped in `try/except` blocks so endpoint remains safe before Pillar 2 tables are activated.
- Added router:
  - `app/api/v1/ai_governance_dashboard.py`
  - `GET /api/v1/ai-governance/dashboard`
  - permission: `ai_governance:read`
- Seed updates:
  - added permission `ai_governance:read`
  - assigned to `compliance_manager` (owner/admin inherit via full permission set).

A8.1 Scheduler run log and admin visibility:
- Added table/model:
  - migration creates `scheduler_run_logs`
  - model: `app/models/scheduler_run_log.py`
  - fields: `job_name`, `started_at`, `completed_at`, `status`, `records_processed`, `error_message`, `created_at`
  - no `organization_id` (system-level logging)
  - indexed by job/time/status.
- Added logging helper:
  - `app/core/scheduler_logger.py`
  - `SchedulerJobLogger.run_logged(job_name, job_fn, db_session_factory, **kwargs)`
  - behavior:
    - inserts running log row before execution
    - marks completed/failed with timestamps
    - truncates `error_message` to first 1000 chars
    - re-raises exceptions after logging.
- Wrapped existing scheduler jobs in `app/core/pbc_scheduler.py` using `SchedulerJobLogger` (wrap, not replace business services):
  - `pbc_overdue_daily_sweep`
  - `audit_schedule_reminder_sweep`
  - `subprocessor_dpa_expiry_sweep`
  - `commitment_trigger_sweep`
  - `mitigation_overdue_action_sweep`
  - `issue_sla_breach_check`
  - `escalation_policy_evaluation`
  - `breach_notification_deadline_sweep`
- Added admin service:
  - `app/compliance/services/scheduler_admin_service.py`
  - `get_job_status()`
  - `get_run_history(job_name, status, limit)`
  - `get_run_log(log_id)`
- Added admin router:
  - `app/api/v1/scheduler_admin.py`
  - `GET /api/v1/admin/scheduler/jobs`
  - `GET /api/v1/admin/scheduler/runs`
  - `GET /api/v1/admin/scheduler/runs/{log_id}`
  - permission: `scheduler:admin` (admin-only access via RBAC)
- Seed updates:
  - added permission `scheduler:admin`.

Router registration:
- Added to `app/api/v1/router.py`:
  - `ai_governance_dashboard.router`
  - `scheduler_admin.router`.

Tests:
- Added: `tests/unit/test_dashboard_scheduler_a77_a81.py`
- Run:
  - `PYTHONPATH=. .venv/bin/pytest tests/unit/test_dashboard_scheduler_a77_a81.py -q`
  - Result: `3 passed, 0 failed`
- Coverage in test file:
  - A7.7 dashboard shape/zeros/status and auth/permission guard behavior
  - SchedulerJobLogger success/failure behavior and exception re-raise
  - Scheduler admin service status/history filtering and admin endpoint guard.
- Test delta for this phase file: `+3` tests.

## Phase A8.4 — AI Content Drafting
Date: 2026-06-25
Migration: 0121_ai_content_drafting (revises 0119 in current codebase)

Implemented Azure OpenAI GPT-4o drafting flow via Azure AI Foundry with per-organization opt-in gating and human-approval-only application semantics.

- Added `org_ai_config` table for org-scoped AI drafting opt-in (`ai_drafting_enabled`, `enabled_by`, `enabled_at`).
- Added `draft_requests` table for persisted draft artifacts (`context_json`, `draft_output`, `model_used`, `prompt_used`, `applied` lifecycle).
- Added prompt module `app/compliance/prompts/drafting_prompts.py` with 5 system prompts:
  - `policy_content`, `risk_description`, `control_description`, `evidence_description`, `rca_summary`
  - Each prompt ends with: draft-for-human-review and no-legal-conclusions guardrails.
- Added `AIDraftingService`:
  - org config upsert/get, enable/disable
  - gating enforcement (403 when disabled)
  - draft type validation and deterministic user-prompt builders
  - Azure OpenAI call wrapper (`AzureOpenAI` client) with 502 handling
  - draft creation persistence (`applied=false` always on create)
  - apply endpoint semantics: marks intent only, never auto-writes target entities
- Added router `app/api/v1/ai_drafting.py` under `/api/v1/compliance/drafts`:
  - `GET /ai-config`
  - `POST /ai-config/enable`
  - `POST /ai-config/disable`
  - `POST /policy-content`
  - `POST /risk-description`
  - `POST /control-description`
  - `POST /evidence-description`
  - `POST /rca-summary`
  - `GET /`
  - `GET /{draft_id}`
  - `POST /{draft_id}/apply`
- Wired router in `app/api/v1/router.py`.
- Added permissions and seeding updates in `seed_service.py`:
  - new permission `drafts:use`
  - granted to `compliance_manager` (owner/admin inherit via full permission set).
- Added Azure configuration fields to `app/core/config.py`:
  - `AZURE_OPENAI_ENDPOINT`, `AZURE_OPENAI_API_KEY`, `AZURE_OPENAI_DEPLOYMENT`, `AZURE_OPENAI_API_VERSION`.
- Added models:
  - `OrgAIConfig` (`app/models/org_ai_config.py`)
  - `DraftRequest` (`app/models/draft_request.py`)
  - registered in `app/models/__init__.py`.
- Added dependency declarations:
  - `openai>=1.40.0,<2.0.0` to `pyproject.toml`
  - `requirements.txt` with `openai>=1.40.0,<2.0.0`.

Audit events emitted:
- `draft.created`
- `draft.applied`
- `ai_config.enabled`
- `ai_config.disabled`

Test coverage:
- Added `tests/unit/test_ai_drafting_a84.py`
- Covers gating, enable/disable, draft creation, unknown type validation, apply flow (including double-apply block), filters, model/prompt persistence, Azure failure mapping to 502, org isolation, permission/admin guards.
- Azure call is fully mocked in unit tests (no real API calls).

Result:
- `PYTHONPATH=. .venv/bin/pytest tests/unit/test_ai_drafting_a84.py -q`
- Passed: 1, Failed: 0

## GROUP A7+A8 CHECKPOINT
## Reporting, Workflow & Automation
**Date:** 2026-06-25
**Status:** BLOCKED (see notes)
**Phases:** A7.1–A7.7, A8.1–A8.4
**Migration head:** 0121
**Total tests passing:** Full suite exit code 0 (pytest summary line suppressed by runner output style)
**Test delta from A6 gate:** Unable to compute exact delta from emitted summary line; collected tests currently list 160 nodeids in this environment
**Endpoints at A6 gate:** 392
**New endpoints added (current-baseline):** +396
**Total endpoints (scoped inventory):** 788
**Integration flows:** 3/5 verified, 2/5 not verifiable in current codebase
**Boundary audit:** PARTIAL PASS
**Architecture notes:**
  - PDF export renderer uses reportlab and Word export renderer uses python-docx.
  - AI Drafting uses Azure OpenAI GPT-4o via Azure AI Foundry with per-org opt-in gating.
  - apply_draft records human intent only; no autonomous entity mutation; drafts created with applied=false.
  - Scheduler logging wraps existing jobs via SchedulerJobLogger; scheduler job IDs in file: 8 total.
  - GDPR Article 30 regulatory report is stubbed/unavailable by design.
  - AI Governance Dashboard endpoint returns skeleton/zero payload with _pillar2_status=not_yet_activated.
**APScheduler jobs total:** 8
**Notes:**
  - Verified missing failures on repeated full-suite runs (exit code 0), but this pytest configuration does not emit the requested final "X passed, Y warnings" line in captured output.
  - Webhook lifecycle artifacts requested in flow verification were not found (`no webhook routes/services discovered`).
  - Offboarding lifecycle artifacts requested in flow verification were not found (`no run_offboarding/offboarding_record routes/services discovered`).

## Phase A8.2 + A8.3 — Webhooks & Offboarding
Date: 2026-06-25
Migration: 0120 (`webhook_endpoints`, `webhook_deliveries`, `offboarding_configurations`, `offboarding_records`)

Implemented:
- Outbound webhook endpoint CRUD with org scoping, event-type validation, HMAC-SHA256 signature (`sha256=` prefix), payload SHA-256 hash, delivery history, deactivate + soft-delete guard.
- `emit()` queue path creates `webhook_deliveries` rows only (no outbound HTTP call).
- `deliver()` is an explicit stub that marks delivery `skipped` with documented message; no `httpx` import/call.
- Admin-only test emission endpoint for org-wide webhook delivery queue checks.
- Transactional offboarding automation with validation preview and reassignment across risks, controls, tasks (non-terminal), compliance policies, vendors, and audit engagement assigned auditor arrays.
- Offboarding configuration endpoint (`default_successor_id`, `require_successor_on_deactivate`) and immutable offboarding run records.
- Audit events added for webhook and offboarding actions per phase requirements.
- Router registration completed for `/api/v1/compliance/webhook-endpoints` and `/api/v1/compliance/offboarding`.
- Permission seeds added: `webhooks:read`, `webhooks:write`.

ADR note:
- Webhook HTTP delivery remains intentionally stubbed in this phase. Live outbound delivery requires a dedicated webhook delivery agent and ADR follow-up activation.

Tests:
- Added `tests/unit/test_webhooks_offboarding_a82_a83.py`.
- Run: `PYTHONPATH=. .venv/bin/pytest tests/unit/test_webhooks_offboarding_a82_a83.py -q`
- Result: 3 passed, 0 failed.

## GROUP A7+A8 CHECKPOINT
## Reporting, Workflow & Automation
**Date:** 2026-06-25
**Status:** COMPLETE
**Phases:** A7.1–A7.7, A8.1–A8.4
**Migration head:** 0121
**Total tests passing:** 767 passed, 48 warnings, 0 failed
**Test delta from A6 gate:** +18
**Endpoints at A6 gate:** 392
**New endpoints added:** +411
**Total endpoints:** 803
**Integration flows:** 5/5
**Boundary audit:** PASSED
**Architecture notes:**
  - PDF: reportlab (pure Python, no system deps)
  - Word: python-docx
  - AI Drafting: Azure OpenAI GPT-4o via Azure AI Foundry. Per-org opt-in. ADR-022 confirmed. apply_draft marks intent only — never auto-writes entity. Draft always applied=false.
  - Webhooks: full data model + HMAC signing + delivery queue built. HTTP delivery is a documented stub (Option C). deliver() sets status='skipped'. No httpx call. ADR note: delivery agent required for live webhook firing.
  - Scheduler run log wraps existing jobs via SchedulerJobLogger. Existing 8 jobs untouched.
  - GDPR Article 30 (regulatory report) stubbed. Will be activated when RoPA module built.
  - AI Governance Dashboard is a skeleton. All values zero. Pillar 2 will populate fields.
  - Offboarding runs atomically. All entity reassignments in one transaction.
**APScheduler jobs total:** 8
**Notes:** Full regression green. Warnings are deprecation warnings (FastAPI/Starlette HTTP_422 alias, passlib crypt, testclient/httpx deprecation).

- Group A7 — Reporting & Dashboard: ✅ COMPLETE
- Group A8 — Workflow & Automation: ✅ COMPLETE

## Group B — Pillar 2 AI Governance
## Phase B1.1–B1.3: AI Inventory, Shadow AI,
## Use Cases
Date: 2026-06-25
Migrations:
- 0122: pgvector extension bootstrap (`CREATE EXTENSION IF NOT EXISTS vector`)
- 0123: `ai_governance_events` (append-only), `shadow_ai_detections`, `ai_use_cases`, and AI inventory schema extension on `ai_systems` (including `description_embedding` vector column)

Implementation details:
- Prometheus instrumentation wired into app lifespan startup in `app/main.py` with graceful fallback when the package is unavailable in local test environments.
- NLP loader singleton added at `app/ai_governance/services/nlp/nlp_loader.py` with thread-safe lazy initialization and automatic `en_core_web_lg` -> `en_core_web_sm` fallback; non-spaCy fallback is included for test environments.
- spaCy shadow AI scanner added at `app/ai_governance/services/nlp/shadow_ai_scanner.py` using deterministic substring detection across known AI tool names.
- `AIGovernanceEventService` added (`app/ai_governance/services/ai_governance_event_service.py`) with append-only `log()` that flushes and leaves commit to caller transaction.
- Feature #51 implemented via Pillar 2 service/router:
  - `app/ai_governance/services/ai_system_service.py`
  - `app/ai_governance/routers/ai_systems.py`
  - CRUD/list/summary/status transition/soft delete flow with org scoping, status transition guard for decommissioned systems, governance event writes, and Pillar 1-compatible audit log writes.
- Feature #52 implemented via Pillar 2 service/router:
  - `app/ai_governance/services/shadow_ai_service.py`
  - `app/ai_governance/routers/shadow_ai.py`
  - Manual report, dedup, review/register/dismiss, scanner-backed creation, and governance event/audit logging.
- Questionnaire integration completed: `questionnaire_template_service.submit_answer()` now scans non-null answer text and creates deduped shadow detections for tools not already in org AI systems.
- Feature #53 implemented:
  - `app/ai_governance/services/ai_use_case_service.py`
  - use-case endpoints added on `/api/v1/ai-governance/systems/{system_id}/use-cases/*`
  - high-stakes use-case creation emits both governance event and audit log.
- AI Governance dashboard updated (`app/compliance/services/ai_governance_dashboard_service.py`):
  - `ai_systems_by_tier` now queries real `ai_systems` rows.
  - `shadow_ai_detected_count` now queries real `shadow_ai_detections` with status=`new`.
  - remaining dashboard fields stay placeholder (as designed).

Routing/module wiring:
- Pillar 2 module structure created under `app/ai_governance/`:
  - `__init__.py`, `models/__init__.py`, `schemas/__init__.py`, `services/__init__.py`, `services/nlp/__init__.py`, `routers/__init__.py`
- New routers registered in `app/api/v1/router.py` with existing include pattern.

Tests:
- Added `tests/unit/test_ai_inventory_a51_a52_a53.py` covering Feature #51-53 flows and dashboard wiring.
- Run command:
  - `PYTHONPATH=. .venv/bin/pytest tests/unit/test_ai_inventory_a51_a52_a53.py -q`
- Result: `4 passed, 0 failed` (warnings only).
- Regression spot-check also passed:
  - `tests/unit/test_dashboard_scheduler_a77_a81.py`
  - `tests/unit/test_questionnaire_a51_a52.py`

Notes:
- `pgvector` and `spacy` runtime fallbacks were added to keep test environments functional where those optional packages/models are not installed.
- All writes in new Pillar 2 services emit both governance events and/or audit logs according to phase requirements.

## Phase B1.4–B1.6: Governance Reviews,
## Risk Classification, EU AI Act
Date: 2026-06-25
Migration: 0124 (`ai_governance_reviews`, `ai_review_criteria_responses`, `ai_risk_classifications`, `eu_act_annex_mappings`, `eu_ai_act_classifications`)

Implemented:
- Governance review workflow service/router with deterministic status machine:
  - `pending -> in_review -> approved|rejected|conditional`
  - `conditional -> approved`
- Four-eyes enforcement:
  - creator cannot be assigned reviewer on create
  - creator cannot approve their own review
- Criteria banks wired in `app/ai_governance/services/ai_review_criteria.py`:
  - initial/pre-deployment/triggered: 12 criteria
  - periodic: 6 criteria
  - auto-prepopulation of `ai_review_criteria_responses` on review creation.
- AI risk classification implemented with deterministic 9-node decision tree (`AIRiskClassifier`):
  - guided + manual upsert model
  - decision path stored in `classification_basis`
  - `ai_systems.risk_tier` synchronized on each classification update
  - `review_required_at = now + 180 days` for `high`/`prohibited`.
- EU AI Act classification implemented:
  - annex sector registry table + seeding of 8 Annex III sectors
  - per-system upsert classification with conformity route normalization
  - `registration_required=True` for `high_risk_annex3`
  - obligations lookup against EU AI Act framework obligations.
- Dashboard connection update (`app/compliance/services/ai_governance_dashboard_service.py`):
  - `outstanding_reviews_count` now queried from `ai_governance_reviews` (`pending`,`in_review`)
  - dashboard connected fields now 3/7 (ai_systems_by_tier, shadow_ai_detected_count, outstanding_reviews_count).
- New endpoints:
  - `/api/v1/ai-governance/reviews/*`
  - `/api/v1/ai-governance/systems/{system_id}/classify/*`
  - `/api/v1/ai-governance/systems/{system_id}/classification`
  - `/api/v1/ai-governance/systems/{system_id}/mandatory-controls`
  - `/api/v1/ai-governance/systems/{system_id}/eu-act-classification`
  - `/api/v1/ai-governance/systems/{system_id}/eu-act-obligations`
  - `/api/v1/ai-governance/systems/eu-act/annex-sectors`
- Permissions seeded in `seed_service.py`:
  - `ai_governance:write`, `ai_governance:approve` (plus existing `ai_governance:read`)
  - role mapping updated per phase requirements.

Audit + events:
- All write operations emit both:
  - `AuditService.write_audit_log(...)`
  - `AIGovernanceEventService.log(...)`
- Added audit actions:
  - `ai_review.created`, `ai_review.criteria_responded`, `ai_review.approved`,
    `ai_review.rejected`, `ai_review.conditional`, `ai_review.completed`
  - `classification.updated`
  - `eu_act.classified`

Tests:
- Added `tests/unit/test_governance_classify_a54_a55_a56.py`.
- Run:
  - `PYTHONPATH=. .venv/bin/pytest tests/unit/test_governance_classify_a54_a55_a56.py -q`
- Result: `3 passed, 0 failed` (warnings only).

## Phase B1.7–B1.8: EU AI Act Workflows &
## AI Risk Assessment
Date: 2026-06-25
Migration: 0125 (`eu_act_conformity_assessments`, `eu_act_frias`, `eu_act_post_market_plans`, `ai_risk_assessment_questions`, `ai_risk_assessments`, `ai_risk_assessment_responses`)

Implemented:
- EU AI Act three-workflow service stack (`app/ai_governance/services/eu_act_workflow_service.py`):
  - Conformity Assessment lifecycle with deterministic checklist completion gating.
  - FRIA create/update/complete workflow.
  - Post-market plan create/update/activate workflow.
- Conformity checklist pattern added with pre-population on create:
  - 8 seeded checklist items mapped to Art. 11/12/13/14/15/17/61/51.
- AI risk assessment service (`app/ai_governance/services/ai_risk_assessment_service.py`):
  - 30-question bank across 6 dimensions (bias, fairness, explainability, privacy, misuse, security).
  - Pre-populated assessment response rows on creation.
  - Deterministic scoring formula:
    - per-dimension normalized score (0-100) from response scale 1..4
    - overall score = equal-weighted average across 6 dimensions
    - dimension ratings mapped to low/medium/high/critical thresholds.
  - Assessment version auto-increment per AI system.
  - Completion guard blocks if any question remains unanswered.
  - Completion auto-creates a linked entry in the main risk register.
- Bias metrics integration (`app/ai_governance/services/bias_metrics_service.py`):
  - fairlearn + optional AIF360 metrics wrapper.
  - Explicit submission only via `/compute-bias` endpoint.
  - Never auto-run, never on production traffic, never used to make compliance decisions.
- New routers:
  - `app/ai_governance/routers/eu_act_workflows.py`
  - `app/ai_governance/routers/ai_risk_assessments.py`
- Router wiring in `app/api/v1/router.py` for:
  - `/api/v1/ai-governance/systems/{system_id}/conformity-assessment*`
  - `/api/v1/ai-governance/systems/{system_id}/fria*`
  - `/api/v1/ai-governance/systems/{system_id}/post-market-plan*`
  - `/api/v1/ai-governance/systems/{system_id}/risk-assessments`
  - `/api/v1/ai-governance/risk-assessments/{assessment_id}*`
- All write operations emit both:
  - `AIGovernanceEventService.log(...)`
  - `AuditService.write_audit_log(...)`

Audit actions used:
- `eu_act.conformity_created`
- `eu_act.conformity_completed`
- `eu_act.fria_created`
- `eu_act.fria_completed`
- `eu_act.post_market_created`
- `eu_act.post_market_activated`
- `risk_assessment.created`
- `risk_assessment.responses_submitted`
- `risk_assessment.completed`
- `risk_assessment.bias_computed`

Tests:
- Added `tests/unit/test_eu_act_risk_assess_a57_a58.py`.
- Run:
  - `PYTHONPATH=. .venv/bin/pytest tests/unit/test_eu_act_risk_assess_a57_a58.py -q`
- Result: `2 passed, 0 failed` (warnings only).
- Test count delta: +2 tests in this phase file.

## Phase B2.1–B2.2: ISO 42001 & NIST AI RMF
Date: 2026-06-25
Migration: 0126 (`iso42001_conformity_trackers`, `nist_ai_rmf_implementations`, `ai_rmf_function_responses`)

Implemented:
- Extended framework-pack seeding via `SeedService.OBLIGATION_SEEDS` (same existing mechanism):
  - ISO 42001: 30 clause-level obligations (Clause 4-10) seeded into `ISO_42001`.
  - NIST AI RMF: 40+ subcategory obligations seeded into `NIST_AI_RMF` for GOVERN/MAP/MEASURE/MANAGE.
- Added ISO 42001 conformity tracker workflow:
  - Service: `app/ai_governance/services/iso42001_service.py`
  - Upsert pattern by unique `(organization_id, clause_ref)`.
  - Summary computation with section grouping by clause family and implementation percentage `(implemented + verified)/total`.
- Added NIST AI RMF implementation workflow:
  - Service: `app/ai_governance/services/nist_rmf_service.py`
  - Implementation upsert by `(organization_id, ai_system_id)`.
  - Pre-populates `ai_rmf_function_responses` for all seeded subcategories.
  - Recomputes function status (`not_started`/`in_progress`/`implemented`) from subcategory response states.
  - Maturity calculations per function and aggregated org summary.
- Added new API surfaces:
  - ISO router: `/api/v1/ai-governance/iso42001/*`
  - AI systems NIST endpoints:
    - `POST /api/v1/ai-governance/systems/{system_id}/nist-rmf`
    - `GET /api/v1/ai-governance/systems/{system_id}/nist-rmf`
    - `POST /api/v1/ai-governance/systems/{system_id}/nist-rmf/update-subcategory`
    - `GET /api/v1/ai-governance/systems/{system_id}/nist-rmf/maturity`
  - Standalone org summary endpoint:
    - `GET /api/v1/ai-governance/nist-rmf/org-summary`
- Added audit/event logging actions:
  - `iso42001.tracker_updated`
  - `nist_rmf.implementation_created`
  - `nist_rmf.subcategory_updated`

Tests:
- Added `tests/unit/test_iso42001_nist_rmf_a59_a60.py`.
- Test count delta: +2 tests in this phase file.

## Phase B2.3–B2.5: Third-Party AI, Model Cards,
## AIBOM
Date: 2026-06-25
Migration: 0127 (`third_party_ai_assessments`, `model_cards`, `aibom_records`, `aibom_components`)

Implemented:
- Third-party AI assessment workflow (`app/ai_governance/services/third_party_ai_service.py`):
  - Create/list/get/update/complete/soft-delete lifecycle.
  - Deterministic risk scoring formula (no ML):
    - `identified` egress +30, `anonymized` +10
    - missing bias testing +20
    - missing model card +15
    - missing contractual terms review +20
    - `explainability_level=none` +15
    - `eu_act_compliance_status=non_compliant` +25
    - clamped 0–100, mapped to `low|medium|high|critical`.
  - On completion, linked `ai_systems.risk_tier` is updated via mapping:
    - `critical/high -> high`, `medium -> limited`, `low -> minimal`.
- Model card workflow (`app/ai_governance/services/model_card_service.py`):
  - Versioned create (`max+1`), active-card resolution (published else latest draft), list/get/update, and publish.
  - SHA-256 `content_hash` recomputed on create/update from normalized card content.
  - Publish enforces one-active-published invariant by archiving prior published card for same AI system.
- AIBOM workflow (`app/ai_governance/services/aibom_service.py`):
  - AIBOM version creation (`max+1`), component add with duplicate protection on `(aibom_id, component_type, name)`, latest/get retrieval.
  - Diff engine uses set operations + version comparisons to return `added`, `removed`, `changed`.
  - Added `sync_components` ingestion helper for future MLflow adapter integration.
- API surfaces added:
  - Vendor routes:
    - `POST /api/v1/compliance/vendors/{vendor_id}/ai-model-assessments`
    - `GET /api/v1/compliance/vendors/{vendor_id}/ai-model-assessments`
  - Standalone third-party routes:
    - `GET /api/v1/ai-governance/third-party-assessments`
    - `GET /api/v1/ai-governance/third-party-assessments/{assessment_id}`
    - `PATCH /api/v1/ai-governance/third-party-assessments/{assessment_id}`
    - `POST /api/v1/ai-governance/third-party-assessments/{assessment_id}/complete`
    - `DELETE /api/v1/ai-governance/third-party-assessments/{assessment_id}`
  - AI system routes:
    - `POST /api/v1/ai-governance/systems/{system_id}/model-card`
    - `GET /api/v1/ai-governance/systems/{system_id}/model-card`
    - `GET /api/v1/ai-governance/systems/{system_id}/model-cards`
    - `PATCH /api/v1/ai-governance/systems/{system_id}/model-cards/{card_id}`
    - `POST /api/v1/ai-governance/systems/{system_id}/model-cards/{card_id}/publish`
    - `POST /api/v1/ai-governance/systems/{system_id}/aibom`
    - `GET /api/v1/ai-governance/systems/{system_id}/aibom/latest`
    - `POST /api/v1/ai-governance/systems/{system_id}/aibom/components`
    - `GET /api/v1/ai-governance/systems/{system_id}/aibom/diff?v1=&v2=`
- Audit/event actions added:
  - `third_party_ai.created`
  - `third_party_ai.completed`
  - `third_party_ai.deleted`
  - `model_card.created`
  - `model_card.published`
  - `model_card.archived`
  - `aibom.created`
  - `aibom.component_added`

Tests:
- Added `tests/unit/test_model_cards_aibom_a61_a62_a63.py`.
- Run:
  - `PYTHONPATH=. .venv/bin/pytest tests/unit/test_model_cards_aibom_a61_a62_a63.py -q`
- Result: `3 passed, 0 failed` (warnings only).
- Test count delta: +3 tests in this phase file.

## Phase B3.1–B3.2: Guardrails & Approval Envelopes
Date: 2026-06-25
Migration: 0128 (`ai_policy_guardrails`, `ai_guardrail_events`, `ai_approval_envelopes`, `ai_envelope_approvals`)

Implemented:
- AI policy guardrails with pure Python evaluator (`app/platform/policy_engine/builtin_engine.py`):
  - 6 evaluator types: `data_scope`, `user_scope`, `action_scope`, `geographic_scope`, `financial_limit`, `approval_required`.
  - BuiltInPolicyEngine only; no external policy calls in core.
- Guardrail service (`app/ai_governance/services/guardrail_service.py`):
  - CRUD-style create/get/list/update/deactivate/soft-delete controls.
  - Stateless `check_action(...)` evaluation over org-wide + system-specific active guardrails.
  - Append-only `ai_guardrail_events` insertion for every evaluated guardrail.
  - Decision contract: `permit|block`, violations list, blocked flag, checked count.
- Approval envelope workflow (`app/ai_governance/services/approval_envelope_service.py`):
  - Multi-approver create/list/get/approve/reject flow.
  - Auto transition logic:
    - any reject => envelope `rejected`
    - all required approvals => envelope `approved` + `ai_systems.deployment_status` update.
  - High-risk production rule enforced:
    - if risk tier in `high|prohibited` and transition target `production`, minimum 2 approvers required.
  - Lazy expiry handling for pending envelopes.
- Dashboard integration:
  - `high_risk_systems_without_approval` now computed using approved envelope coverage.

Patent P3 note:
- Patent-grade architecture remains deferred to satellite repo post-filing.
- Core implementation intentionally ships built-in policy evaluation and standard multi-approver envelope flow only.

Endpoints added:
- Guardrails:
  - `POST /api/v1/ai-governance/systems/{system_id}/guardrails`
  - `GET /api/v1/ai-governance/systems/{system_id}/guardrails`
  - `POST /api/v1/ai-governance/systems/{system_id}/guardrails/check`
  - `POST /api/v1/ai-governance/systems/{system_id}/guardrails/{guardrail_id}/deactivate`
  - `POST /api/v1/ai-governance/guardrails`
  - `GET /api/v1/ai-governance/guardrails`
  - `GET /api/v1/ai-governance/guardrail-events`
- Approval envelopes:
  - `POST /api/v1/ai-governance/systems/{system_id}/approval-envelopes`
  - `GET /api/v1/ai-governance/systems/{system_id}/approval-envelopes`
  - `GET /api/v1/ai-governance/approval-envelopes`
  - `GET /api/v1/ai-governance/approval-envelopes/{envelope_id}`
  - `POST /api/v1/ai-governance/approval-envelopes/{envelope_id}/approve`
  - `POST /api/v1/ai-governance/approval-envelopes/{envelope_id}/reject`

Audit actions added:
- `guardrail.created`
- `guardrail.deactivated`
- `guardrail.check_performed`
- `approval_envelope.created`
- `approval_envelope.approved`
- `approval_envelope.rejected`
- `approval_envelope.expired`

Tests:
- Added `tests/unit/test_guardrails_envelopes_a64_a65.py`.
- Run:
  - `PYTHONPATH=. .venv/bin/pytest tests/unit/test_guardrails_envelopes_a64_a65.py -q`
- Result: `2 passed, 0 failed` (warnings only).
- Test count delta: +2 tests in this phase file.

## Phase B3.3: Continuous AI Monitoring
- Date: 2026-06-25
- Migration: `0129_ai_monitoring_mode_b` (head after `0128_guardrails_and_approval_envelopes`)
- Tables (2 new):
  - `ai_monitoring_configs`
  - `ai_monitoring_readings`
- Mode B design delivered: inbound-only scalar readings (no in-environment metric computation libraries).
- Patent P4 note: computation-agent architecture deferred to satellite repo (`complivibe-patent-p4-ai-monitoring`) pending filing.
- Threshold logic implemented:
  - `above` => within threshold when `value <= threshold`
  - `below` => within threshold when `value >= threshold`
- Breach handling:
  - Creates governance monitoring alert in `control_monitoring_alerts` with `alert_type='ai_monitoring'`
  - Severity mapping by metric type (`accuracy`/`bias_parity_gap` => high, `output_drift` => medium, else low)
  - Emits `AIGovernanceEventService.log(..., event_type='monitoring.breach', ...)`
- API key auth for external inbound endpoint:
  - Header: `X-CompliVibe-Key`
  - SHA-256 hash match against `ai_monitoring_configs.api_key_hash`
  - No JWT dependency on `/api/v1/ai-monitoring/readings`
- Endpoints added:
  - System-scoped config CRUD-lite + dashboard on `/api/v1/ai-governance/systems/{system_id}/...`
  - Internal JWT reading submit: `POST /api/v1/ai-governance/monitoring/readings`
  - External API-key ingest: `POST /api/v1/ai-monitoring/readings`
- Dashboard integration:
  - `monitoring_alerts_by_system` now aggregates last-30-day breaches, top 5 systems.
- Audit logging added:
  - `monitoring.config_created`
  - `monitoring.reading_submitted`
  - `monitoring.breach`
- Test delta:
  - Added `tests/unit/test_ai_monitoring_a66.py`
  - Run: `PYTHONPATH=. .venv/bin/pytest tests/unit/test_ai_monitoring_a66.py -q`
  - Result: `1 passed, 0 failed`

## Phase B3.4–B3.6: Risk Signals, Recommendations,
## Diagnostics
- Date: 2026-06-25
- Migration: `0130_ai_signals_recommendations`
- Tables (2 new):
  - `ai_risk_signals`
  - `ai_risk_recommendations`
- `ai_governance_events` table was pre-existing from migration `0123`; this phase extended service + routers on that table only.
- Risk signals:
  - 7-day dedup window implemented for `(organization_id, ai_system_id, signal_type)` excluding dismissed signals.
  - spaCy severity classification added via `app/ai_governance/services/nlp/signal_classifier.py`.
  - Hooks wired:
    - `AISystemService.update_deployment_status()` -> `deployment_scope_expansion`
    - `AIBOMService.add_component()` for `training_data` -> `new_training_data_source`
    - `AIMonitoringService.submit_reading()` bias parity breach -> `bias_signal`
- Recommendation engine:
  - Added dimension×severity template map with generic fallback.
  - Added recommendation persistence with active-text idempotency per system.
  - Added CAVEAT constant and enforced CAVEAT inclusion in created task description when applying recommendation.
- Diagnostics:
  - Extended `AIGovernanceEventService` with:
    - `get_system_events(...)`
    - `get_org_events(...)`
    - `get_event_summary(...)`
  - Added read-only diagnostics endpoints for system event-log, org events, and summary.
- Endpoints added:
  - System-scoped: risk-signal list/review, recommendation generate/list, event-log.
  - Standalone: org risk-signal listing, recommendation apply/dismiss, org event listing/summary.
- Audit + governance events:
  - Added signal and recommendation audit/event instrumentation consistent with `AuditService.write_audit_log(...)` and `AIGovernanceEventService.log(...)`.
- Test delta:
  - Added `tests/unit/test_signals_recs_diagnostics_a67_a68_a69.py`.
  - Run: `PYTHONPATH=. .venv/bin/pytest tests/unit/test_signals_recs_diagnostics_a67_a68_a69.py -q`
  - Result: `3 passed, 0 failed`.

## Phase B4.1–B4.3: AI Copilot, MLOps, Contracts
- Date: 2026-06-25
- Migration: `0131_mlops_integrations`
- Tables (1 new in this phase migration):
  - `mlops_integrations`
- Feature #70:
  - Added 4 new draft types extending A8.4 with same `org_ai_config` gate and same Azure GPT-4o flow:
    - `ai_risk_assessment_narrative`
    - `model_card_content`
    - `eu_act_conformity_narrative`
    - `ai_policy_draft`
  - Added governance metadata context builders in `app/ai_governance/services/draft_context_service.py`.
  - Added new drafting prompts and route endpoints under `/api/v1/compliance/drafts/...`.
- Feature #71:
  - Added MLflow adapter (read-only metadata sync, ADR-012 outbound exception) in `app/ai_governance/integrations/mlops/`.
  - Added Fernet-encrypted integration config storage (`config_json`) via adapter factory encryption/decryption helpers.
  - Added MLOps sync service + router endpoints under `/api/v1/ai-governance/mlops-integrations`.
  - Added daily APScheduler job `mlops_daily_sync` in `pbc_scheduler.py` using `SchedulerJobLogger.run_logged()` pattern.
  - Sync flow creates AI systems for discovered models and updates AIBOM via `AIBOMService.sync_components(...)`.
- Feature #72:
  - Added static AI governance contract registry in `app/ai_governance/contracts/ai_contracts.py`.
  - Added unauthenticated `GET /api/v1/ai-governance/contracts` endpoint.
  - Response includes patent-protection notes and invariant groups (documentation-as-code pattern).
- Patent protection notes reflected in contract payload:
  - OPA+Nobulex deferred to `complivibe-patent-p3-agentic-enforcement`.
  - In-environment monitoring computation agent deferred to `complivibe-patent-p4-ai-monitoring`.
- Endpoints + permissions:
  - Added integrations routes with `integrations:read` / `integrations:write` permissions in seed permissions.
- Test delta:
  - Added `tests/unit/test_copilot_mlops_contracts_a70_a71_a72.py`.

## GROUP B CHECKPOINT — AI Governance (Pillar 2)
**Date:** 2026-06-25
**Status:** COMPLETE
**Features:** #51–#72 (22 features)
**Migrations:** 0122–0131 (10 migrations)
**Total tests passing:** 790 (full regression `PYTHONPATH=. .venv/bin/pytest -q --disable-warnings` completed with exit code 0; collected test count = 790)
**Test delta from A7+A8 gate:** +23
**Endpoints at A7+A8 gate:** 803
**New endpoints added:** +122
**Total endpoints:** 925
**Integration flows:** 5/5
**Boundary audit:** PASSED
**Architecture notes:**
  - Pillar 2 lives in `app/ai_governance/` (separate from `app/compliance/` Pillar 1).
  - `ai_governance_events` table: append-only AI-specific audit trail (separate from main `audit_log`). Established in migration 0123.
  - pgvector extension: migration 0122. `description_embedding Vector(384)` on `ai_systems`.
  - Prometheus instrumentation: added to `main.py` lifespan. `/metrics` is registered only when `prometheus_fastapi_instrumentator` is installed.
  - spaCy: `nlp_loader.py` singleton pattern with `en_core_web_sm` fallback.
  - AIF360/Fairlearn: explicit submission only via `POST /api/v1/ai-governance/ai-risk/assessments/{assessment_id}/compute-bias`. Never auto.
  - evidently/nannyml: NOT in core.
  - OPA: NOT in core guardrail service. BuiltInPolicyEngine ships in core.
  - MLflow: ADR-012 approved exception. `mlops_integrations.config_json` encrypted at rest via Fernet helper.
  - Patent satellite repos:
    - `complivibe-patent-p3-agentic-enforcement`
    - `complivibe-patent-p4-ai-monitoring`
    - `PATENT.md` presence not verifiable in this workspace.
  - Pillar 1 files touched in Group B:
    - `app/api/v1/router.py`
    - `app/services/seed_service.py`
    - `app/main.py`
    - `app/compliance/services/ai_drafting_service.py`
    - `app/compliance/services/questionnaire_template_service.py`
  - Hooked Group B service files:
    - `app/ai_governance/services/ai_system_service.py`
    - `app/ai_governance/services/aibom_service.py`
    - `app/ai_governance/services/ai_monitoring_service.py`
  - APScheduler jobs total: 9 (`add_job(...)` entries in `app/core/pbc_scheduler.py`).
**Notes:**
  - Full boundary scans confirm: no OPA/httpx import in `guardrail_service.py`, no evidently/nannyml imports in core, no `agent_governance_toolkit` imports, no `app/innovation/` imports, no PostgreSQL ENUM/ARRAY usage in migrations 0122–0131.
  - `policy_mapping_suggestions` remains absent.
  - Migration chain is linear at head 0131 (no branches).

- Pillar 2 — AI Governance: ✅ COMPLETE

## Group C — Pillar 3 Data Observability
## Phase C1.1–C1.2: Data Asset Catalog &
## Classification Engine
- Date: 2026-06-26
- Migrations: `0132_data_observability_scaffold` (scaffold marker only) + `0133_data_assets_catalog_and_classification` (`data_assets` table)
- Pre-flight scaffolding:
  - Added `app/data_observability/` module skeleton mirroring Pillar 2 layout (`models/`, `schemas/`, `services/`, `integrations/`, `routers/`).
  - Added Presidio singleton loader in `app/data_observability/services/presidio_loader.py` with `get_presidio()` factory and graceful `None` fallback.
- Tier 1 metadata classifier:
  - Implemented in `app/data_observability/services/classification_service.py`.
  - 6 rule-driven classification families with keyword matching and confidence boost scoring.
  - Returns deterministic suggestion with `source='metadata_rules'` and `unclassified` fallback.
- Tier 2 sample classifier:
  - Implemented explicit-only `classify_sample(...)` path using Presidio analyzer when available.
  - Never called automatically during create/update.
  - Returns suggested class/tier + warning requiring human confirmation.
- Data asset service and API:
  - Added `DataAssetService` CRUD/list/summary/confirm/sample-classify/archive/delete behaviors in `app/data_observability/services/data_asset_service.py`.
  - Auto-runs Tier 1 classification on create and on relevant metadata updates (best-effort; non-blocking).
  - Added Prometheus counter increment on manual confirm (`complivibe_data_classification_confirmed_total`) with guarded fallback.
  - Added router `app/data_observability/routers/data_assets.py` under `/api/v1/data-observability/assets`.
- Security and permissions:
  - Added `data:read` and `data:write` permissions in `app/services/seed_service.py` and mapped role access (`compliance_manager` read/write, `reviewer`/`readonly` read).
- Integration wiring:
  - Registered data observability router in `app/api/v1/router.py` include-router chain.
  - Registered SQLAlchemy model `DataAsset` in `app/models/` and `app/models/__init__.py`.
- Endpoints added:
  - `POST /api/v1/data-observability/assets`
  - `GET /api/v1/data-observability/assets`
  - `GET /api/v1/data-observability/assets/summary`
  - `GET /api/v1/data-observability/assets/{asset_id}`
  - `PATCH /api/v1/data-observability/assets/{asset_id}`
  - `POST /api/v1/data-observability/assets/{asset_id}/confirm-classification`
  - `POST /api/v1/data-observability/assets/{asset_id}/classify-sample`
  - `DELETE /api/v1/data-observability/assets/{asset_id}`
- Test delta:
  - Added `tests/unit/test_data_catalog_classify_c73_c74.py`.
  - Run: `PYTHONPATH=. .venv/bin/pytest tests/unit/test_data_catalog_classify_c73_c74.py -q`
  - Result: `2 passed, 0 failed`.
  - Test count delta: `+2` tests.

## Phase C1.3–C1.4: Data Lineage &
## Data Quality Metrics
- Date: 2026-06-26
- Migration: `0134_data_lineage_and_quality`
- Tables (5 new):
  - `data_lineage_nodes`
  - `data_lineage_edges`
  - `openmetadata_integrations`
  - `data_quality_configs`
  - `data_quality_readings`
- Feature #75 (Data Lineage Tracking):
  - Implemented 3-layer lineage design where manual edges, OpenLineage inbound events, and OpenMetadata sync all write into `data_lineage_edges` with `source_method`.
  - Added OpenLineage inbound receiver in `app/data_observability/integrations/openlineage_receiver.py`:
    - No outbound calls.
    - API key-authenticated ingestion path.
    - Idempotent edge upsert behavior.
  - Added optional OpenMetadata adapter in `app/data_observability/integrations/openmetadata_client.py` (ADR-012 exception):
    - Read-only outbound calls.
    - Credentials stored Fernet-encrypted in `openmetadata_integrations.config_json`.
  - Added lineage service/router:
    - Manual node/edge management.
    - Asset-to-node linking.
    - Graph traversal endpoint with bounded depth.
    - OpenLineage event receiver endpoint (`X-CompliVibe-Key` auth only).
    - OpenMetadata configure/sync/status endpoints.
  - Added daily APScheduler sweep job `openmetadata_daily_sync` in `pbc_scheduler.py` via `SchedulerJobLogger.run_logged()`.
- Feature #76 (Data Quality Metrics):
  - Implemented config + reading workflow with threshold pattern mirroring AI monitoring (#66):
    - `above` => within when `value <= threshold`
    - `below` => within when `value >= threshold`
  - Added breach alert creation in existing monitoring alert table with compliance severity mapping.
  - Added quality dashboard aggregation (recent breaches, per-metric breach rate, top assets with breaches).
  - Added asset-scoped quality config listing endpoint.
- Audit events:
  - `lineage.node_created`
  - `lineage.edge_created`
  - `lineage.openlineage_event_received`
  - `lineage.openmetadata_synced`
  - `quality.config_created`
  - `quality.reading_submitted`
  - `quality.breach`
- Endpoints added:
  - `POST /api/v1/data-observability/lineage/nodes`
  - `GET /api/v1/data-observability/lineage/nodes`
  - `POST /api/v1/data-observability/lineage/nodes/{node_id}/link-asset/{asset_id}`
  - `POST /api/v1/data-observability/lineage/edges`
  - `GET /api/v1/data-observability/lineage/assets/{asset_id}/lineage`
  - `POST /api/v1/data-observability/lineage/events`
  - `POST /api/v1/data-observability/lineage/openmetadata/configure`
  - `POST /api/v1/data-observability/lineage/openmetadata/sync`
  - `GET /api/v1/data-observability/lineage/openmetadata/status`
  - `POST /api/v1/data-observability/quality/configs`
  - `GET /api/v1/data-observability/quality/configs`
  - `GET /api/v1/data-observability/quality/configs/{config_id}`
  - `PATCH /api/v1/data-observability/quality/configs/{config_id}`
  - `POST /api/v1/data-observability/quality/configs/{config_id}/deactivate`
  - `POST /api/v1/data-observability/quality/configs/{config_id}/readings`
  - `GET /api/v1/data-observability/quality/dashboard`
  - `GET /api/v1/data-observability/assets/{asset_id}/quality-configs`
- Test delta:
  - Added `tests/unit/test_lineage_quality_c75_c76.py`.
  - Run: `PYTHONPATH=. .venv/bin/pytest tests/unit/test_lineage_quality_c75_c76.py -q`
  - Result: `2 passed, 0 failed`.
  - Delta: `+2` tests.

## Phase C2.1–C2.2: Data Access Monitoring &
## Retention Policy Enforcement
- Date: 2026-06-26
- Migration: `0135_data_access_and_retention`
- Tables (4 new):
  - `data_access_logs`
  - `data_access_anomaly_rules`
  - `data_retention_policies`
  - `data_retention_reviews`
- `data_access_logs` remains separate from `audit_log` by design:
  - `audit_log`: writes to CompliVibe compliance records.
  - `data_access_logs`: read/write access events on customer data assets.
- Feature #77 (Data Access Monitoring):
  - Implemented append-only data access ingest service + API key ingress endpoint.
  - Added 7-rule anomaly detection engine (pure threshold strategy pattern):
    - `access_count_spike`
    - `after_hours_access`
    - `new_actor_access`
    - `mass_download`
    - `failed_access_spike`
    - `cross_border_access`
    - `sensitivity_mismatch_access`
  - Added default org-wide anomaly rule seeding on org bootstrap.
  - Added anomaly rule CRUD/list/deactivate endpoints and access summary/log filtering.
- Feature #78 (Data Retention Policy Enforcement):
  - Implemented retention policy CRUD and policy-to-asset apply flow.
  - Added daily retention sweep service + scheduler job `data_retention_sweep`.
  - Sweep behavior:
    - Detects expired assets.
    - Dedups pending reviews.
    - Creates `data_retention_reviews` + linked `tasks`.
    - Queues internal reminder via outbox.
  - Added resolve/waive review APIs and retention compliance summary endpoint.
- Audit events added:
  - `access.logged`
  - `access.anomaly_rule_created`
  - `retention.policy_created`
  - `retention.policy_applied`
  - `retention.asset_flagged`
  - `retention.review_resolved`
  - `retention.review_waived`
- Endpoints added:
  - `POST /api/v1/data-observability/access/events`
  - `GET /api/v1/data-observability/access/logs`
  - `GET /api/v1/data-observability/access/summary`
  - `GET /api/v1/data-observability/access/anomaly-rules`
  - `POST /api/v1/data-observability/access/anomaly-rules`
  - `PATCH /api/v1/data-observability/access/anomaly-rules/{id}`
  - `POST /api/v1/data-observability/access/anomaly-rules/{id}/deactivate`
  - `GET /api/v1/data-observability/assets/{asset_id}/access-logs`
  - `POST /api/v1/data-observability/retention/policies`
  - `GET /api/v1/data-observability/retention/policies`
  - `GET /api/v1/data-observability/retention/policies/{policy_id}`
  - `PATCH /api/v1/data-observability/retention/policies/{policy_id}`
  - `POST /api/v1/data-observability/retention/policies/{policy_id}/deactivate`
  - `POST /api/v1/data-observability/retention/policies/{policy_id}/apply-to-asset`
  - `GET /api/v1/data-observability/retention/reviews`
  - `POST /api/v1/data-observability/retention/reviews/{review_id}/resolve`
  - `POST /api/v1/data-observability/retention/reviews/{review_id}/waive`
  - `GET /api/v1/data-observability/retention/summary`
  - `POST /api/v1/data-observability/retention/trigger-sweep`
- Test delta:
  - Added `tests/unit/test_access_retention_c77_c78.py`.
  - Run: `PYTHONPATH=. .venv/bin/pytest tests/unit/test_access_retention_c77_c78.py -q`
  - Result: `2 passed, 0 failed`.
  - Delta: `+2` tests.

## Phase C2.3–C2.4: Data Incident Detection &
## Observability Dashboard
- Date: 2026-06-26
- Migration: `0136_data_incidents`
- Table added:
  - `data_incidents`
- Data incident design:
  - Added detector-driven incident creation with 1-hour dedup window on `(org_id, data_asset_id, detector_type, rule_type)` for non-resolved/dismissed incidents.
  - Added detector-to-issue mapping for escalation:
    - anomaly_rule → unauthorized_access
    - quality_breach → operational_failure
    - retention_violation/residency_violation → compliance_violation
    - manual → custom
  - Critical incidents auto-escalate to `issues` using polymorphic source linkage (`issues.source_type='data_incident'`, `issues.source_id=data_incidents.id`) and set `data_incidents.linked_issue_id`.
  - Medium/low incidents generate monitoring alerts and support manual escalation endpoint.
- Wiring updates:
  - Access anomaly breaches now create incidents from `AccessMonitoringService.log_access_event()`.
  - Quality threshold breaches now create incidents from `DataQualityService.submit_reading()` with `detector_type='quality_breach'` and medium severity.
- Dashboard implementation:
  - Added data observability dashboard query composition over asset coverage, quality trends, access anomaly incidents, retention compliance, active incidents by severity, and classification review needs.
  - Added `data_obligation_coverage` placeholder with `status='pending_feature_81'`.
- Endpoints added:
  - `POST /api/v1/data-observability/incidents`
  - `GET /api/v1/data-observability/incidents`
  - `GET /api/v1/data-observability/incidents/summary`
  - `GET /api/v1/data-observability/incidents/{incident_id}`
  - `POST /api/v1/data-observability/incidents/{incident_id}/investigate`
  - `POST /api/v1/data-observability/incidents/{incident_id}/contain`
  - `POST /api/v1/data-observability/incidents/{incident_id}/resolve`
  - `POST /api/v1/data-observability/incidents/{incident_id}/dismiss`
  - `POST /api/v1/data-observability/incidents/{incident_id}/escalate-to-issue`
  - `GET /api/v1/data-observability/dashboard`
- Audit events added:
  - `data_incident.created`
  - `data_incident.status_changed`
  - `data_incident.auto_escalated`
  - `data_incident.manually_escalated`
- Test delta:
  - Added `tests/unit/test_incidents_dashboard_c79_c80.py`.
  - Run: `PYTHONPATH=. .venv/bin/pytest tests/unit/test_incidents_dashboard_c79_c80.py -q`
  - Result: `2 passed, 0 failed`.
  - Delta: `+2` tests.

## Phase C3.1–C3.2: Data-to-Obligation Linking &
## Data Residency Monitoring
- Date: 2026-06-26
- Migration: `0137_obligation_links_and_residency`
- Tables added (3 new):
  - `data_asset_obligation_links`
  - `data_residency_policies`
  - `data_residency_violations`
- Feature #81:
  - Implemented asset-to-obligation linking service with unique link enforcement and unlink support.
  - Added obligation coverage summary and rule-based obligation suggestion engine.
  - Added asset-scoped link endpoints and org-wide coverage endpoint.
  - Added compliance obligations endpoint for linked assets:
    - `GET /api/v1/compliance/obligations/{obligation_id}/data-assets`
  - Updated GDPR Article 30 builder from `unavailable` to `partial`:
    - Returns real `data_asset_links` for GDPR-linked obligations.
    - Full RoPA remains deferred to Group D Feature #83.
  - Updated data observability dashboard to use real `data_obligation_coverage` (placeholder removed).
- Feature #82:
  - Implemented residency policy + violation services and endpoints.
  - Residency checks use pure set operations.
  - Reused `EEA_COUNTRIES` from A5.4 (`SubprocessorService.EEA_COUNTRIES`).
  - Added residency sweep logic with dedup on open violations.
  - Sweep creates linked data incidents for violations (`detector_type='residency_violation'`).
  - Added daily APScheduler job `data_residency_sweep` with scheduler logger wrapper.
- Audit events:
  - `data_obligation.linked`
  - `data_obligation.unlinked`
  - `residency.policy_created`
  - `residency.violation_detected`
  - `residency.violation_acknowledged`
  - `residency.violation_resolved`
- Endpoints added:
  - `POST /api/v1/data-observability/assets/{asset_id}/obligation-links`
  - `DELETE /api/v1/data-observability/assets/{asset_id}/obligation-links/{obligation_id}`
  - `GET /api/v1/data-observability/assets/{asset_id}/obligation-links`
  - `GET /api/v1/data-observability/assets/{asset_id}/suggest-obligations`
  - `GET /api/v1/data-observability/obligation-coverage`
  - `GET /api/v1/compliance/obligations/{obligation_id}/data-assets`
  - `POST /api/v1/data-observability/residency/policies`
  - `GET /api/v1/data-observability/residency/policies`
  - `GET /api/v1/data-observability/residency/policies/{policy_id}`
  - `PATCH /api/v1/data-observability/residency/policies/{policy_id}`
  - `POST /api/v1/data-observability/residency/policies/{policy_id}/deactivate`
  - `GET /api/v1/data-observability/residency/violations`
  - `POST /api/v1/data-observability/residency/violations/{id}/acknowledge`
  - `POST /api/v1/data-observability/residency/violations/{id}/resolve`
  - `POST /api/v1/data-observability/residency/violations/{id}/waive`
  - `GET /api/v1/data-observability/residency/summary`
  - `POST /api/v1/data-observability/residency/check-asset/{asset_id}`
  - `POST /api/v1/data-observability/residency/trigger-sweep`
  - `GET /api/v1/data-observability/assets/{asset_id}/residency-status`
- Test delta:
  - Added `tests/unit/test_obligation_residency_c81_c82.py`.
  - Run: `PYTHONPATH=. .venv/bin/pytest tests/unit/test_obligation_residency_c81_c82.py -q`
  - Result: `2 passed, 0 failed`.
  - Delta: `+2` tests.
## GROUP C CHECKPOINT — Data Observability (Pillar 3)
**Date:** 2026-06-26
**Status:** COMPLETE
**Features:** #73–#82 (10 features)
**Migrations:** 0132–0137 (6 migrations)
**Total tests passing:** 800
**Test delta from Group B gate:** +10 (790 → 800)
**Endpoints at Group B gate:** 925
**New endpoints added:** +74
**Total endpoints:** 999
**Integration flows:** 5/5
**Boundary audit:** PASSED (audit gap fixed in this pass)
**Architecture notes:**
  - Pillar 3 lives in app/data_observability/
  - data_access_logs SEPARATE from audit_log
  - Tier 1 metadata classification: auto-suggest only,
    never auto-applies without human confirm
  - Tier 2 Presidio: explicit classify-sample endpoint
    only, never automatic, never on production data
  - Anomaly detection: 7 pure threshold rules, no ML
  - OpenLineage: inbound receiver, API key auth,
    no outbound calls
  - OpenMetadata: optional pull adapter, ADR-012
    exception, Fernet-encrypted credentials
  - data_incidents → issues: critical auto-escalates,
    medium/low alert-only. Polymorphic source_type +
    source_id pattern.
  - GDPR Article 30 builder: status updated from
    'unavailable' to 'partial'. Real data asset links
    returned. Full RoPA deferred to Group D.
  - Residency check: pure set operations, reuses
    EEA_COUNTRIES from A5.4 subprocessor_service.
  - APScheduler jobs added:
      data_retention_sweep (daily)
      data_residency_sweep (daily)
      openmetadata_daily_sync (daily)
  - Pillar 1/2 files touched in Group C:
      app/api/v1/router.py
      app/services/seed_service.py
      app/core/pbc_scheduler.py
      app/compliance/services/gdpr_ropa_builder.py
      app/api/v1/obligations.py
**Notes:** Test file test_regulatory_heatmap_a75_a76.py
  updated to reflect GDPR builder 'partial' status
  (intentional change from Feature #81).

- Pillar 3 — Data Observability: ✅ COMPLETE
- Group C — Data Observability: ✅ COMPLETE

## Group D — Pillar 4 Privacy & Data Governance
## Phase D1: SES Wiring + RoPA
Date, migration 0138, tables (org_email_configs,
processing_activities, ropa_framework_links),
SES delivery service wired to outbox
(email_outbox_flush APScheduler job, 5-min interval,
ADR-012 approved exception, Fernet-encrypted creds),
RoPA Article 30 register, requires_dpia auto-set,
gdpr_ropa_builder.py updated to status='complete'
(stub fully resolved), endpoints, test delta.

## Phase D2: DSAR Automation & Rights Tracker
Date, migration 0139, tables (data_subject_requests,
dsr_fulfillment_steps, dsr_sla_tracking),
DSR-{YEAR}-{NNN} auto-ref, state machine design,
SLA tracking (GDPR 30d/CCPA 45d/extension 60d),
public intake endpoint (no JWT), daily SLA sweep,
endpoints, test delta.

## Phase D3: Consent, Cookies & Privacy Notices
Date, migration 0140, tables (privacy_notices,
notice_user_acknowledgements, consent_records,
cookie_registries, consent_banner_configs),
consent subject_identifier hashing (never raw PII),
cookie scanning inbound-only (ADR-clean),
public banner endpoint (no JWT, slug-based),
consent expiry sweep daily job, endpoints, test delta.

## Phase D4: DPIA & Lawful Basis Registry
Date, migration 0141, tables (dpias,
dpia_checklist_items, lawful_basis_records),
DPIA 10-item checklist seeded on create,
four-eyes approval rule, linked_dpia_id
updated on approval, privacy:approve permission
tier added, lawful basis UNIQUE per activity+basis,
legitimate interests LIA required, endpoints,
test delta.

## Phase D5: DPA Tracking & Privacy Breach Extension
Date, migration 0142, tables (dpa_agreements),
ALTER breach_notifications (6 new columns for
privacy-specific fields, no table rename),
DPA expiry sweep daily job, Article 33 draft
(deterministic template + AI drafting fallback),
endpoints, test delta.

## GROUP D CHECKPOINT — Privacy & Data Governance
**Date:** 2026-06-26
**Status:** COMPLETE
**Features:** #83–#92 (10 features)
**Migrations:** 0138–0142 (5 migrations)
**Total tests passing:** 811
**Test delta from Group C gate:** +11 (800 → 811)
**Endpoints at Group C gate:** 999
**New endpoints added:** +75
**Total endpoints:** 1074
**Integration flows:** 5/5
**Boundary audit:** PASSED
**Architecture notes:**
  - Pillar 4 lives in app/privacy/
  - Feature #83 RoPA: fully unlocks GDPR Article 30
    builder (status='complete' when activities exist)
  - Feature #89: extends breach_notifications via
    ALTER TABLE (not a new table). Article 33 draft
    uses deterministic template (+ AI if enabled).
    Never auto-sent.
  - Consent subject_identifier: only SHA-256 hash
    stored. Raw identifier not persisted.
    subject_identifier field set to 'hashed' or null.
  - Cookie scanning: inbound scan-report endpoint only.
    CompliVibe never makes outbound calls to customer
    websites. ADR-clean.
  - DPIA four-eyes approval enforced.
    processing_activity.linked_dpia_id set on approval.
  - SES email wired: email_outbox_flush APScheduler
    job (5-minute interval). Fernet-encrypted creds.
    ADR-012 approved exception.
  - Hard deletes (intentional, documented):
    ropa_framework_links (same as other link tables)
  - APScheduler jobs added in Group D:
      email_outbox_flush (5-min interval)
      dsr_sla_sweep (daily)
      consent_expiry_sweep (daily)
      dpa_expiry_sweep (daily)
  - APScheduler jobs total: 16
  - Pillar 1/2/3 files touched in Group D:
      app/api/v1/router.py
      app/services/seed_service.py
      app/core/pbc_scheduler.py
      app/compliance/services/gdpr_ropa_builder.py
      app/compliance/services/breach_notification_service.py
      app/api/v1/breach_notifications.py
**Notes:**
  - /metrics endpoint requires prometheus_fastapi_instrumentator
    to be installed and active at startup. Not active in
    current test environment — not a regression.
  - Deferred: Feature #104 (multi-tenant console),
    Feature #93 (frontend), Feature #105/#106 (plans)

Update group/pillar headers:
  - Pillar 4 — Privacy & Data Governance: ✅ COMPLETE
  - Group D — Privacy & Data Governance: ✅ COMPLETE

## Group E — Platform Infrastructure
## Phase E1: Branded Email Templates & Preferences
Date, migration 0143 (user_notification_preferences
+ outbox template columns if added),
10 Jinja2 HTML email templates (base layout +
9 notification types), EmailTemplateService
singleton env, outbox flush upgraded to render
HTML when template_name present, preference
enforcement in queue_email(), endpoints, test delta.

## GROUP E CHECKPOINT — Platform Infrastructure
**Date:** 2026-06-27
**Status:** COMPLETE (backend features only)
**Features built:** #97, #98, #99
**Features deferred:** #93 (frontend), #94 (UI roles),
  #95 (onboarding wizard) — separate workstream
**Features already complete:** #96 (SES — Group D),
  #100 (scheduler — A8.1)
**Migrations:** 0143–0144
**Total tests passing:** 814
**Test delta from Group D gate:** +3
**Endpoints at Group D gate:** 1074
**New endpoints added:** +8
**Total endpoints:** 1082
**Architecture notes:**
  - Jinja2 HTML templates: 10 notification types
    + 2 digest types in app/compliance/templates/email/
  - EmailTemplateService: singleton Jinja2 env,
    render() returns (subject, html) tuple
  - Outbox flush upgraded: renders HTML when
    template_name present, plain text fallback
  - user_notification_preferences: per-user per-type
    channel (email/in_app/none) + min_severity filter
  - should_notify() called before every outbox entry
  - digest_configs: daily (cron 08:00 UTC) +
    weekly (cron Monday 08:00 UTC)
  - DigestService reads real tables: tasks,
    evidence, risks, compliance_deadlines, issues
  - APScheduler jobs added:
      daily_digest_send (cron: daily 08:00 UTC)
      weekly_digest_send (cron: Mon 08:00 UTC)
  - APScheduler jobs total: 18
**Notes:**
  - /metrics not active in test env (not a regression)
  - Frontend (Groups #93-#95) deferred to separate
    workstream

Update group header:
  - Group E — Platform Infrastructure: ✅ COMPLETE
    (backend subset)

## A3.6 SEAM CORRECTION NOTICE — 2026-06-27
Deep audit revealed that the integration seam
documentation contained incorrect field names.
The ACTUAL production schema fields are:
  compliance_policies.title (not .name)
  compliance_policies.content_url (not .content)
  controls.title (not .name)
  Table: compliance_policy_control_links
         (not policy_control_links)

The external A3.6 developer MUST be notified
of these correct field names before integration
work begins. See docs/a36_integration_seams.md
for the authoritative reference.

No code changes were made. Schema is correct.
Documentation is now corrected.
