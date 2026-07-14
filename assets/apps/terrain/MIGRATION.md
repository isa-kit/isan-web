# terrain (Models) — schema migrations

## 2026-07-12 r3 — per-model diagram framing + actions on the fill body (additive)

seedVersion `2026-07-12.3-diagram-frames`. Click-through polish of r1, found by
the standing real-build verification. (1) POSITIONS REBASED: `model_steps.pos`
was laid out for the retired whole-fleet Pipeline canvas, so each model's
scoped Diagram opened mostly off-viewport (negative x). Every model's outer
chain is rebased to start near the origin (left-to-right from ~60,80), and
each parent step's child cluster is rebased the same way for the step_detail
sub-diagram. Pure data heal — ids/links/labels untouched. (2) ACTIONS MOVED
INSIDE THE PANE: a screen `header` never renders above a fill body, so a
first pass put Edit model / Scenarios & runs / View output on map in a
button row above the fill segments switcher — that render fixed the drop,
but real-web-runtime click-through testing found the row never received
pointer events (hover/press painted, onTap never fired; confirmed via both
trusted coordinate clicks and semantics DOM clicks, three independent
passes), while content INSIDE the segments panes (bubl bubbles, segment
tabs, card/record row taps) reliably did. Rather than ship dead buttons,
the actions moved one level deeper: each screen's first pane now opens
with the button row above its content — model's Diagram pane is
`column[ row(Edit model, Scenarios & runs, View output on map), bubl ]`,
step_detail's Inside pane is `column[ row(Edit step), text, bubl, text ]`.
The chrome fill-column shape is unchanged (`column[ segments(fill) ]`, no
header); only where the buttons sit relative to the tab bar changed.
Existing installs: LWW seed heal updates un-edited `pos` fields on the
bump; no schema change.

## 2026-07-12 r2 — HuggingFace models as selectable methods (additive)

seedVersion `2026-07-12.2-hf-options` (renumbered from the handoff spec's
suggested `2026-07-12.1-hf-options` — that slot was taken by the r1
model-diagram round landing first; bumped to `.2` here). Two nullable columns
on `methods`: `intent` (enum translate|embed|classify|stt|chat|none, default
none — `none` = pure math, every existing row unchanged) and `hf_repo` (text
provenance repo id, never an endpoint/key). SEED: two new `methods` rows
(`mt_hf_opusmt` opus-mt translation, `mt_hf_embed` sentence-transformers
embeddings — `approach` intentionally empty, they are models not formulas)
and two new `model_layers` rows attaching them to `m_4step`
(`ml_4s_hf_trans` status `selectable`, `ml_4s_hf_embed` status `planned`,
both `stage:pre`) — reusing the exact calibration-solver pattern
(`ml_4s_cal_*`) already shipped. `intent`/`hf_repo` also added to the
Registries → Methods cards' `expandFields` so the repo id is visible; no
other UI change, no new tables, no engine change. HF options are
documentation-only in this round — inert until Phase 2-forge (server-side
inference) lands; see `_claude/projects/model-subcomponents-hf-selectable-spec.md`.
Existing installs: additive LWW seed heal only; no destructive step.

## 2026-07-12 r1 — model view = the diagram; drill; Pipeline retired (additive)

seedVersion `2026-07-12.1-model-diagram` (merges two sibling rounds: this and
the unshipped `2026-07-10.3-model-flow` branch — one combined landing).
(1) TAP-TO-OPEN: the Models registry cards set `expandable:false`, so a tap
fires the existing selectRow→goto chain directly (long-press keeps
Open/Edit/Delete). (2) MODEL VIEW RESHAPED: the model screen body is a
fill `segments` switcher — Diagram · Summary · Components · Layers · Data.
Diagram is the model's own flow (bubl scoped via the NEW engine `filter` prop
`{column: model_id, value: $selected:models}`; `showNested:false` tucks
sub-steps behind a +N badge on their parent). (3) FRACTAL DRILL: tapping a
process in the diagram selectRow→goto's the NEW off-nav `step_detail` screen —
"Inside" is a second bubl scoped `{column: parent, value:
$selected:model_steps}` showing that step's internal input → processing →
output chain; "This step" is the step's full record. SEED: `tt_net` (m_ttmap)
and `cn_graph` (m_centrality) gain 4 child steps + 3 internal links each
(model_steps 42→50, step_links 36→42) so the Korea models drill for real,
matching the existing `p_interm` pattern. (4) OUTPUT ACCESS: the model header
gains "View output on map" → Outputs. (5) PIPELINE TAB RETIRED: the global
Pipeline nav tab + screen are removed — the unfiltered whole-fleet bubl was
the bug the scoped Diagram replaces; its genuinely-global "Inputs" and
"Stacks" cards move into Registries as two new segments. nav is now
overview · runs · outputs · catalog · about. Existing installs: additive
LWW seed heal only (new rows seed in; saved rows win); no destructive step.

