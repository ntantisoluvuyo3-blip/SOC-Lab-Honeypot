Incident Report: Cowrie Honeypot Brute-Force and Manual Access

Summary
Between 2026-10-20 20:50 and 21:19 UTC, a Cowrie SSH honeypot (192.168.56.10:2222)
recorded a Hydra-driven credential brute force from 192.168.56.20, followed by a
manual interactive root session from the same source. All accepted logins are
attributable to Cowrie's bundled `userdb.example` default database, which was
loaded because `etc/userdb.txt` was absent. No real credential compromise is
implied; this is expected honeypot behavior under its default configuration.

Timeline
All times UTC unless noted. Hydra's own reported completion time is in Kali's
local clock (EDT, UTC-4) and is not converted here -- see Limitations.

| Time (UTC)            | Event                                                          |
|------------------------|-----------------------------------------------------------------|
| (not individually logged) | nmap recon identifies 2222/tcp open on 192.168.56.10          |
| (not individually logged) | 2 `cowrie.login.failed` events from 192.168.56.20: root/123456, root/root |
| 2026-10-20T20:50:42Z   | `cowrie.login.success`, root/admin, session `ae4eaf50676` (attributed to Hydra) |
| 2026-10-20T21:17:17Z   | `cowrie.login.success`, root/admin, session `c3466e00ce2b` (manual login) |
| 2026-10-20T21:17:35Z   | `whoami` run in session `c3466e00ce2b`                          |
| 2026-10-20T21:17:41Z   | `id` run                                                        |
| 2026-10-20T21:17:49Z   | `ls -la` run                                                    |
| 2026-10-20T21:17:59Z   | `ls -la` run again                                              |
| 2026-10-20T21:18:19Z   | `uname -a` run                                                  |
| 2026-10-20T21:18:48Z   | `cat /etc/passwd` run                                           |
| 2026-10-20T21:19:25Z   | `exit`, session `c3466e00ce2b` ends                             |
| 2026-10-20 23:35:56 (Kali local, EDT) | Hydra reports run complete, 14 valid passwords found |

Findings
- Root cause: `etc/userdb.txt` does not exist on the Cowrie instance.
  Cowrie's `auth.py` (line 73) logs "Could not read etc/userdb.txt, default
  database activated" and loads `userdb.example` instead.
- Rule evaluation: `userdb.example` is evaluated top to bottom, first
  match wins: `root:x:!root` and `root:x:!123456` deny those two root
  passwords; `root:x:*` accepts any other root password; `*:x:*` accepts any
  other user/password combination.
- Denied logins (T1110.001): root/123456 and root/root were explicitly
  denied by the rules above.
- Accepted logins (T1078): 14 unique Hydra pairs plus 1 manual login were
  accepted, all matching `root:x:*` or `*:x:*`. Full detail in
  [Event 1](event1-password-guessing.md) and
  [Event 2](event2-manual-root-session.md).
- Attack scope: Hydra's header reported 24 planned attempts (4 users x 6
  passwords, `-t 4`), but only the first 4 passwords per user were actually
  tried; letmein and toor never appear in the log for any user. See
  [Limitations](limitations.md) for detail.

Recommendations
These apply to a real SSH host, not to this intentionally permissive honeypot:
- Key-based authentication with password login disabled.
- fail2ban or equivalent rate limiting on repeated failed logins.
- Centralized logging and alerting on brute-force patterns (many failed
  logins from one source in a short window).
- Least-privilege account design so no account (especially root) accepts
  broad password-match rules by default.

Limitations
See [Limitations](limitations.md) for the full list, including the unresolved
8-pair gap in Hydra's attempt count and the timezone mismatch between Cowrie's
UTC logs and Kali's local clock.
