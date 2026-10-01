# TKET Agent Skill

An [Agent Skill](https://github.com/K-Dense-AI/scientific-agent-skills) for **TKET / pytket**,
Quantinuum's open-source quantum compiler toolkit. It follows the K-Dense-AI
`scientific-agent-skills` conventions and the open [Agent Skills specification](https://agentskills.io),
so it loads as a drop-in agent skill in any skills-compatible host.

The skill teaches an agent to build, optimise, and execute quantum circuits with pytket: the
half-turn angle convention, compilation passes and predicates, architecture-aware placement and
routing, rebasing to a hardware gateset, local Aer simulation, shot sampling and expectation values,
Pauli measurement reduction, noise tailoring, Quantinuum H2 execution via `qnexus`, OpenQASM 2
import/export, and conversions to Qiskit, Braket, IQM, and cuTensorNet.

## Contents

```
skills/tket/
  SKILL.md                     entry point
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
```

`SKILL.md` is what an agent reads first. The `references/` files are consulted on demand — read only
the one that matches the current task.

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

Each script answers `--help` even before pytket is installed:

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

The suite skips cleanly when pytket or `pytket-qiskit[aer]` is not installed.

## Verification

Verified on **2026-09-30 / 2026-10-01** against `pytket 2.18.4`, `pytket-qiskit 0.78.0`,
`qiskit-aer 0.17.2`, and `pytket-quantinuum 0.59.3`, cross-checked against the `Quantinuum/tket`
sources.

## License

Apache-2.0. See [LICENSE](LICENSE).

Cite TKET as: S. Sivarajah, R. Duncan, and A. Kissinger, *TKET*, Quantum Science and Technology
**6**(1) 014003 (2020), [arXiv:2003.10611](https://arxiv.org/abs/2003.10611). Companion references
for routing, phase gadgets, and UCC are listed in `skills/tket/references/sources.md`.