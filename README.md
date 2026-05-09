# mas-llm-robotics

Implementation of the proposed Hierarchical Multi-Agentic System (MAS)
for LLM-driven autonomous problem-solving in robotics.

This repository contains the source code: the proposed Hierarchical MAS,
the two comparison baselines, the per-agent prompts, the Webots world
and Pioneer 3-AT controller, and the TCP/JSON bridge between the LLM-side
orchestrator and the Webots-side robot.

For the evaluation harness, scenarios, results, grading rubric, and the
documentation that explains each evaluation, see the companion repository:

> **[mas-llm-robotics-eval](https://github.com/kugesan1105/mas-llm-robotics-eval)**

## Repository layout

- [`mas/`](mas/) — the proposed system. 17 agents in [`mas/agents/`](mas/agents/), 21-node LangGraph state machine in [`mas/app.py`](mas/app.py), shared topological/RRT tools in [`mas/tools/`](mas/tools/).
- [`baselines/`](baselines/) — Baseline A ([`rule_based.py`](baselines/rule_based.py)) and Baseline B ([`single_llm.py`](baselines/single_llm.py)).
- [`prompts/`](prompts/) — one prompt per LLM-driven agent.
- [`webots/`](webots/) — Webots world ([`webots/worlds/home.wbt`](webots/worlds/home.wbt)) and the Pioneer 3-AT controller.
- [`comm/`](comm/), [`robot_server/`](robot_server/) — TCP/JSON bridge between the LLM-side orchestrator and the Webots-side robot.
- [`requirements.txt`](requirements.txt) — pinned Python dependencies.

## Architecture overview

The system is organised as a hierarchical Multi-Agentic System with three
logical layers: an interpretation layer that parses the user command into
a strategic plan; a supervisory layer that decomposes the plan into
sub-tasks and handles errors escalated from below; and an
execution-and-perception layer that interfaces with the robot and reports
perception events back upward. A failure observed at the execution layer
(e.g., an unexpectedly closed door) is propagated upward together with
the full mission context, allowing the supervisory layer to re-invoke the
strategic planner under the updated environmental constraints rather than
abort.

## Running end-to-end in Webots

End-to-end re-execution requires both repositories (this one for the
system code, the companion repo for the eval harness), Webots R2023b+,
an OpenAI API key, and a Python env with the pinned dependencies.
Estimated cost: ~3 hours of Webots wall-clock plus ~$5–10 in GPT-4o API.

### Setup

```bash
# Clone both repos side by side
git clone https://github.com/kugesan1105/mas-llm-robotics
git clone https://github.com/kugesan1105/mas-llm-robotics-eval
cd mas-llm-robotics-eval

# Conda env (or any equivalent venv)
conda create -n agent python=3.11
conda activate agent
pip install -r ../mas-llm-robotics/requirements.txt

# Make the code repo importable
export PYTHONPATH=$PYTHONPATH:$(pwd)/../mas-llm-robotics

# OpenAI API key
export OPENAI_API_KEY=sk-...
```

### Three-terminal launch

| # | Terminal | Command | What it does |
|---|----------|---------|--------------|
| 1 | Webots GUI | open `../mas-llm-robotics/webots/worlds/home.wbt` from inside Webots | Starts the simulator, the Pioneer 3-AT robot, and the controller in `webots/controllers/project_12_pioneercontroller_1/`. The controller connects to the TCP bridge in step 2. |
| 2 | Linux terminal | `python -m robot_server.mainserver` | Starts the bridge server on port 5000. Both the Webots controller and the orchestrator connect here. |
| 3 | Linux terminal | `python -m eval.run_experiment --systems rule_based single_llm mas --trials 1 --scenarios-file scenarios/scenarios.json --results-dir results/` | Runs each scenario on each of the three systems and writes per-scenario JSON logs. |

Verify connectivity before starting scenarios: terminal 2 should print
*"Server listening on 0.0.0.0:5000"* followed by *"Client 'webots'
connected from ..."* once the Webots controller has come up.

### Manual door step (before each scenario)

Each scenario in
[scenarios/scenarios.json](https://github.com/kugesan1105/mas-llm-robotics-eval/blob/master/scenarios/scenarios.json)
declares a `door_states` field. Doors not listed are open by default.
Before running a scenario, manually set the door positions in the Webots
GUI to match its `door_states` and place the robot at the scenario's
`initial_position`.

| Scenario | initial_position | door_states |
|---|---|---|
| s01, s02, s05, s06, s07, s08, s09, s10, s11, s12, s13, s15, s18, s19, s20 | R3 | (all open) |
| s03 | R3 | D3 closed |
| s04 | R3 | D3 closed, D4 closed |
| s14 | R2 | D2 closed |
| s16 | R1 | D2 closed |
| s17 | R2 | (all open) |

This manual step is the main reproduction friction. It can be automated
via the Webots Supervisor API but is left manual in this release to keep
the orchestrator independent of any Webots-version-specific binding.

## References

Please cite the corresponding IEEE Access paper if you use this code.
Machine-readable metadata is in [`CITATION.cff`](CITATION.cff).

## License

MIT — see [`LICENSE`](LICENSE).
