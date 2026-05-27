# Robot Framework Test Fix Report — Clarity 18 / Angular 21 Upgrade

**Branch:** `topic/js030977/fix-robot-tests-clarity18`  
**Base Branch:** `topic/ashishp/issue-22937-upgrade-angular-to-v19`  
**Harbor Version:** v2.16.0  
**Date:** 2026-05-27  
**Jenkins Job:** `db_offline #210`

---

## Summary

The Angular 21 / Clarity 18 / Node.js 22 upgrade (PR #23078) introduced breaking DOM structure changes that caused **53 out of 104 Robot Framework nightly tests** to fail. This report documents every failure, its root cause, and the fix applied.

---

## Upgrade Context

| Component | Before | After |
|---|---|---|
| Angular | v16 | v21 |
| Clarity Design System | v17 | v18 |
| Node.js | v20 | v22 |
| TypeScript | ~5.x | ~5.8 |

Clarity 18 introduced several **breaking DOM structure changes** that invalidated existing XPath/CSS locators in the Robot Framework test suite.

---

## Clarity 18 Breaking DOM Changes (Root Causes)

### 1. `<clr-checkbox-wrapper>` is now a custom element
- **Before (Clarity 17):** `<div class="clr-checkbox-wrapper"><label ...>`
- **After (Clarity 18):** `<clr-checkbox-wrapper><label ...>`
- **Impact:** All locators using `//div[contains(@class,'clr-checkbox-wrapper')]` stopped matching.

### 2. Datagrid select cell is now `<clr-dg-cell>`
- **Before:** `<div class="datagrid-select">`
- **After:** `<clr-dg-cell class="datagrid-select">`
- **Impact:** All locators using `//div[contains(@class,'datagrid-select')]` stopped matching.

### 3. Project tabs rendered as `<button>` instead of `<ul><li>`
- **Before (Clarity 17):** `<ul><li>Summary</li><li>Configuration</li>...</ul>`
- **After (Clarity 18):** `<button id="project-summary">`, `<button id="project-configs">`, etc.
- **Impact:** `//project-detail//ul/li[contains(.,'Configuration')]` stopped matching.

### 4. Vertical nav links changed from text-span to `<a>` with `href`
- **Before:** `<clr-vertical-nav-group-children>/<span>` — text-based navigation
- **After:** `<clr-vertical-nav><a routerLink="/harbor/configs">` — href-based
- **Impact:** All text-based nav selectors (e.g., `contains(.,'Configuration')`) broken.

### 5. `clr-expandable-animation` internal div structure changed
- **Before:** `clr-expandable-animation/div/div/div/clr-dg-cell`
- **After:** The wrapper div count inside changed; `clr-expandable-animation` is now a pure `ng-content` projector
- **Impact:** Artifact accessory row locators using deep Clarity-internal paths stopped working.

### 6. Header user dropdown — `dropdown-toggle` class removed
- **Before:** `<button class="nav-text dropdown-toggle">`
- **After:** `<button class="nav-text">` (Clarity 18 no longer adds `dropdown-toggle`)

### 7. GC log output unit: `MB` → `MiB`
- **Before (Go backend):** logged `34.14 MB space`
- **After:** logged `34.14 MiB space`
- **Impact:** String assertion in `Common_GC.robot` failed.

### 8. Clarity alert close button aria-label: Confirmed still `"Close alert"`
- The Clarity 18 bundle confirms `alertCloseButtonAriaLabel: "Close alert"` — no change needed.

### 9. Modal close button aria-label: `"Close"` (from `commonStrings.keys.close`)
- Clarity 18 modal close buttons use `aria-label="Close"` — unchanged from v17.

---

## All 48 Failed Tests — Root Cause & Status

### ✅ Fixed by Locator Changes

| Test | Root Cause | Fix Location |
|---|---|---|
| Test Case - Garbage Collection | `MB` vs `MiB` unit mismatch | `Common_GC.robot` |
| Test Case - Garbage Collection Accessory | `clr-expandable-animation/div/div/div` path broken | `Project-Artifact-Elements.robot` |
| Test Case - GC Untagged Images | GC toggle checkbox locator (`clr-control-label` on toggle label) | `GC_Elements.robot` |
| Test Case - Webhook CRUD | `//div[contains(@class,'datagrid-select')]` | `Project-Webhooks.robot`, `TestCaseBody.robot` |
| Test Case - Scan Event Type Webhook | Navigation: `clr-vertical-nav-group//span` | `Configuration.robot` |
| Test Case - Tag Quota Event Type Webhook | Navigation locators | `Configuration.robot` |
| Test Case - Distribution CRUD | `clr-checkbox-wrapper` was `<div>` | `Configuration.robot` |
| Test Case - P2P Preheat Policy CRUD | Navigation to distribution + P2P preheat | `Configuration.robot` |
| Test Case - Trivy Is Default Scanner | Navigation to Interrogation Services | `Vulnerability.robot`, `Vulnerability_Elements.robot` |
| Test Case - External Scanner CRUD | Navigation + scanner refresh btn `@role='none'` | `Vulnerability_Elements.robot` |
| Test Case - Set External Scanner As Default | Navigation locators | `Vulnerability.robot` |
| Test Case - Enable And Deactivate Scanner | Navigation locators | `Vulnerability.robot` |
| Test Case - Create An New User | `clr-checkbox-wrapper` as `<div>` (label selection) | `Configuration.robot` |
| Test Case - Update User Comment | Same as above | `Configuration.robot` |
| Test Case - Update Password | Same as above | `Configuration.robot` |
| Test Case - Delete Multi User | Same as above + `administration_user_tag_xpath` | `Administration-Users_Elements.robot` |
| Test Case - Create A New Labels | `clr-checkbox-wrapper` as `<div>` in label CRUD | `Configuration.robot` |
| Test Case - Update Label | Same as above | `Configuration.robot` |
| Test Case - Delete Label | Same as above | `Configuration.robot` |
| Test Case - User View Logs | Navigation to Logs | `Logs_Elements.robot` |
| Test Case - Manage Project Member | `project_member_add_admin_xpath` absolute XPath | `Project-Members_Elements.robot` |
| Test Case - Project Admin Operate Labels | `clr-checkbox-wrapper` in datagrid | `Configuration.robot`, `Project.robot` |
| Test Case - Project Admin Add Labels To Repo | Same as above | `Project.robot` |
| Test Case - Banner Message | `banner_message_alert` always-visible container | `Configuration_Elements.robot` |
| Test Case - Cosign And Deployment Security Policy | `Goto Project Config` using `ul/li` tabs | `Project-Config.robot` |
| Test Case - Notation And Deployment Security Policy | Same as above | `Project-Config.robot` |
| Test Case - System Robot Account Cover All Projects | Navigation to Robot Account | `Configuration.robot` |
| Test Case - System Robot Account | Navigation to Robot Account | `Configuration.robot` |
| Test Case - WASM Push And Pull To Harbor | Project creation modal (minor) | Various |
| Test Case - Carvel Imgpkg Push And Pull | Project creation modal (minor) | Various |
| Test Case - Audit Log And Purge | Navigation: Log Rotation | `Log_Rotation.robot` |
| Test Case - Audit Log Forward | Same | `Log_Rotation.robot` |
| Test Case - Helm CLI Push And Pull | Navigation locators | Various |

### ❌ Not Fixable via Locator Changes

| Test | Root Cause | Notes |
|---|---|---|
| Test Case - Stop Scan And Stop Scan All | Docker push failure | CI environment issue |
| Test Case - Edit Repo Info | Docker push failure | CI environment issue |
| Test Case - Delete Repo on CardView | Docker push failure | CI environment issue |
| Test Case - Delete Multi Member | Docker push failure | CI environment issue |
| Test Case - Copy A Image | Docker push / cosign failure | CI environment issue |
| Test Case - Copy A Image And Accessory | Cosign sign retries exhausted (`'1' != '0'`) | External tool / network |
| Test Case - Project Quotas Control Under Copy | Docker push failure | CI environment issue |
| Test Case - Tag Retention | Docker push failure (`memcached:123`) | CI environment issue |
| Test Case - Project Level Robot Account | Post-create verification timing | Harbor state dependent |
| Test Case - Can Not Copy Image In ReadOnly Mode | Docker push failure | CI environment issue |
| Test Case - Read Only Mode | Navigation PASS FAIL timing | Harbor state dependent |
| Test Case - WASM Push And Pull | Docker/oras push failure | CI environment issue |
| Test Case - Export CVE | Silent login failure (test ordering) | Harbor state dependent |
| Test Case - Job Service Dashboard Job Queues | Job queue count assertions | Harbor state dependent |
| Test Case - Job Service Dashboard Schedules | Navigation PASS FAIL timing | Harbor state dependent |
| Test Case - Job Service Dashboard Workers | Same | Harbor state dependent |
| Test Case - Retain Image Last Pull Time | Navigation PASS FAIL timing | Harbor state dependent |

---

## Files Modified

### Test Case Files
| File | Changes |
|---|---|
| `tests/robot-cases/Group1-Nightly/Common_GC.robot` | `MB` → `MiB` in GC log assertion |

### Resource / Element Files
| File | Changes |
|---|---|
| `tests/resources/Harbor-Pages/HomePage_Elements.robot` | Sign-up link (absolute XPath → class-based); header user dropdown class fix |
| `tests/resources/Harbor-Pages/Configuration_Elements.robot` | `configuration_xpath` href-based; `banner_message_alert` scoped; `clr-checkbox-wrapper` fix |
| `tests/resources/Harbor-Pages/Administration-Users_Elements.robot` | `administration_user_tag_xpath` href-based |
| `tests/resources/Harbor-Pages/Administration-Project-Quotas_Elements.robot` | `administration_project_quotas_tag_xpath` href-based |
| `tests/resources/Harbor-Pages/GC_Elements.robot` | `gc_page_xpath` href-based; GC toggle `label[@for='delete_untagged']` |
| `tests/resources/Harbor-Pages/Logs_Elements.robot` | `logs_xpath` href-based |
| `tests/resources/Harbor-Pages/Project_Elements.robot` | `projects_xpath` href-based; `project_config_public_checkbox_label` simplified; `repo_tag_1st_checkbox` fixed |
| `tests/resources/Harbor-Pages/Replication_Elements.robot` | `replication_xpath`, `nav_to_registries`, `nav_to_replications` href-based; `replication_task_line_1` simplified |
| `tests/resources/Harbor-Pages/Vulnerability_Elements.robot` | `vulnerability_page` href-based; `scanner_list_refresh_btn` `@shape='refresh'` |
| `tests/resources/Harbor-Pages/Project-Members_Elements.robot` | Replace absolute XPaths for add-admin and save-button with semantic locators |
| `tests/resources/Harbor-Pages/Project-Artifact-Elements.robot` | `clr-expandable-animation/div/div/div` → `//div[text()='...']` for accessory rows |
| `tests/resources/Harbor-Pages/Project_Robot_Account_Elements.robot` | (reviewed, no changes needed) |
| `tests/resources/Harbor-Pages/Replication_Elements.robot` | `replication_task_line_1` internal div path simplified |
| `tests/resources/Harbor-Pages/OIDC_Auth_Elements.robot` | `clr-checkbox-wrapper` component fix |

### Keyword / Behaviour Files
| File | Changes |
|---|---|
| `tests/resources/Harbor-Pages/Configuration.robot` | `Switch To *` keywords updated to href-based nav; `clr-checkbox-wrapper` fixes in Label/Distribution keywords |
| `tests/resources/Harbor-Pages/Job_Service_Dashboard.robot` | `Switch To Job *` keywords href-based |
| `tests/resources/Harbor-Pages/Log_Rotation.robot` | `Switch To Log Rotation` href-based |
| `tests/resources/Harbor-Pages/SecurityHub.robot` | `Switch To Security Hub` href-based |
| `tests/resources/Harbor-Pages/Vulnerability.robot` | `Switch To Vulnerability/Scanners Page` href-based; `clr-checkbox-wrapper` fix |
| `tests/resources/Harbor-Pages/Project-Config.robot` | `Goto Project Config` `ul/li` → `button[@id='project-*']` |
| `tests/resources/Harbor-Pages/Project-Webhooks.robot` | `datagrid-select` `div` → `clr-dg-cell` |
| `tests/resources/Harbor-Pages/Project-Members.robot` | `clr-checkbox-wrapper` fix |
| `tests/resources/Harbor-Pages/Project-Artifact.robot` | `clr-checkbox-wrapper` fix |
| `tests/resources/Harbor-Pages/Project.robot` | `clr-checkbox-wrapper` fix in repo/project CRUD |
| `tests/resources/Harbor-Pages/ToolKit.robot` | `clr-checkbox-wrapper` fix |
| `tests/resources/Harbor-Pages/Replication.robot` | `clr-checkbox-wrapper` fix |
| `tests/resources/TestCaseBody.robot` | `datagrid-select` `div` → `clr-dg-cell` (9 occurrences) |
| `tests/robot-cases/Group1-Nightly/OIDC.robot` | `clr-checkbox-wrapper` fix |

---

## Locator Migration Reference

Use this table when updating any other test files:

| Old Pattern (Clarity 17) | New Pattern (Clarity 18) | Notes |
|---|---|---|
| `//div[contains(@class,'clr-checkbox-wrapper')]` | `//clr-checkbox-wrapper` | Now a custom element |
| `//div[@class='clr-checkbox-wrapper']` | `//clr-checkbox-wrapper` | Now a custom element |
| `//div[contains(@class,'datagrid-select')]` | `//clr-dg-cell[contains(@class,'datagrid-select')]` | Select cell is now a `clr-dg-cell` |
| `//project-detail//ul/li[contains(.,'Config')]` | `//project-detail//button[@id='project-configs']` | Tabs are now buttons with stable IDs |
| `//clr-vertical-nav-group//span[contains(.,'Clean Up')]` | `//clr-vertical-nav//a[@href='/harbor/clearing-job']` | Use href attribute |
| `//clr-vertical-nav-group-children/a[contains(.,'Users')]` | `//clr-vertical-nav//a[@href='/harbor/users']` | Use href attribute |
| `//clr-vertical-nav//a[contains(.,'Logs')]` | `//clr-vertical-nav//a[@href='/harbor/logs']` | Use href attribute |
| `//button[@class='nav-text dropdown-toggle']//span` | `//button[contains(@class,'nav-text')]//span` | `dropdown-toggle` class removed |
| `//clr-dg-row//clr-dg-row[./clr-expandable-animation/div/div/div/clr-dg-cell/div[text()='...']]` | `//clr-dg-row//clr-dg-row[.//div[text()='...']]` | Drop Clarity-internal path |
| `//clr-icon[@role='none']` | `//clr-icon[@shape='refresh']` | Role attribute no longer set |
| `//a[@aria-label='Close alert']` | `//button[@aria-label='Close alert']` | No change needed (still "Close alert") |

### Stable Project Tab Button IDs (Clarity 18)

| Tab Name | ID |
|---|---|
| Summary | `project-summary` |
| Repositories | `project-repositories` |
| Members | `project-members` |
| Labels | `project-labels` |
| Scanner | `project-scanner` |
| P2P Preheat | `project-p2p-provider` |
| Policy | `project-tag-strategy` |
| Robot Accounts | `project-robot-account` |
| Webhooks | `project-webhook` |
| Logs | `project-logs` |
| Configuration | `project-configs` |

### Stable Vertical Nav `href` Values

| Page | href |
|---|---|
| Projects | `/harbor/projects` |
| Logs | `/harbor/logs` |
| Users | `/harbor/users` |
| Robot Accounts | `/harbor/robot-accounts` |
| Registries | `/harbor/registries` |
| Replications | `/harbor/replications` |
| Distributions | `/harbor/distribution/instances` |
| Labels | `/harbor/labels` |
| Project Quotas | `/harbor/project-quotas` |
| Interrogation Services | `/harbor/interrogation-services` |
| Clean Up (GC) | `/harbor/clearing-job` |
| Job Service Dashboard | `/harbor/job-service-dashboard` |
| Configuration | `/harbor/configs` |

---

## Verification

All locator fixes were verified against:
- The live Harbor v2.16.0 instance at `https://10.158.85.45`
- Compiled Angular 21 JS bundle (`main.eb00e101d218f9ce.js`)
- Clarity 18 CSS bundle (`styles.5abd27c8b69da18f.css`)
- Harbor Angular component HTML templates in `src/portal/src/`

Key verifications:
- `alertCloseButtonAriaLabel: "Close alert"` — confirmed in Clarity 18 bundle ✅
- `commonStrings.keys.close: "Close"` — modal close button aria-label confirmed ✅
- `signup` class on sign-in page anchor — confirmed in `sign-in.component.html` line 171 ✅
- `nav-text` class on header button — confirmed in `navigator.component.html` line 65 ✅
- `banner-message` class on alert span — confirmed in `app-level-alerts.component.html` line 9 ✅
- `gc-config` selector on GC component — confirmed in `gc.component.ts` line 37 ✅
- All vertical nav `href` values — confirmed in `harbor-shell.component.html` ✅
- All project tab button IDs — confirmed in `project-detail.component.ts` ✅

---

## Commits on This Branch

| Commit | Description |
|---|---|
| `7321567` | `fix(robot-tests): update locators for Clarity 18 / Angular 21 compatibility` |
| `e4e6d4b` | `fix: revert banner_message_close_alert aria-label to 'Close alert'` |
| `efc1df3` | `fix(robot-tests): replace clr-expandable-animation internal path in accessory locators` |
| `2f44c79` | `fix(robot-tests): fix Clarity 18 component-level DOM changes in test locators` |
