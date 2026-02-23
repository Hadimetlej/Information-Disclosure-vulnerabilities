# Trip Duration Limit Bypass → Application-Level DoS & Error Disclosure

**Vulnerability Type:** Business Logic Abuse / Insecure Design  
**Impact:** Application-level Denial of Service + Information Disclosure  
**Risk:** Medium  
**Affected Component:** AI Trip Planner (Trip generation flow)  
**Environment:** `[REDACTED_DOMAIN]` (dev)

---

## Summary

The application enforces a maximum trip duration of **7 days** through the UI/flow, however this limitation can be bypassed.  
By submitting a **significantly longer duration** (e.g., ~30 days), the backend enters a long-running processing state and returns a **504 Gateway Timeout** (nginx).  
The **raw HTML error page** is rendered directly inside the chat UI, disclosing internal server details and degrading availability.

---

## Preconditions

- Valid authenticated user session
- Access to AI Trip Planner trip creation flow

---

## Steps to Reproduce (High-Level)

1. Start creating a trip through **AI Trip Planner**.
2. Bypass the **7-day duration** restriction in the trip flow.
3. Submit a trip duration significantly above the allowed maximum (example: ~30 days).
4. Observe backend processing time exceeding gateway limits.
5. Observe a **504 Gateway Timeout** returned and **displayed in the chat response**.

---

## Observed Result

- Backend fails to gracefully handle the oversized trip duration.
- Server responds with **504 Gateway Timeout** (nginx).
- **Raw HTML error response** is shown to the end user inside the chat UI.
- Trip generation becomes inconsistent / stuck.

---

## Expected Result

- Server-side validation should reject any trip duration above the allowed maximum **before** heavy processing starts.
- The user should receive a safe, generic error message (no raw HTML/server headers).

---

## Security Impact

- **Application-Level DoS:** Repeated triggering can degrade service availability and stability.
- **Business Logic Abuse:** Core functional constraints (max 7 days) are bypassable.
- **Information Disclosure:** Internal server error page + server banner details are exposed to the user.
- **UX / Trust Impact:** Users see raw infrastructure errors in the product UI.

---

## Recommendations

1. **Enforce duration limits server-side** (hard validation) for all trip creation/update endpoints.
2. Add **workload caps** (max activities/POIs/itinerary items) proportional to trip duration.
3. Implement **rate limiting / abuse detection** for trip generation requests.
4. Move long-running generation to an **async job** with queue + progress status, and apply strict timeouts per stage.
5. Replace raw upstream error pages with **user-safe** error handling (no HTML passthrough).
6. Add monitoring/alerting on **high-latency** requests and **504/5xx spikes**.

---

## Evidence (Screenshots)

Place screenshots under `images/` and reference them here:


- `01-504-error-disclosure-ui.png` — 504 HTML error rendered inside chat UI

---

## Notes

This issue was observed in a development environment. Even if production has stronger resources, server-side validation and safe error handling are still required to prevent abuse and information disclosure.
