# ADR-003: Default Browser & Multi-Purpose Application Categorization

**Date:** 2026-11-17  
**Status:** Recommended (Requires Owner Confirmation on keeping 6 categories with `Others / Uncategorized` default vs adding a dedicated `Web Browsing` category)  
**Owners:** Product Owner, Application Recognition Engineering  

---

## 1. Context

`PRD.md` (§10) establishes six initial categories (`Productivity`, `Entertainment`, `Gaming`, `Communication`, `System & Utilities`, `Others / Uncategorized`) and explicitly states: *"Do not assume Chrome or any multipurpose application is always productive. Offer a sensible initial suggestion and make category correction easy."* `PRIVACY_SECURITY.md` and `PRODUCT_REQUIREMENTS.md` (`PV-004`) forbid browser URL, tab title, or website history tracking in v1. However, `APPLICATION_RECOGNITION_SPEC.md` left the concrete default category for recognized web browsers (`chrome.exe`, `msedge.exe`, `firefox.exe`, `brave.exe`, `opera.exe`, `arc.exe`, etc.) undefined.

## 2. Problem

Because Timo v1 tracks browsers solely as foreground applications (no domain/URL inspection), a user spends time in a browser for work, coursework, email, video streaming, or browser games simultaneously. Automatically defaulting browsers to `Productivity` violates `PRD.md` §10 and misleads users; defaulting them to `Entertainment` creates the opposite bias; marking them `unknown` identity hides the fact that Timo clearly recognized Google Chrome or Microsoft Edge.

## 3. Options Considered

1. **Option A: Neutral `Others / Uncategorized` category with `high` identity confidence + `app_type = "browser"` + gentle non-blocking review hint.**
2. **Option B: Default browsers to `System & Utilities`.**
3. **Option C: Add a 7th default category (`Web & Browsing`).**
4. **Option D: Default to `Productivity` with `low` confidence.**

## 4. Technical Trade-offs

| Option | Alignment with `PRD.md` | Identity Honesty | Review UX |
|---|---|---|---|
| Option A: `Others / Uncategorized` + `app_type = "browser"` | Exact match: avoids judging browser intent | High identity (`Google Chrome` clearly recognized), honest neutral category | Appears in Review Queue / Applications with a calm prompt: *"Browsers cover many activities. Pick a category that fits how you use it."* |
| Option B: `System & Utilities` | Misleading (browsing is rarely OS utility maintenance) | Confuses utilities with web apps | Users rarely think to recategorize Chrome out of Utilities |
| Option C: Add 7th category `Web & Browsing` | Clean conceptually, but alters the 6-category baseline in `PRD.md` & `DESIGN_SYSTEM.md` | High honesty | Requires owner approval & 7th color/icon/pattern token |
| Option D: `Productivity` (`low` confidence) | Explicitly violates `PRD.md` line 106 | Biases Overview metrics upward on first install | Bad default |

## 5. Recommended Decision

Adopt **Option A (Recommended)**:
1. **Identity vs Category Separation:** Curated recognition rules identify known browsers with **`classification_state = "high"`** (identity confidence via verified publisher + PE metadata + curated app rule) and **`app_type = "browser"`**, while assigning initial **`category_id = "others"`** (`Others / Uncategorized`) unless/until the user overrides it.
2. **Gentle Review Prompt (Non-Blocking):** In `Applications` (Review Queue) and `Application Details`, browsers show a plain-language helper note explaining that browsers are used for many tasks (study, work, media, or chat) and can be categorized manually in one click, without tracking URLs (`PV-004`).
3. **Rule Precedence & Override Persistence:**
   - Tier 1 (`confirmed_user`) always overrides curated browser rules once the user selects `Productivity`, `Entertainment`, `Communication`, etc., and persists across updates/restarts (`AR-008`).
   - Clicking "Reset to automatic" returns the browser to `Others / Uncategorized` (`app_type = "browser"`).
4. **Non-Browser Unknown EXEs:** Unrecognized standalone executables (`AR-003`, `AR-009`) receive `classification_state = "unknown"` and `category_id = "others"`, never guessing `Gaming` from filename alone (`PRD.md`, `APPLICATION_RECOGNITION_SPEC.md`).

*Alternative for Owner Choice:* If the owner prefers a 7th `Web & Browsing` category instead of keeping browsers under `Others / Uncategorized` by default, add one semantic category token (`#E6B89C` / amber-sand or neutral warm slate) in `DESIGN_SYSTEM.md` and seed it in `categories`.

## 6. Decision Status

**Recommended** — **Requires Owner Confirmation** (Confirm Option A with 6 approved categories vs Option C adding a 7th `Web & Browsing` category).

## 7. Consequences

- **Positive:** Fully consistent across `PRD.md`, `PRODUCT_REQUIREMENTS.md`, `APPLICATION_RECOGNITION_SPEC.md`, and `DATABASE_SCHEMA.md`; never fabricates productivity claims from browser use; keeps v1 URL-free.
- **Negative / Risk:** On day one before review, a heavy browser user sees substantial time in `Others / Uncategorized` (mitigated by clear label `"Uncategorized (Chrome, Edge...)"` and one-click categorization from Overview/Review Center).

## 8. Implementation Implications

- Separate `identity_confidence` vs `category_confidence` clearly, or document in `APPLICATION_RECOGNITION_SPEC.md` and `DATABASE_SCHEMA.md` that a recognized browser carries `app_type = "browser"`, `source_rule_id = "curated:browser_neutral"`, and `classification_state = "low"` (category certainty) so it surfaces in the Review Queue even though its `display_name` and publisher are verified!
- Ensure the Review Queue filters on `classification_state IN ('unknown', 'low')` so neutral browsers remain easy to review alongside unknown EXEs without blocking tracking.

## 9. Validation Requirements

- Unit tests confirm Chrome/Edge/Firefox/Brave never auto-assign `Productivity` or `Gaming`.
- UI/integration tests verify user changing Chrome to `Productivity` updates `classification_state = 'confirmed_user'`, persists across restarts, and recalculates Overview/Insights category breakdowns without altering raw `usage_sessions`.

## 10. Related Documents

- [`PRD.md`](../PRD.md) (§10)
- [`PRODUCT_REQUIREMENTS.md`](../PRODUCT_REQUIREMENTS.md) (`AR-007`, `AR-008`, `AR-009`, `PV-004`, `UI-008`)
- [`APPLICATION_RECOGNITION_SPEC.md`](../APPLICATION_RECOGNITION_SPEC.md)
- [`DATABASE_SCHEMA.md`](../DATABASE_SCHEMA.md)
- [`USER_FLOW.md`](../USER_FLOW.md) (§3)
