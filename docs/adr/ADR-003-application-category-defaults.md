# ADR-003: Default Browser & Multi-Purpose Application Categorization

**Date:** 2026-11-17  
**Status:** Approved (Owner confirmed keeping the 6 existing categories for MVP, `app_type = browser`, `Others / Uncategorized` default, and persistent manual overrides)  
**Owners:** Product Owner, Application Recognition Engineering  

---

## 1. Context

`PRD.md` (§10) establishes six initial categories (`Productivity`, `Entertainment`, `Gaming`, `Communication`, `System & Utilities`, `Others / Uncategorized`) and explicitly prohibits assuming Chrome or any multi-purpose browser is automatically productive. `PRIVACY_SECURITY.md` and `PRODUCT_REQUIREMENTS.md` (`PV-004`) prohibit collecting browser URLs, tab titles, or website history in v1. An explicit decision was needed on the default browser category and whether MVP should retain 6 categories or add a 7th `Web & Browsing` category.

## 2. Problem

Because Timo v1 tracks browsers solely at the foreground application level (without URL/content inspection), users perform work, study, messaging, video watching, and web gaming inside the same browser binary. Defaulting browsers to `Productivity` biases insights upward unfairly, defaulting them to `Entertainment` introduces the opposite judgment, and labeling them `unknown` identity hides the fact that Timo accurately identified Chrome, Edge, or Firefox.

## 3. Options Considered

1. **Option A: Maintain the 6 approved categories; recognize browsers as `app_type = "browser"` with default category `Others / Uncategorized` and persistent manual override.**
2. **Option B: Default browsers to `System & Utilities`.**
3. **Option C: Add a 7th MVP category (`Web & Browsing`).**
4. **Option D: Default browsers to `Productivity` with `low` confidence.**

## 4. Technical Trade-offs

| Option | Alignment with `PRD.md` | Identity Honesty | MVP Simplicity |
|---|---|---|---|
| Option A: 6 categories + `app_type = "browser"` + `Others / Uncategorized` default | Exact match: zero judgment about browser intent | Recognizes `Google Chrome` / `Microsoft Edge` cleanly (`app_type = browser`) while keeping category neutral | Preserves the 6-category schema, icons, and accessibility palette tokens (`ADR-004`) |
| Option B: `System & Utilities` | Misleading for everyday browsing | Confuses browsers with OS tools | Low discoverability |
| Option C: Add 7th `Web & Browsing` category | Clean, but expands UI palette, legend, and seed schema | High honesty | Unnecessary schema/palette expansion for MVP |
| Option D: `Productivity` (`low` confidence) | Violates `PRD.md` §10 | Biases Overview metrics upward on day one | Rejected |

## 5. Recommended Decision

Adopt **Option A (Approved Owner Decision)**:
1. **Keep 6 Categories for MVP:** Seed only the 6 established categories (`Productivity`, `Entertainment`, `Gaming`, `Communication`, `System & Utilities`, `Others / Uncategorized`). Do **not** add a `Web & Browsing` category in MVP.
2. **Browser Recognition & Neutral Default:**
   - Curated recognition rules identify known web browsers (`chrome.exe`, `msedge.exe`, `firefox.exe`, `brave.exe`, `opera.exe`, `arc.exe`, etc.) with **`app_type = "browser"`**.
   - Default initial **`category_id` = `Others / Uncategorized`**, never `Productivity` (`PRD.md`, `AR-007`).
   - Store `classification_state = "low"` (category certainty, while `applications.display_name`, `publisher`, and `app_type = "browser"` reflect verified application identity) so browsers remain discoverable in the Review Queue alongside uncategorized apps without blocking tracking.
3. **Manual Override & Persistence:**
   - Users can assign any browser to `Productivity`, `Entertainment`, `Communication`, or any other category manually (`AR-008`).
   - User overrides receive `classification_state = "confirmed_user"`, persist across app updates and restarts, and take precedence over automatic rules.
   - "Reset to automatic" returns the browser to `Others / Uncategorized` (`app_type = "browser"`).
4. **Non-Browser Unknown EXEs:** Unrecognized standalone executables (`AR-003`, `AR-009`) default to `classification_state = "unknown"` under `Others / Uncategorized` and are never assumed to be games solely from their filename.

## 6. Decision Status

**Approved** (Owner confirmed 6 categories for MVP, `app_type = "browser"`, `Others / Uncategorized` default, persistent manual category overrides, and no Productivity assumption).

## 7. Consequences

- **Positive:** Aligns `PRD.md`, `PRODUCT_REQUIREMENTS.md`, `APPLICATION_RECOGNITION_SPEC.md`, `DATABASE_SCHEMA.md`, and `DESIGN_SYSTEM.md` without expanding category tokens or violating v1 URL-free privacy rules.
- **Negative / Risk:** Until a user categorizes their browser, browser sessions sit in `Others / Uncategorized` (mitigated by clear label display and one-click Review Queue / Application Details recategorization).

## 8. Implementation Implications

- Seed the 6 approved categories cleanly in SQLite migration `001`.
- Ensure curated browser rules set `app_type = "browser"` and `category_id = "others"` with neutral explanatory text in Review Queue / Application Details (*"Browsers are used for many kinds of work and leisure. Choose a category anytime."*).
- Keep manual category edits atomic (`UPDATE applications SET category_id = ?, classification_state = 'confirmed_user', source_rule_id = 'user_override' WHERE id = ?`) without altering historical `usage_sessions`.

## 9. Validation Requirements

1. Unit tests confirm Chrome, Edge, Firefox, Brave, and Opera resolve to `app_type = "browser"` and default category `Others / Uncategorized` (never `Productivity`).
2. Integration/UI tests confirm manual category change persists across app restarts (`confirmed_user`), updates Overview/Insights category breakdowns immediately, and resets cleanly back to `Others / Uncategorized` on "Reset to automatic".

## 10. Related Documents

- [`PRD.md`](../PRD.md) (§10)
- [`PRODUCT_REQUIREMENTS.md`](../PRODUCT_REQUIREMENTS.md) (`AR-007`, `AR-008`, `AR-009`, `PV-004`, `UI-008`)
- [`APPLICATION_RECOGNITION_SPEC.md`](../APPLICATION_RECOGNITION_SPEC.md)
- [`DATABASE_SCHEMA.md`](../DATABASE_SCHEMA.md)
- [`USER_FLOW.md`](../USER_FLOW.md) (§3)
- [`QA_TESTING_PLAN.md`](../QA_TESTING_PLAN.md)
