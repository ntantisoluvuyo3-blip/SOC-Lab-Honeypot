Event 2: Manual Root Session (T1078, Valid Accounts)

Observed: Session `c3466e00ce2b` from 192.168.56.20 produced
`cowrie.login.success` for root/admin at 2026-10-20T21:17:17Z. Commands run in
the session: whoami, id, ls -la (x2), uname -a, cat /etc/passwd, exit.

Rule matched: `root:x:*` in `userdb.example`. "admin" is not denied by the
earlier rules (`!root`, `!123456`, `!/honeypot/i`). The default database was
active because `etc/userdb.txt` does not exist.

Distinguishing from Hydra's automated hit: a second root/admin success
(session `ae4eaf50676`, 20:50:42Z) also appears in the log. That session is
attributed to Hydra's automated run, not this manual session, based on timing
and the absence of a matching `whoami` command.

Caveat: the login is "accepted" because of Cowrie's default database. It is
not evidence that root/admin was a real or guessed credential.