## 2026-07-10 r2 — flow navigation + pre/post stages + provider seam (additive)

seedVersion `2026-07-10.2-flow-nav`. Three additive moves. (1) STAGE: NEW
`model_layers.stage` enum pre|core|post (default core) — attached functions
declare whether they run before the chain, inside it, or after; the model
view's layers block now groups pre → core → post (status stays as the colored
tag + teal selectable accent); all 16 seeded layers stamped. (2) NAVIGABLE
FLOW: the Pipeline bubl gains `onLinkTap: editRow` (NEW engine prop,
kEngineBuild bubl-link-tap.1 — tapping a drawn arrow opens that data-flow
row), `pinchZoom`, `holdMenu`, `zoomable` — the diagram is now the navigation
surface (tap a process = its full row incl. math; tap an arrow = what flows;
pinch/hold-ring to move around). Catalog gains an "Algorithm layers" segment
with an "Attach a function to a model" addRow button (the tray: pick model,
registry method, stage). (3) PROVIDER SEAM (llmlayer-0710 coordination): NEW
`models.provider` text ("Runs on — server / provider", shown on the model
record) seeded on all 9 models — LABELS ONLY; endpoints/keys stay out of
applet JSON (data-not-code), so LLM/server-backed models later ride the same
record shape without a migration. Existing installs: new column/fields arrive
on the bump (LWW seeds fields that have never been edited).

## 2026-07-09 r2 — algorithm layers: methods become composable per model (additive)

seedVersion `2026-07-09.2-model-layers`. NEW join table `model_layers` (one row
per algorithm working inside a model: model_id × method_id refs + layer /
algorithm display text + role + status active·selectable·planned) — entering a
model now lists its algorithms as layer rows you can add/swap/remove; the
calibration solver trio from `.1-calib-select` appears as the selectable case
(teal accent). Methods registry += the six core algorithms (`mt_crosswalk`,
`mt_crossclass`, `mt_gravity`, `mt_logit`, `mt_assign`, `mt_bca`), each with
purpose/when-use/approach/cons/must-document + package. NEW `models.uses_model`
column ("Model used (builds on)" — curated text, seeded on all 7 models); the
model drill-in record gains uses_model + data_in + data_out. Wellbeing's six
planned method-layers are seeded honestly as `planned` (the model itself is
planned). Existing installs: new table/column/rows arrive on the bump; no
reseeded fields on existing rows (nothing to heal — uses_model is a NEW field,
which LWW seeds cleanly onto existing model rows).

## 2026-07-09 — calibration solver selectable per run (additive)

seedVersion `2026-07-09.1-calib-select`. The calibration ALGORITHM becomes an
interchangeable, selectable module (user direction): NEW `run_config.calib_method`
enum (bisection · IPF · Bayesian optimization, default bisection) + NEW
`runs.calib_method` (`first:run_config.calib_method` — the selection freezes onto
every run snapshot like β/tier/zone do). NEW methods rows `mt_ipf`
(IPF/Furness — table balancing to margins) + `mt_bayesopt` (multi-parameter
search) join `mt_bisect`, each with its must-document clause + a new additive
`methods.package` column naming where it runs (Julia server packages vs the
~30-line device function). NEW `model_steps.s_calib` swappable component +
`step_links.l_calsel` calibration arrow surface the solver on the m_4step
module list and pipeline. Seeded runs stamp `calib_method` ONLY where the
cited source documents the calibration (beta present) — 5 rows stay honestly
empty. Existing installs: new rows/columns arrive on the bump; m_4step's
updated `calibration`/`cal_method` prose heals via `migrateTerrainCockpitSeeds`
(heal-table entries added). Compute stays server-side: selecting IPF/Bayes-opt
documents intent today; the live solver swap goes real when /model unlocks
(same honest-dormant contract as ▶ Run scenario).

