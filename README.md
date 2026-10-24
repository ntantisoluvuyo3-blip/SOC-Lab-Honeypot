SOC Lab: Cowrie Honeypot Brute-Force Analysis

A small home lab pairing a Cowrie SSH honeypot (target) with a Kali attacker VM
running Hydra, analyzed as a SOC exercise: triaging the honeypot's JSON log,
confirming root cause in the Cowrie source, and documenting findings and
limitations.

Environment
- Target: Cowrie honeypot VM, 192.168.56.10:2222 (SSH)
- Attacker: Kali VM, 192.168.56.20
- Tooling: nmap (recon), Hydra (credential brute force), jq (log analysis)

What happened
1. nmap identified 2222/tcp open (SSH) on the target.
2. Hydra ran a 4-user x 6-password brute force against the honeypot.
3. A manual SSH session was also run against the honeypot as root.
4. Cowrie's JSON log was analyzed with jq to reconstruct both the automated
   and manual activity.

Key finding

Cowrie's `etc/userdb.txt` does not exist in this setup, so Cowrie fell back to
its bundled `userdb.example` default database. That file's deny/allow rules
(evaluated top to bottom, first match wins) explain every accepted and denied
login in this lab -- this was Cowrie's default behavior, not a configured
weak-credential policy.

Analysis
- [Event 1: Denied password guesses](analysis/event1-password-guessing.md)
- [Event 2: Manual root session](analysis/event2-manual-root-session.md)
- [Limitations](analysis/limitations.md)
- [Incident report](analysis/incident-report.md)

Notes

All data in this repo was generated in this lab. No external or real-world
incident data is included.
