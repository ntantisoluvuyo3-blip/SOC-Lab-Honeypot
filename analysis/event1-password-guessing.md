Event 1: Denied Password Guesses (T1110.001, Password Guessing)

Observed: 2 `cowrie.login.failed` events from 192.168.56.20: root/123456 and root/root.

Cause (confirmed in code): `etc/userdb.txt` does not exist, so Cowrie logged
"Could not read etc/userdb.txt, default database activated" (auth.py line 73) and
loaded `userdb.example`. That file is evaluated top to bottom, first match wins.
`root:x:!root` and `root:x:!123456` explicitly deny those two passwords. The next
rule, `root:x:*`, accepts any other root password.

Note: the denials come from explicit deny rules in Cowrie's default database.
They are not evidence of a real password policy.

Recommendations (for a real SSH host, not this honeypot):
- Key-based authentication with password login disabled
- fail2ban or equivalent rate limiting
- Alerting on repeated failed logins from one source
