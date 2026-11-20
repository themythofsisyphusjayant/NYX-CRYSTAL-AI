# Nyx — Decision Intelligence Simulator

A deterministic simulator for hard decisions. Given a question, Nyx builds a
small society of agents that debate, influence each other, and occasionally
break down, then reports a forecast: the outcome, the fragile assumptions
behind it, and what would change it.

Live app: [nyx-crystal-ai.lovable.app](https://nyx-crystal-ai.lovable.app/)

New here? `docs/EXPLAINED_FOR_BEGINNERS.md` explains the same material in
plain language, with the seed-42 worked example.

## Problem

Hard decisions are usually argued from a single viewpoint. Nyx approaches
them as a system: several agents with different psychologies argue the
question, and the dynamics of that argument — convergence, cascades,
recoveries — are the output. The point is not to predict the future. It is
to expose which assumptions the outcome depends on, and which agent or
variable would change it.

## The kernel

All simulation math lives in one file, [`public/nyx_kernel.py`](public/nyx_kernel.py).
It is deterministic: the same seed produces the same result. Two runs of the
worked example below on 21 Nov 2026 produced byte-identical output.

Each round runs perception → cognition → intent emission → world resolution.
Each agent carries ten psychological variables, initialised in `[0,1]`:

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

The kernel runs in the browser via Pyodide in the web app, so the same code
runs locally with no server. It also runs standalone.

## Worked example

Question: *"Should we ban smartphones in schools?"* — 6 agents (Maya,
student; Mr. Rao, teacher; Mrs. Khan, parent; Principal; Dev, student;
Dr. Iyer, researcher), seed `42`, 8 rounds. Run it with:

```bash
python examples/smartphone_ban.py
```

Verified by running it on 21 Nov 2026:

- Mean reputation climbs 0.50 (initial) → 0.65 → 0.84 → 0.93 → 0.98 →
  0.98 → 1.00 → 1.00. The group converges fast.
- Dev, the anxious student (starting anxiety 0.7), cascades in rounds 4–5
  and recovers from round 6. Nobody else crashes.
- Final outcome vector: reputation_mean 1.00, inequality 0.069, trust_proxy
  0.714, centralization 0.139.

Or any question, with the kernel directly:

```bash
python public/nyx_kernel.py --seed 42 --scenario "ban smartphones in schools" --agents 12
```

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

The web app exposes seed control, custom agent profiles, batch experiments,
and kernel inspection through its advanced controls.

## Stack

Vite, React, TypeScript. The app is built with Lovable; the simulation kernel
is plain Python and independent of the frontend.

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

## Limits

- Probabilistic forecasts are simulation-internal. They are not calibrated
  against real-world outcomes.
- Agent variables are initialised in `[0,1]`, but update equations can push
  values outside that range during a run.
- The kernel is deterministic given a seed, but the web app adds its own
  layers (LLM-assisted features, Supabase auth) that are not part of the
  deterministic core.

## Contributing

Open an issue to discuss major changes first. The kernel
(`public/nyx_kernel.py`) must remain deterministic and pass the validation
suite.

## License

No license file is present in this repository. The previous README stated
MIT, but there is no LICENSE file at the repository root, so that claim has
been removed.
