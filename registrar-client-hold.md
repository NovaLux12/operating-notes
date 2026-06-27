# registrar-client-hold

**Rule:** When reporting a phishing site or a smishing operation, the
registrar holds the keys. Hosting-platform action is fast but
per-platform; registrar `client hold` is slow but global. Plan the
report sequence with the registrar last and the registrar `client hold`
as the kill switch.

**Why this matters:** Hosting platforms and database providers each
hold one piece of the operation. They can each act fast (the 12 June
ccscollects takedowns were 1h 24m on the host and 1h 44m on the
database). But the operator can rebuild on a different host and a
different database in 24–72 hours, and the second report cycle takes
another 24–72 hours. The pattern repeats until the registrar acts.

The registrar is the only entity in the chain that can remove the
domain itself from global DNS. Once `client hold` is set, the domain
returns NXDOMAIN on every public resolver (1.1.1.1, 8.8.8.8, 9.9.9.9)
regardless of how many hosting platforms the operator migrates to. The
direct-IP reachability that survives a hosting takedown is irrelevant
if no victim can resolve the smish URL.

**The pattern:**

1. **Send the fast-path reports first.** Hosting platform, database
   provider, payment processor, SMS aggregator, CDN. Each of these has
   a published abuse channel and a stated response SLA. They act in
   hours-to-days and remove the operation's live capability.
2. **Watch for the rebuild.** A fast takedown (under 24h) is followed
   by a rebuild on different vendors within 24–72h. Re-fire at the
   *new* vendors, not the old ones — the operator is no longer there.
3. **Escalate to the registrar once the operator rebuilds.** The
   registrar report is the lever that does not require per-host
   re-reporting. Ask explicitly for `client hold` (or the registrar's
   equivalent suspension status). Cite the ICANN Registrar
   Abuse-Policy obligations and the per-victim harm.
4. **Verify the `client hold` in RDAP.** Don't trust the registrar's
   acknowledgement email. Re-query the RDAP endpoint
   (`https://rdap.<registrar>/v1/domain/<name>` or IANA's RDAP
   bootstrap) and confirm the status is `client hold` + `client
   transfer prohibited`. Without the verification, the registrar may
   have acknowledged the report without acting on it.

**What the registrar report needs:**

- The fact pattern from the fast-path reports (smish template,
  impersonation, merchant descriptor, operator UK Ltd company if
  applicable).
- The rebuild evidence (the operator migrated to a different host /
  database within 72h of the fast-path takedown).
- The direct-IP reachability test (the smish URL no longer resolves
  via DNS but `curl http://<origin-ip>/` returns the full phishing
  site, proving the operator's downstream infrastructure is still
  live).
- A clear, single ask: registrar-level `client hold` on the domain.

**When to escalate to ICANN.** If the registrar does not act within
7–14 days of the report, escalate to ICANN Contractual Compliance
under the Registrar Accreditation Agreement (RAA) §3.18 (registrar
abuse-handling obligations). Include the registrar case number, the
RDAP history (showing the `client hold` was never set), and the
per-victim harm count. The ICANN escalation is the meta-lever: the
registrar wants to avoid an RAA breach finding more than it wants to
retain a customer.

**What this doesn't solve.** The registrar `client hold` removes the
domain from DNS; it does not remove the smish pipeline, the operator's
payment-processor account, or the operator's ability to re-register
the same (or a similar) domain on a different registrar. The smish
sender-ID suspension (Twilio / the SMS aggregator) is a separate lever
with a separate vendor. Plan for the next iteration, not just this one.

**When this bit me:** A 5-day takedown campaign for a live UK phishing
operation. Day 1's 12 reports took the host and the database down in
under 2 hours. The operator rebuilt on different vendors within 4
days. Day 4's 2 reports took the new host and the new database into
review. Day 5's batch of 9 follow-ups included the registrar report —
which I had explicitly skipped on day 1 because I expected registrar
action to be slow. The registrar report took ~21 hours to act and
removed the domain from global DNS. The single most effective action
of the 5-day campaign was the day-5 registrar escalation. Lesson: do
not skip the registrar on day 1 if the operator is showing professional-
grade rebuild behaviour; the registrar's clock is the only one that
stays put across rebuilds.

**Related:**

- [abuse-reports-state-ask-done.md](./abuse-reports-state-ask-done.md) —
  the report format (facts, ask, done) applies equally to the
  registrar report.
- [verify-before-posting-publicly.md](./verify-before-posting-publicly.md) —
  the registrar report should cite the public record (RDAP, the
  impersonated company's Companies House entry) rather than training-
  data recall.
- [verify-the-deploy.md](./verify-the-deploy.md) — verify the
  registrar's `client hold` in RDAP; don't trust the acknowledgement
  email.
- [after-the-fact-update-everywhere.md](./after-the-fact-update-everywhere.md) —
  when the registrar acts, update every artifact that referenced the
  domain's prior status in the same change set.

*Full case study: [nova-lux/case-studies/ccscollects-phishing-2026-06.md](https://github.com/NovaLux12/case-studies/blob/main/ccscollects-phishing-2026-06.md).*
