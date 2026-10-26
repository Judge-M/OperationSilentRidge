# Operation Silent Ridge

A cooperative, defensive incident-response exercise. The current two-server
profile supports **up to eleven teams of five**. Each team receives three
shared desktops, for 33 desktops total. Participants investigate a fictional
breach using Autopsy, Wireshark, Cutter and Wazuh, coordinate work in IRIS, and
answer scored questions in CTFd. Every team works through the same 20 tickets
and 80 questions independently, while verified findings contribute to the
shared incident picture. Everything in the scenario is fictional; the
evidence, tools and teamwork are real.

This repository contains the full event: the authored scenario and evidence,
the tooling that builds and runs it, the participant materials, and the
facilitator guides.

## Running the event? Start here

**Open [Run Operation Silent Ridge](docs/START-HERE.md) and follow it from top
to bottom.** It assumes the operator has never seen this repository. It covers
the server, network, registration, projector brief, live controls, closing
brief, backup and shutdown in event order.

The facilitator keeps START-HERE and the two room decks open. The designated
operator may also keep the command sheet and runbook available for recovery:

| What you need | File |
|---|---|
| One complete beginner checklist | [Run Operation Silent Ridge](docs/START-HERE.md) |
| Operator-only installation and recovery commands | [Operator command sheet](docs/event-day-commands.md) |
| Opening room brief with embedded intro video | [Opening PowerPoint](docs/event-day-deck/opening-brief.pptx) or [LibreOffice ODP](docs/event-day-deck/opening-brief.odp) |
| Closing room brief, outro and winner reveal | [Closing PowerPoint](docs/event-day-deck/closing-brief.pptx) or [LibreOffice ODP](docs/event-day-deck/closing-brief.odp) |
| Troubleshooting during play | [Operator runbook](docs/runbook.md) |

For the two Windows event servers, install the
[one-time boot automation](docs/windows-boot-automation.md) before event day.
It starts Docker Desktop on both machines, waits for the worker from the
primary, and safely runs the existing `up`/`start` lifecycle. After its reboot
test passes, facilitators do not run server commands: power on the worker and
primary, present the opening deck, and tell participants to begin.

The current reusable event shape is
[`example-dynamic-two-host-11-team.json`](deployment/profiles/example-dynamic-two-host-11-team.json)
plus its matching private-host template,
[`two-host-11-team.local.example.json`](deployment/examples/two-host-11-team.local.example.json).
It declares exactly 33 desktop slots: 14 on the primary and 19 on
`worker-02`. Eleven teams are representable and pass the declared capacity
model; they are **not performance-certified** until the human/load rehearsal
passes on the event hardware.

