# Three-qubit finite-sampling simulations

[results.csv](results.csv) contains the 300 reconstructions used in Table 1 and Figure 1. The gate labels are `rx` for $R_x(\pi/2)^{\otimes 3}$ and `cz` for $\mathrm{CZ}_{q_0,q_1}\otimes I_{q_2}$. Each method used the same 50 independently sampled data sets for each gate. PPRGD and SLSQP also shared the initial Kraus matrices for each paired run.

For **Table 1**, group the rows by `gate` and `method` and calculate the arithmetic mean and sample standard deviation (divisor 49) of each of the four metrics below. Use all 50 rows in each group. For **Figure 1**, each row gives one point with `runtime_s` on the horizontal axis and `relative_choi_error` on the vertical axis. The left panel uses `rx` and the right panel uses `cz`.

| Column in results.csv | Definition |
| --- | --- |
| `gate`, `root_seed`, `method` | The gate, simulated data set and estimator identify a row. The root seed also appears in the input filename. |
| `relative_choi_error` | Frobenius distance between the fitted and ideal Choi matrices divided by the Frobenius norm of the ideal matrix, 8. Expressed as a fraction. |
| `average_gate_fidelity_pct` | Average gate fidelity to the ideal unitary, in percent. For the unnormalized Choi matrices this equals $100[\mathrm{tr}(\widehat J J_\star)+8]/72$. |
| `fit_rmse` | Root mean square of the differences between predicted probabilities and the 13,824 observed outcome frequencies, in probability units. |
| `runtime_s` | Recorded wall-clock duration of the original solver run in seconds. |
| `iterations`, `stop_reason` | Terminal update count and status from the solver. Counts refer to PPRGD matrix updates, SLSQP major iterations or PLS projection iterations, respectively. |
| `input_sha256` | SHA-256 digest of the complete per-seed input NPZ file. The three methods share this digest for each gate and seed. |

## Inputs and fitted channels

The [rx](rx/) and [cz](cz/) directories have the same layout. The design and per-seed input files are byte-for-byte copies of the original inputs. For each gate, seeds 9141501–9141506 come from the initial 4,096-shot batch and seeds 9141507–9141550 from the extension to 50 data sets. The recorded runtimes for each paired PPRGD and SLSQP reconstruction use the first run (repetition 0).

| File in each gate directory | Contents |
| --- | --- |
| `design.npz` | `states` holds 64 input kets as rows. `measurement_vectors` contains 216 measurement kets, one row per basis and outcome. `input_labels`, `basis_labels` and `outcome_labels` give their three-qubit labels. `U`, `true_K`, `true_J` and `true_probabilities` give the ideal unitary, its stacked Kraus representation, its Choi matrix and the sampling probabilities. `Rx` or `CZ` gives the elementary gate. |
| `inputs/seed_<root_seed>.npz` | The integer `counts` array has axes (input, basis, outcome) and shape (64, 27, 8). Each setting sums to 4,096. Divide by 4,096 for observed frequencies. `K_initial` is the shared 16 × 8 feasible start. Scalar arrays give the root, data, initialization and perturbation seeds. |
| `estimates.npz` | The leading axis follows the stored `root_seed` array, in ascending order. `pprgd_K` and `slsqp_K` have shape (50, 16, 8) and contain the final stacked Kraus matrices. `pls_J` has shape (50, 64, 64) and contains the final projected Choi matrices. |

Qubit labels are ordered $(q_0,q_1,q_2)$, with $q_2$ varying fastest. The single-qubit inputs are $0,1,+,+i$ and the measurement axes are $X,Y,Z$. Outcome bits 0 and 1 denote the positive and negative eigenstates of the selected axis. Row $8b+o$ of `measurement_vectors` gives outcome $o$ in basis $b$, using zero-based indices. The Choi convention places the input tensor factor first and vectorizes each Kraus matrix by columns.

For example, this reads the first rotation-gate reconstruction from the `simulation` directory and forms its Choi matrix.

```python
import numpy as np

with np.load("rx/estimates.npz", allow_pickle=False) as fits:
    root_seed = int(fits["root_seed"][0])
    kraus = fits["pprgd_K"][0].reshape(2, 8, 8)
vectors = kraus.transpose(0, 2, 1).reshape(2, 64)
choi = vectors.T @ vectors.conj()
with np.load(f"rx/inputs/seed_{root_seed}.npz", allow_pickle=False) as data:
    frequencies = data["counts"] / 4096
```

## Reconstruction settings

[run_parameters.json](run_parameters.json) gives the settings used for the reported runs, including tolerances, update budgets and software versions. PPRGD and SLSQP fit a rank-at-most-two model with equal weight on all 13,824 squared frequency residuals. PPRGD used step size 40. PLS used linear inversion followed by Frobenius projection onto the full CPTP set. Its runtime includes construction of the design inverses, linear inversion and projection. For PPRGD and SLSQP the timer covers the solver through its actual endpoint, including the in-solver stopping checks. File I/O and post-run diagnostics are outside the reported runtimes.
