# Spatiotemporal Resource Allocation in Photonic Architectures

This repository studies temporal multiplexing and shared-buffer constraints in probabilistic photonic resource preparation. It combines analytical photon-availability models with seeded Monte Carlo simulation to examine the effects of optical delays, switching transmission, selection policies, and synchronization windows on accepted output rates. The final section introduces an event-driven scheduler in which multiple photon rails compete for finite optical storage.

The models operate at the level of **photon-availability events**. An accepted output is not a computed graph-state fidelity or a fault-tolerant logical operation, and the parameters are not calibrated to a particular optical device.

## Repository

```text
.
├── README.md
└── Spatiotemporal_Photonic_Architecture.ipynb
```

The single notebook contains the baseline model (Parts 1–8) followed by the finite-buffer extension (Part 9). All required Python definitions appear in the notebook; no separate source modules or input files are needed.

| Part | Analysis |
| --- | --- |
| 1–3 | Emission time bins, a sequential-emitter reference, optical storage and switch losses |
| 4 | Exact acceptance for latest-available-photon selection |
| 5 | Optional balanced beam-splitter example in Perceval |
| 6 | Monte Carlo estimation, uncertainty, and source-attempt accounting |
| 7–8 | Rate–resource Pareto optimization, hardware sensitivity, and a reproducible parameter sweep |
| 9 | First-arrival retention with a finite shared buffer, blocked arrivals, and a matched-policy comparison |

## Independent-rail multiplexing

For $k$ rails, $m$ emission bins, and $f$ required heralded operations, the baseline defines the initially known photon-availability probability as

$$
p_0=\eta_{\mathrm{source}}\eta_{\mathrm{transmission}}\eta_{\mathrm{detector}}.
$$

The model retains the **latest available photon** on each rail. A photon selected with storage age $a$ survives the delay with probability $\eta_d^a$. For a switch tree of depth $d_s(m)=\lceil\log_2m\rceil$, the accepted single-rail probability is

$$
q_m=p_0\eta_s^{d_s(m)}\sum_{a=0}^{m-1}\big[(1-p_0)\eta_d\big]^a.
$$

With independent rails and fusion-herald flags,

$$
P_{\mathrm{out}}=q_m^k h^f,\qquad
R_{\mathrm{out}}=\frac{P_{\mathrm{out}}}{mT},
$$

where $T$ is the emission-bin period. Windows do not overlap, and the rate does not include pipelining, routing delays, or feed-forward latency.

The factor $\eta_{\mathrm{detector}}$ inside $p_0$ assumes detection or heralding informs *photon availability before selection*. For an output-only destructive detector, detection efficiency belongs after selection instead; the present parameterization should not be read as a general model for unheralded quantum-dot emission.

The latest-arrival policy **can be causal**: an online controller may replace an older retained candidate with a fresher one as photons arrive. The baseline presumes dedicated per-rail storage and replacement without extra physical penalties; it does not explicitly simulate that optical controller.

### Throughput versus resource provisioning

The notebook records two distinct source-attempt metrics:

- **Provisioned attempts per window:** $C_{\mathrm{window}}=km$, a measure of emission opportunities allocated to each attempted output window.
- **Attempts per accepted output:** $C_{\mathrm{output}}=km/P_{\mathrm{out}}$, an efficiency statistic that includes discarded windows.

At fixed $k$ and $T$ these satisfy

$$
C_{\mathrm{output}}R_{\mathrm{out}}=\frac{k}{T}.
$$

Consequently, maximizing rate and minimizing $C_{\mathrm{output}}$ are exactly the same optimization problem. The **Pareto frontier uses $C_{\mathrm{window}}$ versus $R_{\mathrm{out}}$** instead, producing a genuine provisioning–throughput trade-off. For the provided baseline parameters and $m=1,\ldots,16$, the Pareto-efficient windows are $m=1,2,3$:

| Bins $m$ | Attempts per window $km$ | Accepted rate ($\mathrm{s}^{-1}$) |
| ---: | ---: | ---: |
| 1 | 3 | $2.3611\times10^6$ |
| 2 | 6 | $4.0271\times10^6$ |
| 3 | 9 | $4.2435\times10^6$ |

The highest accepted rate occurs at $m=3$. Neither attempt metric is equivalent to the number of fabricated emitters, device price, or energy consumption.

## Finite shared-buffer extension

The independent-rail model assumes that selected photons can always be retained. The extension instead gives $k$ rails access to a buffer with $c$ shared slots, processes emission bins sequentially, and records arrivals blocked by full storage.

