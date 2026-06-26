# Tasks: ElitePass UI/UX Upgrades (POS & Reservations)

This checklist tracks the design specification and implementation stages for each of the three upgrades.

---

## 1. Client-Side Barcode Scanning (PWA Camera)
- [x] Define camera permission states and request workflows.
- [x] Model scanner overlays (darkened container, scanning reticle boundaries, sweeping laser line).
- [x] Plan hardware constraints controls (torch state toggle, switch camera, and manual entry fallback button).
- [x] Design success haptics (Vibration API) and synthesized audio cue (Web Audio API).
- [x] Implement scanner React components.
- [x] Integrate with POS central inventory counts and bar settings forms.

---

## 2. Interactive Category Navigation (POS Grid)
- [x] Design hierarchical category data representation (parent-child self-relationship).
- [x] Plan grid component and folder button visual tokens (using Material Symbols Outlined).
- [x] Model active folder history (state stack tracking) and horizontal breadcrumb visual trail.
- [x] Detail responsive transitions and grid display rules.
- [x] Build hierarchical categories client-side tree integration.
- [x] Implement category selection grid layout.

---

## 3. Turnstile CAPTCHA Feedback (Reservations Form)
- [x] Formulate non-blocking background verification strategy.
- [x] Detail component UI feedback mappings (Initializing, Verifying, Verified, Failed) with matching color codes (OKLCH system).
- [x] Implement visual reset workflow on verification failure.
- [x] Create React/NextJS page code blueprints.
- [x] Integrate Turnstile widget container in Reservations public registration forms.
- [x] Establish Turnstile verification in public-registration-form.
