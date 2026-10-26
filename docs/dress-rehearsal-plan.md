# Dress rehearsal + certification session plan (event hardware)

One chronological checklist for the rehearsal day on the real event machine.
It sequences three things that must all pass before the event is declared
ready: the **F03 certifying capacity run**, the **F06 dress rehearsal and
recovery drill**, and the **release freeze**. It delegates procedure detail
to the existing runbooks and only adds order, gates, and evidence capture.

Hard rules (from the F06 card, non-negotiable):

- Use **only** a verified bundle built from current `main` and the private profile on the event
  machine — no image builds, no manual database IDs, no SQL patching, no
  downloads during setup. Anything missing from the bundle fails the drill
  and gets fixed in the bundle, not on the machine.
- Block internet access during the local/offline phases.
- No waiving a failed gate because event day is close. Fix, rerun the
  affected phase, refreeze if artifacts changed.
- A beginner rehearsal with real people is a **separate** gate; this
  technical rehearsal cannot validate the six-hour schedule or learning
  outcomes.

## Prerequisites before the session

- [ ] The release tag, distribution manifest and offline bundle all identify
      the same current source commit. Do not certify the historical
      `v0.10.0-rehearsal` bundle; it predates dynamic eleven-team registration
      and boot automation.
- [ ] **Current bundle assembled** on the build host:
      `python scripts/assemble_offline_store.py --store work/offline-store --manifest work/offline-manifest.json --evidence work/evidence-public --case-template work/case-template --wazuh-config work/n1-run/wazuh-config --release-vault work/release/controller/releases` then
      `python -m ridge.bundle work/offline-store work/offline-manifest.json work/offline-bundle --allow-incomplete --max-part-bytes 2147483648`
      and publish as a new pre-release from current `main`.
- [ ] Event profile prepared (`docs/event-day-commands.md` §1) with the
      real `<LAN-IP>`; runtime directory chosen (referred to as `R` below).
- [ ] Presenter landing page open on the facilitator's laptop — open
      `docs/presenter-landing.html` (offline, single file; deck, participant
      URLs, operator docs, run-of-show, runbook quick reference in one place).
      Projector/screen available for the briefing walkthrough.
- [ ] A stopwatch/notes doc open — measured times are acceptance evidence.

## Phase 0 — Cold install (timed)

Follow `docs/event-day-commands.md` §0–§2 exactly, starting from a machine
state with no Docker images and no repo checkout. Record:

- [ ] wall-clock time from "nothing installed" to `offline_install` receipt
- [ ] asset extraction completed with the four `tar` commands, unassisted
      by anyone who built the system (the point of the cold-read)
- [ ] `local.json` written from the §2 template without consulting other docs
- [ ] any step where the docs were ambiguous — each one is a doc bug; file it

**Gate 0:** install succeeds with zero improvisation. Failure → fix docs or
bundle, restart phase 0.

## Phase 1 — Bring-up and F03 certifying run (~1 hour)

Use the load-test procedure and acceptance thresholds in
`docs/f03-rehearsal.md`, but use the current dynamic profile and the commands
below; the older fixed-roster startup example in that document is historical.

- [ ] `up` with the current dynamic profile and no `--teams` argument (timed);
      re-run `up` on "services not ready" — that
      is reconcile, not failure
- [ ] `status` shows `event_ready: true`
- [ ] `start` the exercise
- [ ] **Cover the answer write path live** (the dev-box dress run only
      exercised reads): before starting the harness, claim one ticket in
      IRIS as one team so answer POSTs render
- [ ] Run the harness: 11 teams × 3 sessions, 1800 s
- [ ] Manual desktop usability pass in parallel: 2–3 desktops via
      Guacamole, launch Autopsy, open the prepared case, run a search
- [ ] Report says `CERTIFYING`; p95 < 2 s; outbox flat/draining; peak
      working set ≤ 80% of host RAM — save the sanitized report as F03
      evidence

**Gate 1 (F03):** the run self-labels CERTIFYING and all acceptance targets
hold. Failure → capacity model or tuning task, rerun.

## Phase 2 — F06 functional rehearsal (~90 minutes)

All on the same stack, still offline:

- [ ] **Briefing walkthrough**: a facilitator who has not run the system
      opens the presenter landing page (`docs/presenter-landing.html`),
      projects the separate opening and closing briefs, verifies both video
      slides with room audio, and follows the participant landing page against
      the real stack. Confirm its displayed URLs match the running services.
      Any mismatch is a deck bug or a stack bug — file it.
- [ ] **Registration/handout dry-run**: register a team with the room code,
      bookmark its private retrieval page, download its labelled handout and
      confirm a cold reader can find every login kind from those two artifacts
      alone. Keep `R/secrets/team-credentials.json` operator-only.
- [ ] **Ticket flow**: play at least one full ticket end-to-end (claim →
      investigate → answer → finding lands in IRIS → next ticket unlocks)
- [ ] **Repeated up**: `down` (keep data), `up` again, confirm the event
      resumes where it left off
- [ ] **Pause/resume**: `pause` during play, announce via
      `ridge.cli announce`, resume with `start`
- [ ] **Desktop disconnect/reconnect**: kill a Guacamole session
      mid-work, reconnect, confirm the desktop and case are intact

## Phase 3 — Recovery drill (lost central host, timed → RTO/RPO)

- [ ] `backup` → verified recovery set (record completion time = RPO point)
- [ ] Make one more accepted answer (so progress exists past the backup)
- [ ] Attempt `down --volumes` — it must **refuse** (stale backup, PR #49
      semantics); re-`backup`, then `down --volumes` succeeds
- [ ] `restore --from <recovery-set>` into an empty runtime; `start`;
      confirm scores and findings reconcile exactly with the pre-wipe
      scoreboard (screenshot before/after)
- [ ] `fence` the old runtime via the **CLI** (not the method) — this is
      the regression PR #49 fixes; confirm the site refuses writes
- [ ] Record measured RTO (wipe → playing again) and RPO (backup age)

**Gate 2 (F06):** every phase passes with no improvisation, score/finding
reconciliation is exact, and setup/RTO/RPO numbers are recorded.

## Phase 4 — Freeze

- [ ] If any artifact changed during fixes: reassemble the bundle, tag a
      new immutable candidate, and rerun the affected phases against it
- [ ] Otherwise tag the rehearsal bundle as the event release candidate
- [ ] Archive the session evidence: F03 report, timings, reconciliation
      screenshots, exception list — under `docs/handoff/evidence/`
- [ ] Write the F03 receipt and F06 receipt per EXECUTION.md and update
      `docs/handoff/NEXT.md`

## Go / no-go summary (fill in on the day)

| Gate | Result | Evidence |
|---|---|---|
| 0 Cold install, zero improvisation | | timing + notes |
| 1 F03 CERTIFYING run | | report |
| 2 F06 functional + recovery | | checklist + reconciliation |
| Freeze: release tag | | tag name |

Any unchecked gate = not ready; the fix is a narrow follow-up task, not a
waiver.
