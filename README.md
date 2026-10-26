# XR-validation-protocols

[![License: CC0-1.0](https://img.shields.io/badge/License-CC0_1.0-lightgrey.svg)](https://creativecommons.org/publicdomain/zero/1.0/)
[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.23668875.svg)](https://doi.org/10.5281/zenodo.23668875)

A community registry of sensor validation procedures for XR hardware.
One CSV, reviewed via pull request.

## Contents

- [The problem this registry solves](#the-problem-this-registry-solves)
- [What this is](#what-this-is)
- [How to read a row](#how-to-read-a-row)
- [Column schema](#column-schema)
- [The one rule worth stating clearly](#the-one-rule-worth-stating-clearly)
- [What's NOT here](#whats-not-here)
- [Quality bar](#quality-bar)
- [How to contribute](#how-to-contribute)
- [Stability and Protocol IDs](#stability-and-protocol-ids)
- [Corrections](#corrections)
- [Connection to ResXR](#connection-to-resxr)
- [Citing this registry](#citing-this-registry)
- [License](#license)

## The problem this registry solves

If you do research with VR/AR headsets, you depend on sensor outputs
that vendors describe in documentation that is incomplete, occasionally
wrong, and never written with research-grade reproducibility in mind.

Most labs rediscover the same sensor quirks independently — finding
that a documented sampling rate is actually different, that a "valid"
flag stays set when it should drop, that a blink event lives in a
stream nobody would think to look at. None of these findings make it
into a vendor changelog. They are empirical knowledge that accumulates
inside individual labs, gets lost between PhD students, and is
rediscovered elsewhere months later.

The point of this registry is to write that knowledge down once, pin
it to a specific device + SDK + firmware, and share it.

## What this is

A single CSV file, [`protocols.csv`](protocols.csv), where each row
documents one validation procedure for one sensor on one device:

- **What the tester physically does** (the procedure)
- **What the SDK flags/values should report** (expected behavior)
- **What they actually reported when someone ran it** (outcome)
- **Under which SDK and firmware version**

That is the entire registry. GitHub renders the CSV as a searchable
table; contributors edit it directly through the GitHub web editor and
submit changes via pull request. No build step. No web app. No code.

### Seed protocols

The registry holds 17 protocols from empirical stress-testing of
the Meta Quest Pro under Meta XR SDK v78, firmware v79–v83 (contributed
by ResXR / Schonberg Lab TAU, 2025; 14 at launch, 3 added in v1.1.0),
covering Hand, Eye, Face, and Head modalities. They are not a complete map — they are starting
points the community is invited to extend.

## How to read a row

Take this seed row as a worked example:

| Column | Value |
| --- | --- |
| `Platform` | Meta |
| `Hardware` | Quest Pro |
| `Modality` | Hand |
| `Test` | Occlusion |
| `Procedure` | Move one hand fully behind back outside the headset FOV while keeping the other hand in the baseline position (arm extended forward at shoulder height, palm down, fingers fully extended); test each hand separately |
| `Expected Behavior` | `HandTracked` drops to 0; position data freezes at last known value |
| `Outcome` | `HandTracked` drops to 0; hand position reads 0 rather than freezing at its last value. |
| `SDK / Firmware` | Meta XR SDK v78 / fw v79-v83 |
| `Data URL` | https://doi.org/10.5281/zenodo.23668483 *(the seed rows' raw recordings on Zenodo)* |
| `Notes` | *(empty in this row; optional caveats)* |
| `Contributed By` | ResXR / Schonberg Lab TAU (2025) |
| `Date of Test` | `2025-12-09` *(YYYY-MM-DD, the date the procedure was physically run)* |
| `Protocol ID` | `META-QUESTPRO-HAND-OCCLUSION` |

Read left-to-right and you have: who, on what device, doing what,
expecting what, observing what, under which software version, with raw
data over there if you want to verify, contributed by whom, citable by
a stable ID.

## Column schema

Every column exists for a reason. The "Why" column below is the
rationale — it is the part to read first if you are deciding whether
this schema fits your work.

| Column | What to put | Why this column exists |
| --- | --- | --- |
| `Platform` | Vendor — Meta, Pico, Apple, HTC, … | Highest-level grouping. SDKs are vendor-scoped. |
| `Hardware` | Specific device — Quest Pro, Quest 3, Neo 3 Pro Eye, … | Same vendor, different sensor stacks, different quirks. |
| `Modality` | One of: Hand, Eye, Face, Body, Head | Researchers think in modalities ("I am validating eye tracking"), not vendor part numbers. |
| `Test` | Short, human-readable name — Occlusion, Baseline, Fist | Lets you talk about "the occlusion test" in prose without quoting an ID. |
| `Procedure` | Plain English: what the tester physically does | The procedure outlives any specific SDK. It is reproducible by anyone wearing the headset. |
| `Expected Behavior` | What the SDK flags/values **should** show, per spec or vendor docs. Use exact flag names — e.g. `HandTracked = 0` | The vendor's stated contract. |
| `Outcome` | What the SDK flags/values **actually** showed when someone physically ran the test | The empirical observation. Often diverges from `Expected Behavior` — that divergence is the whole point of the registry. Leave blank if not yet physically run. |
| `SDK / Firmware` | SDK and firmware version — e.g. `Meta XR SDK v78 / fw v79-v83` | Behavior changes across versions; without this, nothing reproduces. |
| `Data URL` | Optional link to a deposit (OSF, Zenodo, Figshare) holding raw recordings | Lets people verify your outcome from data, not just take your word for it. Zenodo gives a citable DOI. |
| `Notes` | Optional caveats, bilateral testing requirements, etc. | Things that don't fit a structured column. |
| `Contributed By` | Lab or person — e.g. `ResXR / Schonberg Lab TAU (2025)` | Provenance. Year matters because the SDK landscape moves. |
| `Date of Test` | Date the procedure was physically run, in `YYYY-MM-DD` format — e.g. `2025-12-09` | Pins the row to a moment in time. Critical when firmware updates silently change sensor behavior between two runs of the same procedure, or when two labs re-run the same procedure months apart. |
| `Protocol ID` | Unique slug **per procedure** — `PLATFORM-HARDWARE-MODALITY-TEST`, uppercase, hyphen-separated, spaces inside Hardware removed. e.g. `META-QUESTPRO-HAND-OCCLUSION` | A stable identifier for the procedure itself, citable in papers. Multiple rows may share the same `Protocol ID` when different labs or dates re-run the procedure; rows are uniquely identified by the combination of `Protocol ID`, `Contributed By`, and `Date of Test`. |

## The one rule worth stating clearly

**The `Procedure` column must describe a physical human action, not a
software state.**

- **Correct:** "Move your hand fully behind your back."
- **Wrong:** "Set `HandTracked` to 0."

The procedure is the load-bearing part of every row. SDKs change,
flags get renamed, vendors update documentation — but the physical
action ("hand behind back") is stable. A protocol whose procedure says
*"set `HandTracked` to 0"* is unreproducible: it presumes the flag
exists with that exact name, and that you have a way to set it
directly. Neither of those survives an SDK update.

What the flags should fire goes in `Expected Behavior`. What they
actually fired goes in `Outcome`. The `Procedure` describes what a
human does in the physical world.

## What's NOT here

To keep the registry focused, the following are deliberately out of
scope:

- **Raw sensor recordings.** Those are large binaries; they belong in
  OSF or Zenodo deposits, with the deposit URL referenced from the
  `Data URL` column. Git is not the place for multi-megabyte recording
  archives, and binary blobs in git history are painful to remove
  later.
- **Per-protocol markdown files.** The CSV is the registry. Adding a
  separate file per protocol would fragment the data and make
  cross-cutting queries (e.g. "all occlusion tests across all
  hardware") harder, not easier.
- **Protocols nobody has actually run.** Inferring expected behavior
  from documentation is the failure mode this registry was built to
  expose. Do not submit a row unless you, personally, ran the test on
  the listed hardware and SDK.

## Quality bar

A submission is in scope when:

1. You **physically ran the test yourself** on the listed device,
   under the listed SDK and firmware. No second-hand reports, no
   "the docs say…", no "I expect…".
2. The `Procedure` is a physical action a stranger could reproduce
   without insider knowledge of your lab's setup.
3. SDK flag names are spelled **exactly** as they appear in the SDK
   source, not paraphrased. `HandTracked` is not the same string as
   `Hand_Tracked` and downstream code will not find the latter.
4. If you observed flag or value behavior that contradicts the vendor
   documentation, **say so** in `Notes`. That is the kind of finding
   the registry exists to surface — leaving it implicit defeats the
   purpose.
5. If you have raw recordings, deposit them on OSF or Zenodo and put
   the URL in `Data URL`. Zenodo gives you a DOI you can also cite
   from your paper.
6. `Contributed By` includes a lab or institution and a year. Bare
   names without affiliation make the row hard to follow up on; bare
   affiliations without a year make the row hard to age-check.

## How to contribute

1. **Fork** this repo on GitHub.
2. **Open [`protocols.csv`](protocols.csv) in the GitHub editor.**
   GitHub renders the file as a searchable table; click the pencil icon
   to switch to edit mode.
3. **Add one row per test procedure** following the column schema
   above.
4. **Construct the `Protocol ID`** using the format above. If your
   test name collides with an existing one within the same modality,
   pick a more specific test name.
5. **Submit a pull request.** The template in `.github/` will walk
   you through the checklist.

That is the entire workflow. No tooling, no environment, no setup.

## Stability and Protocol IDs

`Protocol ID` is the registry's permanent identifier. Once a row is
merged and an ID is out in the world — cited in a paper, referenced
in code, mentioned in a Supplementary Table — **do not rename it**.

A few stability rules follow from that:

- **Don't rename a Test once its ID is published.** If the test
  itself evolves, add a *new* row with a new `Protocol ID` and leave
  the old one in place. Old citations should still resolve.
- **One row = one run.** Every row is a single observation tied to a
  specific lab, date, and SDK/firmware. When *any* of those change —
  a different lab runs the same procedure, the same lab re-runs it
  on a different date, or the firmware version moves — **add a new
  row**, do not edit the existing one. This preserves an audit trail
  of who observed what, when, on which software.
- **Edits are reserved for fixing genuine mistakes** — typos, wrong
  flag names, missing notes — not for updating the empirical record
  over time. The empirical record grows by accumulating rows, not by
  rewriting them.

Protocol IDs encode the procedure they identify — vendor, device,
modality, test — which means a reader who only has the ID can
reconstruct what procedure it refers to without looking it up. The
full identity of a row is the combination of `Protocol ID`,
`Contributed By`, and `Date of Test`.

### Example: how a Protocol ID accumulates rows

Take `META-QUESTPRO-HAND-OCCLUSION`. The seed row was contributed by
ResXR / Schonberg Lab TAU on `2025-12-09` under Meta XR SDK v78,
firmware v79–v83. A year later, suppose Lab B at another institution
re-runs the same procedure on a Quest Pro under firmware v95 and
observes that `HandTracked` still drops to 0 but hand position now
freezes at its last value instead of reading 0.

They add a *second* row with the same `Protocol ID`, their own
`Contributed By`, `Date of Test` `2026-08-04`, the new firmware
string in `SDK / Firmware`, and the new behavior in `Outcome`.

Both rows now live in `protocols.csv`. A reader searching for
`META-QUESTPRO-HAND-OCCLUSION` finds the full historical record:
two observations, two labs, two firmware versions, two outcomes —
and can see exactly how the behavior changed between them.

That is the registry working as intended.

## Corrections

Fixes to rows already published, newest first.

- **2026-10-22**, `META-QUESTPRO-HAND-OCCLUSION`, `Outcome`: was
  "HandTracked drops to 0;". The outcome was cut off; the recordings
  show hand position reading 0 rather than freezing at its last value.
- **2026-10-22**, `META-QUESTPRO-FACE-LOWER-FACE-OCCLUSION`, `Outcome`:
  was "Diverges: upper confidence drops to 0, lower stays high (inverse
  of expected)." An earlier ResXR template build had the upper- and
  lower-face confidence columns swapped; read with the correct labels,
  lower-face confidence drops and upper-face confidence is unchanged,
  as expected.

## Connection to ResXR

[ResXR](https://github.com/ResXR) is an open-source toolkit for
behavioral research with standalone Meta Quest headsets: a Unity
template that records head, hand, eye and face tracking, and an
independent Python pipeline that checks data quality and exports
Motion-BIDS datasets. The seed protocols come from the stress tests
that chose the vendor signals read by the pipeline's hand-tracking-loss
and eye-closure checks. ResXR is described in a bioRxiv preprint,
[doi:10.64898/2026.06.17.732869](https://doi.org/10.64898/2026.06.17.732869).

The registry is intentionally a **standalone community resource**. It
is not part of the ResXR codebase, and contributions are welcomed
regardless of which XR toolchain you use. Pico, Vive, Varjo, and Apple
Vision Pro labs — the seed data are all from the Meta Quest Pro because
that is what we ran, but the schema is vendor-neutral.

## Citing this registry

If you use protocols from this registry in published work, please
cite the registry itself. Tagged releases are archived on Zenodo:
[doi:10.5281/zenodo.23668875](https://doi.org/10.5281/zenodo.23668875)
always points to the newest release, and each release has its own DOI
on that page. GitHub's **Cite this repository** link, in the right sidebar
of the repository page, gives APA and BibTeX entries from
[`CITATION.cff`](CITATION.cff).

When you cite a *specific* protocol from the registry, reference its
`Protocol ID` (e.g. `META-QUESTPRO-HAND-OCCLUSION`) so readers can
locate the exact procedure in `protocols.csv`. Protocol IDs are
stable — the data attached to a procedure may grow as labs add their
own runs, but the ID itself does not change.

## License

CC0 1.0 Universal (public domain dedication); see [LICENSE](LICENSE).
Use the protocols freely; please credit the original contributors when
you cite them.
