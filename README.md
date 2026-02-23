# ⚠️ Trip Duration Limit Bypass → DoS & Error Disclosure

This repository documents a business logic vulnerability discovered during authorized security testing of an AI trip generation system.

The application enforces a 7-day limit on the frontend, but fails to validate this constraint on the backend. By submitting excessive trip durations, the system enters a long processing state, resulting in server timeouts and exposed error responses.

---

## 📌 Key Issue

- Missing server-side validation for trip duration
- Ability to submit oversized requests beyond intended limits
- Backend processes unbounded input leading to instability

---

## ⚠️ Impact

- Application-level Denial of Service (DoS)
- 504 Gateway Timeout errors under load
- Exposure of internal server details (error pages)
- Degraded user experience and system reliability

---

## 🛠️ Root Cause

- Trusting client-side restrictions
- Lack of input validation on backend
- No workload limits or request throttling

---

## ✅ Recommendation

- Enforce strict duration limits on backend
- Reject oversized requests before processing
- Implement rate limiting and abuse detection
- Return safe, sanitized error responses
