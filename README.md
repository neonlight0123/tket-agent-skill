# TKET Agent Skill

An [Agent Skill](https://github.com/K-Dense-AI/scientific-agent-skills) for **TKET / pytket**,
Quantinuum's open-source quantum compiler toolkit. It is written to the K-Dense-AI
`scientific-agent-skills` conventions.

The skill teaches an agent to build, optimise, and execute quantum circuits with pytket: the
half-turn angle convention, compilation passes and predicates, architecture-aware placement and
routing, rebasing to a hardware gateset, local Aer simulation, shot sampling and expectation values,
Pauli measurement reduction, noise tailoring, Quantinuum H2 execution via `qnexus`, OpenQASM 2
import/export, and conversions to Qiskit, Braket, IQM, and cuTensorNet.

## Contents

```
skills/tket/
  SKILL.md                     entry point (277 lines; the spec cap is 500)
  references/
    setup.md                   install, versions, credentials, troubleshooting
    circuits.md                construction, gates, classical logic, boxes, inspection
    compilation.md             passes, predicates, placement, routing, rebases, combinators
    backends.md                Backend abstraction, simulators, results, Quantinuum hardware
    algorithms.md              Pauli algebra, expectation values, ZX, tableau, QFT/QPE
    noise.md                   noise models, tailoring, SPAM correction, measurement reduction
    interop.md                 QASM 2, Quipper, extension converter matrix, cross-SDK gotchas
    sources.md                 verification snapshot, canonical URLs, how to refresh
  scripts/
    check_environment.py       report installed vs verified versions; smoke-test the core API
    run_local_simulation.py    sample, get statevectors, compute expectation values locally
    compile_and_report.py      compile against a topology and report the gate/depth/CX delta
tests/tket/
  conftest.py                  loads the upstream structural contract when available
  test_scripts.py              53 tests + 32 subtests
```

`SKILL.md` is what an agent reads first. The `references/` files are consulted on demand — read only
the one that matches the current task. `skills/tket/references/sources.md` records exactly what this
skill was verified against and how to refresh it.

## Using the skill

Create a virtual environment and install pytket with the Aer simulator extra:

```bash
python -m venv .venv
# Windows: .venv\Scripts\activate   |   macOS/Linux: source .venv/bin/activate
python -m pip install pytket "pytket-qiskit[aer]"
```

> There is no `pytket-aer` distribution. The Aer backends ship inside `pytket-qiskit` behind the
> `[aer]` extra, so the install line above is the correct one.

Then run the bundled scripts:

```bash
python skills/tket/scripts/check_environment.py
python skills/tket/scripts/run_local_simulation.py --shots 1000
python skills/tket/scripts/compile_and_report.py --demo adder-ish --qubits 5 --architecture linear:6
```

Each script imports only the standard library at module level and defers every `pytket` import into
its functions, so `--help` works even before pytket is installed:

```bash
python skills/tket/scripts/check_environment.py --help
```

`check_environment.py --require-simulator` additionally smoke-tests the Aer backend; `--strict`
turns a version drift from a warning into an error. `compile_and_report.py --json` and
`run_local_simulation.py --json` emit machine-readable reports.

## Running the tests

```bash
python -m pip install pytest
pytest tests/tket -q
```

The suite is skipped cleanly when pytket or `pytket-qiskit[aer]` is not installed. `tests/tket/`
exposes the upstream `tests/_contract` package as `skill_contract` when it is present (after grafting
into `scientific-agent-skills`, or if that repo is cloned alongside this one) and skips the shared
CLI-help contract test when it is not; everything else runs either way.

## Grafting into `scientific-agent-skills`

To add this skill to a clone of [K-Dense-AI/scientific-agent-skills](https://github.com/K-Dense-AI/scientific-agent-skills):

1. Copy `skills/tket/` into the repo's `skills/` directory and `tests/tket/` into its `tests/`
   directory. Only `SKILL.md`, `references/`, `scripts/`, and `assets/` may live under a skill
   directory — the upstream contract rejects tests and bytecode there.
2. Add a `[skills.tket]` entry to `tests/skill-requirements.toml`. A minimal set that resolves
   together in one clean environment is `pytket[zx]`, `pytket-qiskit[aer]`, and `numpy`; add
   `pytket-quantinuum[pecos]` and `qnexus` only if the hardware workflow is exercised. Do **not**
   add `qermit` — see the note below.
3. Bump the plugin version. Upstream's `tests/_meta/test_repo_contract.py` requires `plugin.json`
   `version` to equal the `pyproject.toml` `[project].version`, and adding a skill is a minor bump.
4. Validate:

   ```bash
   uv run skills-ref validate skills/tket
   pytest tests/_meta -q
   pytest tests/tket -q
   ```

### The `qermit` dependency conflict

`qermit` (ZNE, CDR, and probabilistic error cancellation) **cannot be installed beside the current
extension pins.** Every released `qermit` requires `pytket-qiskit<0.78` and
`pytket-quantinuum[pecos]<0.59`. Resolving them together does not fail — pip *silently backtracks* to
`qermit 0.7.1`, a release that predates the API the current documentation describes. Give ZNE/CDR/PEC
work its own environment, and always print the resolved version before trusting any documented API:

```bash
python -c "import importlib.metadata as m; print(m.version('qermit'))"
```

`skills/tket/scripts/check_environment.py` detects this conflict at runtime and warns about it.
Install guidance is in `skills/tket/references/setup.md`; the mitigation workflow is in
`skills/tket/references/noise.md`.

## Verification

Verified on **2026-09-30 / 2026-10-01** against `pytket 2.18.4`, `pytket-qiskit 0.78.0`,
`qiskit-aer 0.17.2`, and `pytket-quantinuum 0.59.3`, cross-checked against the `Quantinuum/tket`
sources.

Every snippet in `SKILL.md` was executed against a real pytket install, every Python block in the
references was parsed, and every API name the skill imports was resolved against the installed
package. Where the official documentation and a `.pyi` type stub disagreed, **the stub won** —
`skills/tket/references/sources.md` lists the known documentation defects.

## License

Apache-2.0, matching pytket itself. See [LICENSE](LICENSE).

Cite TKET as: S. Sivarajah, R. Duncan, and A. Kissinger, *TKET*, Quantum Science and Technology
**6**(1) 014003 (2020), [arXiv:2003.10611](https://arxiv.org/abs/2003.10611). Companion references
for routing, phase gadgets, and UCC are listed in `skills/tket/references/sources.md`.
