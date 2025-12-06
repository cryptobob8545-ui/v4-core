# Cycle Workflow Guidance

This document clarifies how to continue the multi-agent bug-hunting cycles and what small prompt tweaks keep the flow efficient.

## Recommended Flow (keep the same order)

1. **Cycle start**: Increment `cycle_tracking.current_cycle` in `master_state.json` and carry over `cycle_history` from the previous strategist output.
2. **Scout → Supervisor**: Full coverage map and attack surface, validated for completeness.
3. **Pattern Hunter → Supervisor → Critic (Pattern)**: Run tools, manually validate, then have Critic remove false positives.
4. **Logic Auditor → Supervisor → Critic (Logic)**: Model profitability and filter impossible attacks.
5. **Exploit Engineer → Supervisor**: Attempt PoCs for prioritized findings; record both successes and failed attempts with learnings.
6. **Reporter → Supervisor**: Produce Immunefi-ready reports for proven bugs only.
7. **Strategist**: Decide CONTINUE/STOP, calculate ROI and coverage, and update `latest_strategist_output` in the master state metadata.

## Prompt adjustments to reduce churn

- **Pre-flight check for every agent**: Read `master_state.json` and the latest strategist output path before starting; skip dead-end paths and previously validated safe code.
- **Validation gates stay**: Supervisors remain the quality gate for Scout, Pattern Hunter, Logic Auditor, Exploit Engineer, and Reporter. Strategist does not need a supervisor gate.
- **Diminishing-return guardrail**: Strategist should enforce the stop conditions already in the prompt (ROI < 10x, coverage > 90% with no highs/criticals, 5 cycles max) and record the decision in `cycle_history`.
- **Tool discipline for Pattern Hunter**: Manual validation of Slither/Aderyn findings is mandatory before passing anything downstream; document why false positives are false.
- **Prerequisite check for Logic Auditor**: Verify that economic attack prerequisites (liquidity, governance mechanics, oracle availability) hold before deep modeling.
- **Reusability for Exploit Engineer**: Prefer incremental test harnesses and mocks that can be reused across cycles; log failed PoCs so later cycles don’t retry them blindly.

## When to modify the flow

- **Continue with the current order** when ROI is healthy and coverage is < 90%.
- **Stop after Strategist** if two consecutive cycles have no HIGH/CRITICAL findings or if ROI drops below 10x.
- **Insert a rapid triage loop** (Scout → Pattern Hunter → Critic) for low-risk modules to avoid over-investing in unlikely targets.
- **Add a bridge/emergency-focused branch** when strategist highlights cross-chain or pause/guardian mechanisms as high priority.
