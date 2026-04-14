# Carryover — Sprint-Submission-Queue

Items deferred from this sprint or identified for future sprints.

---

## Deferred items

| # | Item | Reason | Target |
|---|------|--------|--------|
| 1 | **Server-side enforcement middleware** — Reject `/sendDARForm/*` requests without `X-Queue-Session` header | Not needed until all app versions are updated; old app backward compatibility takes priority | Sprint N+1 |
| 2 | **Admin dashboard** — View/manage active queue sessions | Nice-to-have; not required for MVP queue functionality | Sprint N+2 |
| 3 | **Priority queuing** — Allow certain supervisors to bypass the queue | Not requested; add only if field feedback demands it | Backlog |
| 4 | **Offline queue** — Queue reports locally when no network, auto-submit when online | Out of scope; reports already persist in Hive; manual retry is acceptable | Backlog |
| 5 | **GitHub Actions CI** — Automated migration + endpoint smoke tests on PR | Recommended by DevOps but out of scope for this sprint | Sprint N+1 |
| 6 | **Credential management** — Ensure DB secrets are managed via environment variables or `.env` files (not committed to git) | Pre-existing concern; flagged for good practice but separate from queue feature | Immediate / separate task |
| 7 | **Queue metrics/observability** — Track average wait time, queue depth, reaper events | Useful for tuning TTL and poll intervals after production rollout | Sprint N+1 |
| 8 | **Exponential backoff on poll** — Increase poll interval when position > 5 | Battery optimization; acceptable at 3s fixed for MVP (queue depth rarely > 5) | Sprint N+1 |

---

## Notes

- Items 1 and 5–7 are recommended for the next sprint after Submission Queue ships.
- Item 6 (credentials) should be addressed independently as a security best practice, not bundled with feature work.
