# mas-llm-robotics

Implementation of the system described in **"Enabling Robotic Cognition: A
Hierarchical Multi-Agentic System for LLM-Driven Autonomous Problem-Solving
in Robotics"** (IEEE Access, manuscript ID **Access-2026-09207**).

This repository contains the source code: the proposed Hierarchical MAS,
the two comparison baselines, the per-agent prompts, the Webots world and
controller, and the TCP/JSON bridge between the LLM-side orchestrator and
the Webots-side robot.

For the evaluation harness, scenarios, results, scoring scripts, grading
rubric, and reproduction guide, see the companion repository:

> **[mas-llm-robotics-eval](https://github.com/kugesan1105/mas-llm-robotics-eval)**

---

## Repository layout

- [`mas/`](mas/) — the proposed system. 17 agents in [`mas/agents/`](mas/agents/), 21-node LangGraph state machine in [`mas/app.py`](mas/app.py), shared topological/RRT tools in [`mas/tools/`](mas/tools/).
- [`baselines/`](baselines/) — Baseline A ([`rule_based.py`](baselines/rule_based.py)) and Baseline B ([`single_llm.py`](baselines/single_llm.py)).
- [`prompts/`](prompts/) — one prompt per LLM-driven agent.
- [`webots/`](webots/) — Webots world ([`webots/worlds/home.wbt`](webots/worlds/home.wbt)) and the Pioneer 3-AT controller.
- [`comm/`](comm/), [`robot_server/`](robot_server/) — TCP/JSON bridge between the LLM-side orchestrator and the Webots-side robot.
- [`requirements.txt`](requirements.txt) — pinned Python dependencies.

## Running

The system runs as a three-process stack: Webots GUI, TCP bridge,
orchestrator. The full launch recipe, the per-scenario door-state table,
known caveats, and troubleshooting are in the companion repository:

- [Reproduction guide](https://github.com/kugesan1105/mas-llm-robotics-eval/blob/master/docs/reproduction.md) (Tier-1 full re-execution and Tier-2 5-second replay).

In short, end-to-end re-execution requires Webots R2023b+, an OpenAI API
key, and both this repository (the code) and the companion repository
(the evaluation harness):

```bash
# Clone both repos side by side
git clone https://github.com/kugesan1105/mas-llm-robotics
git clone https://github.com/kugesan1105/mas-llm-robotics-eval
cd mas-llm-robotics-eval

# Install deps from the code repo
pip install -r ../mas-llm-robotics/requirements.txt

# Make the code repo importable
export PYTHONPATH=$PYTHONPATH:$(pwd)/../mas-llm-robotics

# Then follow the three-terminal recipe in
# https://github.com/kugesan1105/mas-llm-robotics-eval/blob/master/docs/reproduction.md
```

## Architecture overview

The system is organised as a five-tier agent hierarchy with hierarchical
error escalation as the central architectural mechanism. Full code-to-paper
mapping (every node mapped to a source file, the `GraphState` schema, the
escalation chain, the per-agent prompt mapping) is in the companion
repository:

- [Architecture documentation](https://github.com/kugesan1105/mas-llm-robotics-eval/blob/master/docs/architecture.md).

## Citation

Please cite the IEEE Access paper if you use this code. Machine-readable
metadata is in [`CITATION.cff`](CITATION.cff).

## License

MIT — see [`LICENSE`](LICENSE).
