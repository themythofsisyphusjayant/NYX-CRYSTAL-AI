# Nyx — Decision Intelligence Simulator

[![Live app](https://img.shields.io/badge/Live%20app-nyx--crystal--ai.lovable.app-brightgreen)](https://nyx-crystal-ai.lovable.app/)
[![Python kernel](https://img.shields.io/badge/Kernel-Python%20%7C%20Pyodide-blue)](public/nyx_kernel.py)
[![Vite + React + TypeScript](https://img.shields.io/badge/Stack-Vite%20React%20TS-61dafb)](package.json)
[![Deterministic](https://img.shields.io/badge/Seed%2042-byte--identical%20runs-orange)](examples/smartphone_ban.py)

A deterministic simulator for hard decisions. Given a question, Nyx builds a
small society of agents that debate, influence each other, and occasionally
break down, then reports a forecast: the outcome, the fragile assumptions
behind it, and what would change it.

**Live app:
[nyx-crystal-ai.lovable.app](https://nyx-crystal-ai.lovable.app/)**
(HTTP 200, checked 21 Nov 2026)

New here? [`docs/EXPLAINED_FOR_BEGINNERS.md`](docs/EXPLAINED_FOR_BEGINNERS.md)
explains the same material in plain language, with the seed-42 worked
example — and labels every claim as verified, reported-but-unverified, or
plan-only.

## Problem

Hard decisions are usually argued from a single viewpoint. Nyx approaches
them as a system: several agents with different psychologies argue the
question, and the dynamics of that argument — convergence, cascades,
recoveries — are the output. The point is not to predict the future. It is
to expose which assumptions the outcome depends on, and which agent or
variable would change it.

## The kernel

All simulation math lives in one file,
[`public/nyx_kernel.py`](public/nyx_kernel.py) (265 lines, 10 functions).
It is deterministic: the same seed produces the same result. Two runs of
the worked example below on 21 Nov 2026 produced byte-identical output.

Each round runs **perception → cognition → intent emission → world
resolution**. Each agent carries ten psychological variables, initialised
in `[0,1]`:

| Variable | Meaning |
|---|---|
| Self-worth | Confidence in their own position |
| Anxiety | Reactivity to conflict and pressure |
| Consistency | Stability of their views |
| Momentum | Drive to push their agenda |
| Reputation | How much others trust them |
| Opportunity access | Ability to influence the group |
| Fragility index | How close they are to an emotional crash ("cascade") |
| Lock-in | Stubbornness or resistance to change |
| Learning rate | Speed of adapting to new arguments |
| Energy | Engagement level |

The variables interact: a drop in self-worth can raise anxiety, lower
consistency, hurt reputation, and pull others into a cascade. The kernel
outputs a full `state_history` (all agents, all rounds) and an
`outcome_vector` (reputation, inequality, trust, centralization).

The kernel runs in the browser via Pyodide (v0.26.4, loaded from jsDelivr;
see `src/hooks/useNyxKernel.ts`), so the same code runs locally with no
server. It also runs standalone.

## Worked example

Question: *"Should we ban smartphones in schools?"* — 6 agents (Maya,
student; Mr. Rao, teacher; Mrs. Khan, parent; Principal; Dev, student;
Dr. Iyer, researcher), seed `42`, 8 rounds. Run it with:

```bash
python examples/smartphone_ban.py
```

Verified by running it on 21 Nov 2026 (and re-run the same day: two runs
byte-identical). Round by round, mean reputation:

| Round | Mean reputation | What happened |
|---|---|---|
| 1 | 0.50 | Initial state |
| 2 | 0.65 | Early convergence |
| 3 | 0.84 | Momentum builds |
| 4 | 0.93 | **Dev cascades** (anxiety-driven crash) |
| 5 | 0.98 | **Dev still cascading** |
| 6 | 0.98 | Dev recovers |
| 7 | 1.00 | Group fully converged |
| 8 | 1.00 | Stable |

Dev, the anxious student (starting anxiety 0.7), cascades in rounds 4–5
and recovers from round 6. Nobody else crashes. Final outcome vector:

| Component | Value |
|---|---|
| reputation_mean | 1.00 |
| inequality | 0.069 |
| trust_proxy | 0.714 |
| centralization | 0.139 |

Or any question, with the kernel directly:

```bash
python public/nyx_kernel.py --seed 42 --scenario "ban smartphones in schools" --agents 12
```

This prints a JSON object with the full state history and outcome vector.

## What a run produces

- Winner and confidence, with a rubric breaking the confidence down into
  feasibility, alignment, risk, and evidence.
- Best-case and worst-case scenarios in plain language.
- Hidden failure points: risks that are easy to miss.
- A timeline of how consensus shifted round by round.
- Agent success chains: who gained momentum and who collapsed.

## Analysis features

- **Counterfactual engine** — "What if anxiety was 20% higher?" Find the
  most influential variables.
- **BlackSwan Assassin** — finds the single most fragile assumption and
  shows how breaking it would cascade.
- **Leverage map** — a graph of who influences whom.
- **Sentiment ridge** — group confidence round by round.
- **Influence flow** — positive and negative influence between agents.
- **Narrative timeline** — key events: cascades, recoveries, BlackSwan
  alerts.
- **Agent drill-down** — all ten variables over time for any agent, with
  mode changes and cascade flags.
- **Variable importance heatmap** — which trait fluctuated most per agent.
- **Game theory** — Nash equilibria and Pareto frontiers.
- **Multi-trial aggregation** — run many simulations, average results,
  classify the pattern (e.g. stable convergence vs polarized stalemate).
  In the app this runs 30 trials over 4 rounds with different seeds,
  clusters outcomes with K-means (K=3), and renders Platt-scaled
  probability estimates (`src/components/MultiTrialAggregation.tsx`).
- **Probabilistic forecasts** — chances of success, backlash, or collapse.
  These are simulation-internal, not real-world validated.
- **Sensitivity and damping analysis** — whether shocks fade or amplify.
- **Validation suite** — same seed, same result; ablation tests for what
  drives outcomes.
- **Dynamical primitives** — narrative diversity, strongest attractor,
  network hubs.
- **Hippocampal episodic replay** — agents remember high-salience past
  events and may replay them when anxious.
- **Regret-driven strategy (CFR)** — agents adjust influence targets based
  on missed opportunities.

## Advanced controls

The app's Setup and Simulation routes expose the kernel's controls
directly, verified in `src/routes/setup.tsx` and
`src/routes/simulation.tsx`:

| Control | Where | What it does |
|---|---|---|
| Seed control | Setup → Simulation | Type a scenario seed; the PRNG seed (`prngSeed`, default 42) drives the whole run and is re-seeded per round in advanced mode |
| Advanced Simulation toggle | Setup | Switches on the advanced engine: per-round PRNG re-seeding, heterogeneous cascade thresholds, and a scale-free influence network built from the seed |
| Initialize from seed | Setup (advanced) | Builds the 10-variable agent state from the scenario seed via the `init_advanced` task |
| Historical Anchor card | Setup (advanced only) | Anchors the scenario to a historical precedent |
| Real-World Context card | Setup (advanced only) | Session-only real-world context for the scenario |
| Agent roster | Agents route | 21 named personas to pick from (`src/lib/nyx-agents.ts`): Harsh (Brutal Critic), Jayant (Strategic Architect), Nova (Trend Forecaster), Atlas (Systems Thinker), Lyra (Ethical Compass), Kai (Devil's Advocate), Mira (Market Analyst), Echo (Cultural Voice), Vox (Public Opinion), Ren (Operator), Sage (Historian), Iris (UX Empath), Orion (Tech Futurist), Juno (Investor Lens), Thane (Legal Risk), Wren (Wildcard), Sol (Optimist), Noor (Pragmatist), Arc (Director), Vera (Report Agent), BlackSwan Assassin (Adversarial Auditor) |
| Multi-trial batch | Simulation route | 30 trials, 4 rounds each, K-means clustering (K=3), Platt-scaled probabilities |
| Simulation length | Simulation route | Default 20 rounds (`DEFAULT_ROUNDS`), configurable per simulation |

The browser kernel path is used only when advanced mode is on and the
Pyodide kernel has loaded; otherwise the app falls back to its TypeScript
engine (`src/lib/nyx-*.ts`).

## Stack and layout

Vite, React, TypeScript. The app is built with Lovable; the simulation
kernel is plain Python and independent of the frontend.

```
public/nyx_kernel.py        Deterministic kernel (265 lines) — the core
examples/smartphone_ban.py  Seed-42 worked example (52 lines)
docs/EXPLAINED_FOR_BEGINNERS.md   Beginner explainer with honesty labels
src/hooks/useNyxKernel.ts   Loads the kernel via Pyodide (v0.26.4)
src/lib/nyx-agents.ts       21 named agent personas
src/lib/nyx-*.ts            TypeScript engine: dynamics, causal, learning,
                            sensitivity, observatory, trajectory, divergence…
src/components/            Analysis panels: MultiTrialAggregation, ValidationSuite,
                            LeverageForceGraph, PolarizationBenchmark, BenchmarkSuite,
                            DominantStrategiesCard, SystemStabilityCard, …
src/routes/                index, setup, agents, simulation, outcomes,
                            report, gallery, login
src/integrations/supabase/  Auth (Supabase) — outside the deterministic core
supabase/functions/nyx-ai/  Server-side AI function
```

| Route | Purpose |
|---|---|
| `/` | Landing |
| `/setup` | Seed → ontology → knowledge graph; Advanced toggle |
| `/agents` | Pick agents from the 21-persona roster |
| `/simulation` | Run rounds, watch cascades, multi-trial batch |
| `/outcomes` | Winner, confidence rubric, scenarios |
| `/report` | Full report view |
| `/gallery` | Saved simulations |
| `/login` | Supabase auth |

## Reproduce it

```bash
git clone https://github.com/themythofsisyphusjayant/NYX-CRYSTAL-AI.git
cd NYX-CRYSTAL-AI
python public/nyx_kernel.py --seed 42 --scenario "ban smartphones in schools" --agents 12
```

This prints a JSON object with the full state history and outcome vector.

In the browser, the kernel loads through Pyodide:

```javascript
const kernelCode = await fetch('public/nyx_kernel.py').then(r => r.text());
await pyodide.runPythonAsync(kernelCode);
const result = pyodide.globals.get('run_simulation')(seed=42, scenario="...");
```

Run the web app locally:

```bash
npm install
npm run dev      # Vite dev server
npm run build    # production build
npm run preview  # serve the production build
npm run lint     # eslint
```

## Limits

- Probabilistic forecasts are simulation-internal. They are not calibrated
  against real-world outcomes.
- Agent variables are initialised in `[0,1]`, but update equations can push
  values outside that range during a run.
- The kernel is deterministic given a seed, but the web app adds its own
  layers (LLM-assisted features, Supabase auth) that are not part of the
  deterministic core.
- The multi-trial batch (30 trials × 4 rounds) is a lighter run than a full
  20-round simulation: it samples outcome patterns, it does not replace a
  full-length run.

## Contributing

Open an issue to discuss major changes first. The kernel
(`public/nyx_kernel.py`) must remain deterministic and pass the validation
suite.

## License

No license file is present in this repository. The previous README stated
MIT, but there is no LICENSE file at the repository root, so that claim has
been removed.
