# CompliVibe Backend Development Log v5.0

## Group Status

- Group A1 — Risk Management Enhancements: ✅ COMPLETE
- Group A2 — Control & Compliance Enhancements: ✅ COMPLETE
- Group A3 — Policy Enhancements: ✅ COMPLETE

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