Do not rebuild or reinstall the event servers on event day. Use the already
verified event installation, keep runtime secrets outside the repository, and
use the two room-wide brief decks. Use PPTX in PowerPoint and ODP in
LibreOffice Impress. The newest public bundle is older than the current event
iteration; the release limitation is explained under [Project status](#project-status).

### Participant start

Connect to the event network and open:

```text
https://ctfd.cyberexperts.org/silent-ridge/register
Enrollment code: SILENT-RIDGE
```

One captain registers the team and bookmarks the private retrieval page. That
page contains the team's logins and three Guacamole desktop names. Use IRIS,
CTFd and Wazuh in normal laptop browser tabs. Use Guacamole only for Autopsy,
Wireshark, Cutter and the desktop file manager.

## What participants experience

Each five-person team receives three isolated, container-hosted Linux desktops
through Guacamole—Forensics A, Forensics B, and Analysis. Everything else
happens in the participants' own browser tabs; nothing is installed on their
laptops.
Teams:

1. Claim up to three team-local investigation tickets at once in IRIS. Other
   teams can investigate the same tickets independently.
2. Investigate with the desktop tools: disk images in Autopsy, network
   captures in Wireshark, a suspicious binary in Cutter, logs in Wazuh.
3. Answer the ticket's questions in CTFd. An unassisted correct answer scores
   four points and posts the finding back into the shared IRIS case
   automatically. Help levels 1 and 2 are free; unlocking the explicit
   walkthrough applies an exact 75% cost, leaving one point for that question.
   Every team's work still helps everyone else's picture of the incident.
4. Every team works through all 20 tickets and 80 questions. Later phases unlock
   only when that team completes the required earlier phase.

The current event schedule is a six-hour hard stop: 25 minutes of orientation,
280 minutes of investigation, a 10-minute operational update, a 15-minute
break, and a 30-minute facilitated after-action review. Only a representative
human rehearsal can establish whether teams finish all 80 questions in that
window.

## What you need to host it locally

Smaller profiles can run on one machine. The current eleven-team shape uses a
primary plus one [desktop worker](docs/multi-host-desktops.md); browser services
remain centralized on the primary while the worker carries 19 Guacamole
desktops. Sizing is based on live measurement of the real stack, not guesswork:

- **32 GB RAM minimum for a smaller run; the current eleven-team event uses two
  64 GB hosts.** Measured on the running stack:
  central services use ~6 GiB; each team desktop idles at ~150 MiB and peaks
  at ~2 GiB with the Autopsy case open and its text index loaded. Thirty-three
  desktop containers are available for eleven teams; actual simultaneous loaded-tool use,
  central services, and the host OS must remain inside the verified host budget.
  Desktop containers carry a 4 GB memory *limit*, but a limit reserves
  nothing — the host only commits what is actually touched.
- **Two 20-thread hosts for the current eleven-team shape**, gigabit LAN for
  participant browsers. Smaller rehearsals may use one host after validating a
  smaller profile.
- **100 GB free NVMe SSD** — measured: ~19 GB of container images, ~14 GB for
  the offline bundle, plus build cache and runtime growth
- **Docker** (Docker Desktop on Windows, or Docker Engine on Linux)
- **Python 3.11+** and **Git with Git LFS** for the build host

A two-team pilot runs comfortably on 16 GB RAM (measured: ~8.4 GiB for the
whole stack with two desktops, one under Autopsy load).

## Quick start

### Existing event servers

Follow [Run Operation Silent Ridge](docs/START-HERE.md). For the normal tested
path, power on the worker first and then the primary, sign in if the configured
boot mode requires it, wait for the registration page, and present the opening
deck. Facilitators run no server commands.

### Fresh installation

Do not use `v0.10.0-rehearsal` as though it were the current event build. It
predates the 11-team self-registration and boot-automation iteration. A fresh
host needs a new release cut from current `main`, followed by the complete
[cold-machine installation](docs/event-machine-cold-start.md). Existing
verified live installations should remain in place until their replacement has
passed the same cold-install and rehearsal gates.

### Developer/source checkout

Building from source is an **online preparation/developer path**. It may
download base images, operating-system packages, and pinned tool archives even
when Docker already has the base image cached. Cache preference does not make a
source build suitable for an air-gapped event host; the complete release bundle
is the supported offline event path. Note that
`build --component all` produces only the container images — the content
assets (evidence tree, Autopsy case template, Wazuh config, release vault)
come from the release bundle or the content pipeline documented in
[`docs/handoff/BUILD-FIRST.md`](docs/handoff/BUILD-FIRST.md). To build
images:

```bash
git clone https://github.com/Judge-M/OperationSilentRidge.git
cd OperationSilentRidge
git lfs pull                                # fetch the evidence assets

python -m ridge.deploy doctor               # check host prerequisites
python -m ridge.deploy build --component all   # build images (downloads dependencies)
python -m ridge.deploy verify-build         # verify the built artifacts
```

Then create a deployment profile. For the current two-host event, copy
`deployment/profiles/example-dynamic-two-host-11-team.json` and pair it with
`deployment/examples/two-host-11-team.local.example.json`. For a smaller
single-host rehearsal, start from `deployment/profiles/example-two-team.json`.
Adjust the documented host-specific fields and bring the event up:

```bash
python -m ridge.deploy up     --profile path/to/profile.json --runtime path/to/runtime
python -m ridge.deploy status --profile path/to/profile.json --runtime path/to/runtime
python -m ridge.deploy start  --profile path/to/profile.json --runtime path/to/runtime
```

The legacy fixed-roster path may use `up --teams N`; it is separate from the
event-day self-registration path and does not define the dynamic-team ceiling.
Dynamic capacity comes from `registration.max_teams`, the exact desktop-slot
set, and validated host/worker capacity.

For the current event-day self-service roster, copy
`deployment/profiles/example-dynamic-two-host-11-team.json`. Do not pass
`--teams`.
After `start`, send teams to
`https://ctfd.cyberexperts.org/silent-ridge/register` and use the static room
enrollment code `SILENT-RIDGE`. The opening deck and facilitator runbook show
both values; no separate registration open/close command is required.
The landing page contains the initial brief; each registration reserves five
seats and three GUI-only desktops, starts those three slots through the
host-side allocator, creates the CTFd/IRIS/Guacamole identities, and returns a
bookmarkable handout with all URLs, credentials, and exact Guacamole connection
names. The runtime registration database is private and is not part of the
repository.

`up` builds out the entire stack and stops at a verified, paused state —
nothing is visible to participants until you explicitly `start`. The `runtime`
directory is private: it holds the event's secrets, state, and operator-only
recovery inventories. Participants receive credentials only through their
private registration page and downloadable handout. On Windows, `ridge.ps1`
wraps the same commands (`.\ridge.ps1 prepare / up / status / start ...`).

In the event's trusted public-TLS mode, participants use these port-443 names.
The event DNS maps them to the LAN address, and the reverse proxy routes each
name to its internal service:

| What | Where |
|---|---|
| Team desktops (Guacamole) | `https://guac.cyberexperts.org` |
| IRIS incident case | `https://iris.cyberexperts.org` |
| CTFd questions and score | `https://ctfd.cyberexperts.org` |
| Wazuh dashboard | `https://wazuh.cyberexperts.org` |

Fresh runtimes generate random facilitator, participant, database, and service
credentials. Retrieve them only from the private runtime or a team's private
registration handout; never commit, publish, or substitute memorable defaults.
Existing runtimes retain their generated credentials when upgraded. The
generated-CA/IP-and-port path remains an offline fallback; only that fallback
requires the participant trust steps in
[`docs/participant-ca-trust.md`](docs/participant-ca-trust.md).

## Running the event

On event day, the [presenter landing page](docs/presenter-landing.html) is an
offline index for the decks, participant URLs, operator documents, run of show
and quick reference. [START-HERE](docs/START-HERE.md) remains the authoritative
beginner checklist.

The [event-day command sheet](docs/event-day-commands.md) is the
cold-machine → running-event → teardown checklist with exact commands.
The [opening room brief](docs/event-day-deck/opening-brief.pptx) introduces the
scenario and workflow, plays the introduction, then directs captains to the
participant landing page. The [closing room brief](docs/event-day-deck/closing-brief.pptx)
plays the final assessment and provides the winner-reveal slide. Native
LibreOffice copies are stored beside both PowerPoint files.
The [facilitator closing brief](docs/facilitator-closing-brief.md) is the
canonical ending used after T20. Completion automatically reveals the final
assessment and containment actions in CTFd and publishes the same assessment
to IRIS. The facilitator then plays or reads the outro once for the entire
room; participant-controlled media playback is intentionally unavailable. The
[approved scripts and production handoff](docs/handoff/audio-briefs/README.md)
remain with the accepted source videos.

```bash
python -m ridge.deploy pause   --profile ... --runtime ...   # pause the clock
python -m ridge.cli --state path/to/runtime/state/state.sqlite announce --operator EXCON --text "..."
python -m ridge.cli --state path/to/runtime/state/state.sqlite recover T03 --team team-03 --generation N --operator EXCON --reason "..."
```

Facilitator materials (solutions, coaching and AAR guides) live in
`facilitator/`; participant worksheets in `participants/`. The
[operator runbook](docs/runbook.md) is the one-page event-day reference with
recovery appendices; see `docs/expanded-operations.md` for the deeper guide.

## Backup, restore, teardown

```bash
python -m ridge.deploy backup  --profile ... --runtime ...   # verified recovery set
python -m ridge.deploy down    --profile ... --runtime ...   # stop; keeps all data
python -m ridge.deploy down --volumes --profile ... --runtime ...   # full wipe
python -m ridge.deploy restore --from <recovery-set> --profile ... --runtime <empty-dir>
```

`backup` produces a checksummed recovery set (databases, state, evidence,
team workspaces, Wazuh index, encrypted secrets). `down --volumes` is
destructive and **refuses to run unless a verified backup exists**. Restore
onto an empty runtime directory replays the complete event — this exact
backup → wipe → restore → keep-playing cycle has been drill-tested live.

## Project status

The local event is working software: the full lifecycle (build → provision →
run → pause → backup → wipe → restore) has been exercised against the real
stack, and a scripted walkthrough solved all 80 questions end-to-end with
findings and points landing in IRIS and CTFd. The newest published offline
bundle is
[`v0.10.0-rehearsal`](https://github.com/Judge-M/OperationSilentRidge/releases/tag/v0.10.0-rehearsal).
It supersedes the `v0.9.x-drill` bundles, but it predates the current 11-team
self-registration and boot-automation iteration. Do not replace a verified
live installation with that older rehearsal bundle. A fresh deployment needs a
new release built from current `main` and verified with
[`docs/event-machine-cold-start.md`](docs/event-machine-cold-start.md).
Event-day materials are ready: the operator command sheet
(`docs/event-day-commands.md`) and the separate opening and closing room decks
(`docs/event-day-deck/`).

**Picking up development (human or agent): start at
[`docs/handoff/NEXT.md`](docs/handoff/NEXT.md)** — its "Current position"
section says what is done, what is in flight, and the single next action,
with links to receipts and evidence. Task cards live alongside it in
`docs/handoff/tasks/`.

The remaining certification and release work is tracked in
`docs/handoff/NEXT.md`:

- Eleven-team human/load rehearsal on the actual two-host event hardware
- A current offline release, cold-install validation, operator freeze and dress
  rehearsal
- A powered-off reboot test proving the configured startup guarantee
- A beginner rehearsal to validate the six-hour schedule with real people
- AWS deployment option (AMI, cloud fencing, teardown) — deferred until the
  local release is proven

## Repository map

| Path | What it is |
|---|---|
| `participants/` | Participant worksheets and handover notes |
| `facilitator/` | Operator runbook, solutions, coaching, AAR guides |
| `expanded/` | Scenario authoring and content tooling |
| `ridge/` | The event platform: state core, deployment, CLI |
| `deployment/` | Container builds, compose files, example profiles |
| `docs/` | Architecture, operations, and planning documents |
| `docs/event-day-commands.md` | **Run the event**: cold-machine → teardown checklist |
| `docs/event-day-deck/` | **Brief the room**: separate opening and closing decks with PowerPoint and LibreOffice copies |
| `docs/presenter-landing.html` | **Event-day hub on the presenter PC**: deck + URLs + operator docs, offline |
| `docs/handoff/` | **Start here when picking up work**: `NEXT.md` (current position), receipts, evidence, and task cards |
| `tests/` | Automated test suite (`python -m unittest discover -s tests`) |
| `app/`, `admin/`, root `Dockerfile` | Retired rehearsal portal, kept only as a test fixture — not part of the event |

All scenario content and answers are public by design (large evidence files
via Git LFS). Credentials, rosters, runtime directories and backups are
private and must never be committed.

## License

Project-authored source code and documentation are licensed under the
[Apache License 2.0](LICENSE). Third-party components, container images,
evidence and assets retain their respective licenses and notices; see
[NOTICE](NOTICE).
