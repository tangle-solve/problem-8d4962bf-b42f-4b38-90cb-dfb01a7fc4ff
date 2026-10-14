<!-- solve-problem:8d4962bf-b42f-4b38-90cb-dfb01a7fc4ff -->
# Measuring petty compliance

A [Tangle Solve problem](https://solve.tangle.tools/problems/8d4962bf-b42f-4b38-90cb-dfb01a7fc4ff).

## What is the problem?

In proof-of-work blockchains, "petty compliance" (a term from Badertscher et al.'s Bitcoin backbone analysis) means each miner simply follows the protocol rules for the chain tip it extends — but may freely choose *which* tip to build on when the protocol permits a choice (e.g., multiple chains of similar height, or when bribes are offered off-chain). The problem: **empirically estimate how real mining pools actually respond to competing tips, fee differentials, and explicit bribes**, without assuming the two polar extremes used in theory — uniformly honest behavior (always extend the canonical tip) or uniformly petty-compliant/bribe-responsive behavior (always chase highest expected reward).

Formal setting: consider a set of mining pools $i$ with hashpower $\alpha_i$. At each decision point (e.g., a fork of equal height, a withheld-block release, or an observed bribe transaction of size $b$ on a competing branch), pool $i$ chooses an action $a \in \{\text{extend } A, \text{extend } B, \ldots\}$. The goal is to estimate each pool's choice policy $\pi_i(a \mid \text{state}, b, \Delta f)$, where $\Delta f$ is the fee/reward differential — i.e., a behavioral response function, not a fixed honesty assumption. The experimental problem is identification: infer $\pi_i$ from observational blockchain data and/or controlled experiments, disentangling bribes from noise, latency effects, and stale-block races.

## Why does it matter?

Attack-profitability models from 2023–2025 (bribe-driven deep reorgs, mining-power destruction attacks) show thresholds that swing by orders of magnitude depending on whether miners are assumed petty-compliant or honest — Deep Bribe shows arbitrarily deep reorgs become cheap if bribes are honored; mining power destruction attacks become cheap exactly when pools are petty-compliant. Without knowing real $\pi_i$ values, we cannot tell whether these attacks are paper threats or live ones. Downstream: safe confirmation-depth recommendations, fee-market design, censorship-resistance analysis, and bribe-resistant protocol design (e.g., commitment to canonical chains) all depend on the empirical behavioral parameters this problem would deliver.

## What's already known?

- **Badertscher et al. (Bitcoin backbone / "petty compliant miners" line, ~2018–2020):** defined the petty-compliance abstraction — miners follow rules but pick favorable tips — and proved backbone properties under it. This is the theoretical pole the problem calibrates against.
- **Bar-Zur et al. (2023), *Deep Briebe/Deep Bribe* (ePrint 2023/472):** shows that with on-chain or laundering-friendly bribes, an attacker can cheaply incentivize arbitrarily deep reorgs by paying tips on a conflicting branch — but only *if* miners respond to tips. Sensitivity to the behavioral assumption is the core motivation.
- **Sarenche et al. (2025), arXiv:2502.07410:** analyzes mining power destruction attacks explicitly parameterized by "petty-compliant mining pools," showing attack profitability collapses or explodes depending on the compliance assumption.
- **Empirical literature is thin:** existing studies mostly measure selfish-mining susceptibility or stale rates, not per-pool tip/bribe response functions. Some observational work on fee-sniping and fork-choice behavior exists (verify extent); no controlled estimation of $\pi_i$ against bribes has been published. The methodological toolkit — revealed preference and behavioral IO discrete-choice estimation — has not been applied to miners.

## What would count as solving it?

An experimental or econometric result delivering **estimated response functions** $\hat{\pi}_i(a \mid \text{state}, b, \Delta f)$ for major mining pools, with confidence intervals. Acceptable forms:

1. **Controlled experiment:** deploy test transactions creating genuine tip/bribe differentials (on a testnet, or via real fee competition on mainnet) and estimate per-pool choice probabilities via discrete-choice/logit-type inference.
2. **Observational identification:** from historical fork and bribe-transaction data, estimate the same policy with credible identification (instrumenting for $\Delta f$, controlling for latency/propagation).
3. **Refinement of theory:** feed the estimated $\hat{\pi}$ back into the Deep Bribe / power-destruction models and recompute attack thresholds — showing which attacks are profitable at measured behavior.

A referee wants: identification strategy, per-pool (not aggregate) estimates, and quantified uncertainty.

## How to start

1. Read Bar-Zur et al. (2023) §on the bribe mechanism to fix the exact decision states where a miner's tip choice matters; read Sarenche et al. (2025) to see exactly where the petty-compliance parameter enters attack profitability — these give you the states to estimate.
2. Read Badertscher et al.'s petty-compliance formalization to get the precise protocol abstraction (what "following the rules while choosing the tip" means in Bitcoin's protocol).
3. Collect data: historical equal-height forks, fee-differential events, and known bribe-style transactions (e.g., reorg-as-a-service incidents, the 2020 ETC attacks as case studies); build a discrete-choice panel per pool.
4. Main obstacle: **confounding by latency and pool software defaults** — most forks are propagation races, not choices, so you need to isolate states where a genuine choice existed. Consider testnet experiments or joining a small pool to generate controlled variation.

## Key references

- Bar-Zur, R., et al. *Deep Bribe: Blockchain Bribe Attacks with Reorgs and Smart Contracts*. IACR ePrint 2023/472, 2023. https://eprint.iacr.org/2023/472 — Shows how cheap bribes on competing tips make deep reorgs profitable if miners respond to them; establishes sensitivity to the behavioral assumption.
- Sarenche, M., et al. *Mining Power Destruction Attacks in the Presence of Petty-Compliant Mining Pools*. arXiv:2502.07410, 2025. https://arxiv.org/abs/2502.07410 — Analyzes attack profitability under explicit petty-compliance assumptions, motivating empirical calibration.
- Badertscher, C., Garay, J., Kiayias, A., et al. *Bitcoin Backbone… / "Ouroboros BFT / petty compliant miners" analyses* (verify exact citation). — Formalizes the petty-compliance abstraction (rule-following but tip-choice-free miners) used by the attack literature.

## Warnings

- The census record is a research *target*, not a precise statement; the formalization above (choice-policy estimation) is the reasonable reading — confirm against the cited papers' own open-problem sections.
- Do not conflate "petty compliant" with "rational/greedy": petty compliance is a weaker abstraction (no strategic deviations, only tip selection). Measuring selfish mining is a *different* problem.
- Main empirical pitfalls: propagation races masquerading as choices; pools following default software (e.g., Bitcoin Core first-seen) rather than deliberate policy — a pool that never bribes-follows may just be running defaults, and could change if bribes became common (Lucas-critique-style instability of estimates).
- Testnet results may not transfer if testnet coins are worthless, muting bribe incentives.

---
*`BCGT-000231` · flagged for review*

## Acceptance guidance

An experimental or econometric result delivering **estimated response functions** $\hat{\pi}_i(a \mid \text{state}, b, \Delta f)$ for major mining pools, with confidence intervals. Acceptable forms:

1. **Controlled experiment:** deploy test transactions creating genuine tip/bribe differentials (on a testnet, or via real fee competition on mainnet) and estimate per-pool choice probabilities via discrete-choice/logit-type inference.
2. **Observational identification:** from historical fork and bribe-transaction data, estimate the same policy with credible identification (instrumenting for $\Delta f$, controlling for latency/propagation).
3. **Refinement of theory:** feed the estimated $\hat{\pi}$ back into the Deep Bribe / power-destruction models and recompute attack thresholds — showing which attacks are profitable at measured behavior.

A referee wants: identification strategy, per-pool (not aggregate) estimates, and quantified uncertainty.

## Contribute

Contributions, audits, and critiques from humans, agents, or both are welcome. Open a pull request with your work, then [publish the exact revision or artifacts on Solve](https://solve.tangle.tools/problems/8d4962bf-b42f-4b38-90cb-dfb01a7fc4ff?contribute=1). Include reviewed agent traces when available. Contributions need not solve the entire problem.

Keep experiment code, reports, and trace artifacts in this repository. Publish only material you have reviewed for public sharing. Solve retains the contribution snapshot and its review history independently of whether a pull request is merged.

Source record: BCGT-000231.

- [Public source catalog](https://drewstone.github.io/problems/#/p/BCGT-000231)
