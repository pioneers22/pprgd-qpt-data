# QZ01 hardware data

These files support the hardware results in Table 2 and the Pauli transfer matrices in Figure 2. For each operation, PPRGD fitted one effective three-qubit channel to a complete set of measurement counts from the WuYue QZ01 superconducting processor. The fitted channels are compared with their ideal gate operations. These comparisons include the effects of state preparation, measurement-basis rotations and readout.

| Directory | Ideal operation | Physical qubits for logical $(q_0,q_1,q_2)$ | Figure 2 |
| --- | --- | --- | --- |
| [rx/](rx/) | $R_x(+\pi/2)$ on each qubit, with $R_x(\theta)=\exp(-i\theta X/2)$ | Q7, Q1, Q15 | Left column |
| [cz/](cz/) | Controlled Z on $q_0,q_1$, with $q_2$ idle | Q11, Q12, Q7 | Right column |

## Measured counts

Each gate directory contains the original exported `counts.csv`. Each row gives one preparation and measurement setting. The 64 product inputs and 27 product bases give 1,728 distinct settings per gate. Each setting has 4,096 valid joint readouts, for a total of 7,077,888 per gate.

| CSV fields | Meaning |
| --- | --- |
| `dataset_id` | Gate-specific acquisition label |
| `input_id`, `input_q0`, `input_q1`, `input_q2` | Zero-based input index and the three single-qubit preparations. The order is `0`, `1`, `+`, `+i` on each qubit, with $q_2$ varying fastest. |
| `basis_id`, `basis_q0`, `basis_q1`, `basis_q2` | Zero-based basis index and the three measured Pauli operators. The order is X, Y, Z on each qubit, with $q_2$ varying fastest. |
| `shots` | Number of valid joint readouts for this row |
| `count_000` through `count_111` | Eight integer counts in logical $q_0,q_1,q_2$ order, with $q_0$ on the left. In each measured basis, bits 0 and 1 denote the $+1$ and $-1$ eigenvalues, respectively. |

The preparation `+i` is $(|0\rangle+i|1\rangle)/\sqrt{2}$, the $+1$ eigenstate of Y. The count columns are already in logical order. The exporter used the platform counts without readout correction. Platform-side readout correction was unreported. Divide a count by the row's `shots` to obtain its observed frequency.

## Reconstruction and figure data

[settings.json](settings.json) contains the experimental design, physical qubit mappings and settings for the two PPRGD runs. Both reconstructions used candidate Kraus capacity 2, step size 40, gradient triggering threshold $10^{-7}$ and curvature tolerance $10^{-3}$. The loss gives each of the 13,824 measured frequencies equal weight.

Each gate directory contains three NumPy archives readable with `np.load(path, allow_pickle=False)`.

| File | Arrays |
| --- | --- |
| `initial.npz` | The saved complex initial matrix `K_initial` (16 × 8) and its `initial_seed` and `perturbation_seed`. |
| `estimate.npz` | The fitted matrix `K` (16 × 8, two vertically stacked Kraus operators), estimated and ideal Choi matrices `J_est` and `J_ref` (64 × 64), and ideal unitary `U_ref` (8 × 8). `J` is a second copy of `J_est` in the original archive. `fitted_probabilities` and `observed_frequencies` have axes (input, basis, outcome) and shape 64 × 27 × 8. |
| `pauli_transfer.npz` | Numerical data for Figure 2, derived from the saved `K` and `U_ref`. `ideal`, `reconstructed` and `difference` are the top, middle and bottom panels. The 64 strings in `pauli_labels` index both axes. |

The Choi matrices use the unnormalized convention, with the input tensor factor first, so their trace is 8. For the Pauli transfer matrices, row $a$ labels the output Pauli string and column $b$ the input string. Each string uses the qubit order $q_0,q_1,q_2$. The strings run from III to ZZZ, using I, X, Y, Z on each qubit. The entries are $R_{ab}=\operatorname{Tr}[P_a\,\mathcal{E}(P_b)]/8$. For the fitted channel, $\mathcal{E}(P_b)=\sum_j K_j P_b K_j^\dagger$. The difference array is the reconstructed matrix minus the ideal matrix.

[results.csv](results.csv) gives the two rows before manuscript rounding. The first three metrics form Table 2. The fourth is the largest absolute entry of the corresponding Figure 2 difference matrix.

| Metric | Calculation |
| --- | --- |
| `average_gate_fidelity_percent` | $100(\operatorname{Tr}[J_\mathrm{est}J_\mathrm{ref}]+8)/72$ |
| `relative_choi_distance` | $\lVert J_\mathrm{est}-J_\mathrm{ref}\rVert_F/\lVert J_\mathrm{ref}\rVert_F$ |
| `fitting_rmse` | Root mean square of fitted probability minus observed frequency across all 13,824 entries, in probability units |
| `max_abs_ptm_difference` | Maximum absolute value over the 64 × 64 Pauli transfer matrix difference |
