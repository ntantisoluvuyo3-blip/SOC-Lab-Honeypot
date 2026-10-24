Limitations

1. 8-pair gap: Hydra's header reports 24 planned tries (4 users x 6
   passwords, `-t 4`). The saved output's 24 numbered ATTEMPT lines confirm all
   24 were counted, but only the first 4 passwords (password, 123456, admin,
   root) were actually tried per user -- letmein and toor never appear for any
   user. This holds even for root, which had already succeeded by attempt 3.
   Confirmed from the log: per-user attempts tried = task count (`-t 4`).
   Unconfirmed: the exact mechanism inside Hydra that produces this. An earlier
   guess ("stderr not captured by tee") is withdrawn -- it doesn't explain a
   pattern this consistent.
2. Credential note corrected: the manual root session (`c3466e00ce2b`) used
   root/admin at 2026-10-20T21:17:17Z. An earlier running-log note said
   "lab123"; that was incorrect and has been corrected.
3. Two root/admin successes: session `ae4eaf50676` (20:50:42Z) is
   attributed to Hydra's automated run; session `c3466e00ce2b` (21:17:17Z) is
   the manual login, identified by its matching `whoami` command.
4. Default database: all accepted logins reflect Cowrie's `userdb.example`
   fallback (triggered because `etc/userdb.txt` does not exist), not a
   configured credential policy.
5. Timestamps: Cowrie logs UTC. Kali's local clock is EDT (UTC-4). No
   cross-log timing claims in this write-up depend on exact offsets.