## 2026-07-06 — model cockpit: runs return as the COCKPIT LEDGER (additive)

seedVersion `2026-07-06.1-cockpit`. Terrain regains a lightweight run surface —
three NEW tables `run_config` (single setup row `cfg`), `runs` (append-only
ledger whose `first:run_config.*` column defaults freeze the model-attribute +
scenario-config snapshot onto every recorded launch), `endpoint_info` (health
probe target) — plus five additive `models` columns (runnable, primary_method,
cal_method, calibration, sources).

**Suite roles (declared canonicity, no takeback of the 2026-07-04 split):**
terrain's `runs` is the *cockpit ledger* — launches recorded in-app plus 13
curated snapshots of real transit4 run reports (each row's `status` cites its
source file). The **scenarios** applet remains the *deep-provenance archive*
(run_inputs / run_checks joins, model_history); Atlas maps outputs; Model
reports charts them. The two run tables live in different stores and never
sync to each other; the cockpit's "Scenarios" button is the bridge.

**Upgrade path (existing installs, dev-channel only):** new tables and new
columns arrive on seedVersion bump; RESEEDED FIELDS ON EXISTING ROWS DO NOT
(content merge is LWW with saved rows winning ties — protects user edits by
design). Concretely, an install that already has terrain keeps m_4step's old
`model_version` "7fa3c41" summary/coverage prose and the `ci_coverage` cities
row until the user opts into applet menu → "Reset from file" (destructive) —
fresh installs seed the corrected values (m_4step `fee6a0b`, folded coverage,
no ci_coverage row). Nothing breaks either way; the stale prose is cosmetic
and documented here rather than silently overwritten.

## 2026-07-04 — suite split: terrain → Models; spine + outputs moved out

The urban-models suite split into four distinct applets (user direction).
Tables REMOVED from terrain's definition and re-homed:

- → **apps/scenarios** (new applet, store `isan_scenarios`): runs, run_config,
  run_inputs, run_checks, soundness_checks, confidence_tiers, model_history,
  datasets, standards — the run + proof spine stays joined in ONE store.
- → **apps/atlas**: results, zone_stats, slices, shapes (+ towns.geojson) —
  the map dashboard's outputs and painted layers.
- RETIRED (no new home): layers (terrain's copy; atlas keeps its own), panels,
  layer_stats — display registries superseded by the split.

Terrain keeps the logic/registry tables only: models, cities, model_steps,
step_links, zone_systems, modes, metric_families, wellbeing_factors, methods.

Data impact: dev-channel only (terrain never shipped to prod). Old rows for
removed tables remain untouched in existing stores (definitions dropped, data
not deleted); the new applets seed their own copies (seedVersions
2026-07-04.10-split / .10-dashboard / scenarios .1-init). No convert-on-read
needed — the seeds ARE the canonical fixture data for these applets today.

## 2026-07-09.2-model-inputs (t4inputs-0709)
Additive seed tables only — no heal needed (LWW keeps user rows; these tables
are new): `data_inputs` (geofenced input registry: source · where-it-applies ·
granularity · vintage · status · script · stitch, per model) and `model_stacks`
(named compositions with documented seams). `models` gains `data_status`
(inputs-satisfied line) + two new rows (m_forecast — back-cast skill on the
card; m_router — exact routing, honesty correction stated). `model_steps` gains
per-model lanes (forecast/router/wellbeing/translator/land-use) + `stack`-kind
links; pipeline screen = header model picker (setFilter) + Flow | Inputs |
Stacks segments (fill). Drill-in gains a per-model inputs block.

## 2026-07-10.1-model-view (t4mview-0710)
LIST -> MODEL VIEW restructure (engine select-row.1 required). Models tab is
now a card LIST; tapping chains the new `selectRow` + `goto` to the dedicated
`model` screen, where record/components/layers/inputs all bind via the
`$selected:models` token. Additive schema: `model_steps` gains `role`
(source/preprocess/engine/math/subprocess/postprocess/output) + `math`
(the mathematical logic, real formulas); `data_inputs` gains `shape` /
`extent` / `regions` (REAL previews computed from the server data);
+`s_engine` row (the Julia four-step engine component). No heal needed:
new columns + new row only; existing row values untouched.
