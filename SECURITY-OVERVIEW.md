# Security overview

Kitchen Portal runs on a public hostname, and the roster of names is visible on its sign-in page. So the design assumes the attacker is **someone who knows the house**: a resident or a guest who knows everyone's name and has physical access to the building.

This page summarizes the controls. Implementation details are private.

| Threat | Control |
|---|---|
| Claiming someone else's account | One-time **setup codes** issued by the manager: stored as a keyed hash, single use, expire in 7 days. Knowing a name is not enough. |
| Guessing a PIN | **Escalating lockout** per account and per IP, persisted so a restart doesn't reset it. Six-digit PINs that reject runs and repeats, **scrypt**-hashed. |
| Stolen or stale sessions | HMAC-signed cookies carrying a **credential version**, checked on every request. A PIN reset, a PIN change or "sign out everywhere" ends every older session immediately. |
| The manager account | **scrypt** password plus **TOTP** authenticator codes (RFC 6238, replay-protected). Rotating either secret ends existing sessions. |
| The kitchen kiosk | **Paired** with a short-lived code, then an httpOnly cookie whose secret is stored hashed. Every device is listed and revocable. No token ever appears in a URL. |
| The dining-room TV | A separate **read-only key** for two endpoints that return names and counts only, never dietary information. |
| Allergy data | Sent only to the chefs' paired kiosk and the manager. Never shown on the TV, in public feeds or to other residents. |
| Network exposure | The app binds to localhost on the server. The only way in is an **outbound-only Cloudflare Tunnel**, with TLS and HSTS at the edge. Admin access is over a private Tailscale network. |
| Everything else | Content Security Policy, `frame-ancestors 'none'`, nosniff, strict referrer policy, no CORS, signed calendar-feed URLs, formula-safe CSV export, and systemd sandboxing. |

**Accountability.** Every sign-in, failure, lockout, code issued, device paired and export taken is written to an append-only audit log.

**Resilience.** Nightly AES-256-GCM encrypted backups with integrity and foreign-key checks and SHA-256 manifests, verified after writing. A scripted drill backs up, destroys, restores and compares every row, and confirms that a wrong passphrase and a tampered archive are both refused.

---

Copyright © 2026 Roman Sidney Ghorbani. All rights reserved.