Under the extension's **first-arrival, no-replacement, hold-until-deadline** policy, the first photon detected as available on a rail occupies one slot until the output deadline. A candidate that cannot be stored is rejected. Photon storage survival, single-stage switch survival, and one fusion-herald outcome are sampled independently.

When $c\geq k$, capacity cannot bind and each rail is independent. The corresponding earliest-arrival probability is

$$
r_m=\eta_{\mathrm{sw}}\sum_{t=0}^{m-1}(1-p)^t p\eta_d^{m-1-t},\qquad
P_{\mathrm{online}}=r_m^k h.
$$

This formula is **not** the same as the latest-arrival formula: the original model uses replace-on-arrival with a switch-tree depth, whereas the extension keeps the first photon and applies a single switch factor. Both policies are causal. The notebook also provides a comparison with matched availability, delay, switching, and fusion assumptions to isolate the selection-policy effect.

Under the stated rule, every selected photon—including one arriving in the final bin—occupies a slot. Thus $c<k$ implies zero accepted windows, while $c\geq k$ behaves as unlimited storage. This is a model-specific constraint: systems that allow final-bin optical bypass, release/retry, or pipelining may behave differently. The paired extension plot reports **both accepted output rate and blocked candidate arrivals per window**, allowing sub-capacity designs to be distinguished even when their accepted rates are zero.

## Numerical checks and results

The notebook contains deterministic limiting-case assertions, independent-seed Monte Carlo convergence estimates, and exact-versus-sampled probability comparisons. It also checks that capacities greater than the rail count do not change results under common random numbers.

For the illustrative three-rail, eight-bin extension ($p=0.42$, $\eta_d=0.97$, $\eta_{\mathrm{sw}}=0.98$, $h=0.50$), the unlimited-buffer earliest-arrival calculation gives $P_{\mathrm{online}}\approx0.269314$. A 150,000-trial run gives approximately $0.268073$ (analytical standard error $\approx0.001145$). With the same input availability, delay, idealized switching, and fusion assumptions, the matched-policy comparison produces:

| Selection policy | Accepted probability |
| --- | ---: |
| Latest available photon | 0.429476 |
| First available photon | 0.286141 |

These probabilities describe alternative idealized policies and **do not measure an experimental hardware advantage**.

Executing the notebook writes seven PNG figures to `figures/` (`timing_bins.png`, `delay_loss.png`, `multiplex_tradeoff.png`, `monte_carlo_convergence.png`, `pareto_frontier.png`, `hardware_sensitivity.png`, and `finite_buffer_scheduling.png`) and a numerical sweep to `results/window_sweep.csv`. The repository contains the source notebook and README only; generated plots are also embedded in notebook outputs.

## Running the notebook

Python 3.10 or later is recommended. From the repository directory:

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install numpy matplotlib jupyter
jupyter notebook Spatiotemporal_Photonic_Architecture.ipynb
```

Select the environment's Python kernel and choose **Restart & Run All**. On Windows, activate the environment with `.venv\Scripts\activate`.

The optional two-mode beam-splitter example requires Quandela Perceval:

```bash
python -m pip install perceval-quandela
```

The numerical resource models do not require Perceval. The optional example is skipped cleanly when that package is unavailable; its physical-circuit execution was not verified in the environment used for this repository's numerical checks.

## Model limitations

The calculations are event-level, phenomenological models, not optical fusion-circuit or quantum-dot device simulations. Their assumptions include independent Bernoulli events, fixed nonoverlapping windows, idealized knowledge of photon availability, prescribed storage and switch losses, and an externally specified fusion-herald probability. The extension omits spatial routing, buffer write/read disturbance, active release, and feed-forward latency. Neither model includes quantum-state propagation, spectral indistinguishability, measured emitter parameters, graph-state fidelity, or logical error correction.

Agreement between exact probability expressions and Monte Carlo estimates checks **implementation consistency under these assumptions**; it is not experimental validation.

## References

- S. Bartolucci *et al.*, “Fusion-based quantum computation,” *Nature Communications* **14**, 912 (2023). https://doi.org/10.1038/s41467-023-36493-1
- M. Chan *et al.*, “Tailoring Fusion-Based Photonic Quantum Computing Schemes to Quantum Emitters,” *PRX Quantum* **6**, 020304 (2025). https://doi.org/10.1103/PRXQuantum.6.020304
- Quandela, [Perceval circuit tutorial](https://perceval.quandela.net/docs/v1.3/notebooks/Circuit_Tutorial.html).
