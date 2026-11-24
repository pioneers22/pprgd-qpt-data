# Three-qubit quantum process tomography data

This repository contains the data behind the tables and figures in *Optimization Guarantees for Perturbed Preconditioned Riemannian Gradient Descent in Quantum Process Tomography*. The experiments cover two gates, $R_x(\pi/2)^{\otimes 3}$ and a controlled-$Z$ on $q_0,q_1$ with $q_2$ idle.

| Manuscript item | Files and documentation | Quantities reported |
| --- | --- | --- |
| Table 1 | [Simulations](simulation/README.md) | Reconstruction error, average gate fidelity, fitting RMSE and runtime for PPRGD, SLSQP and PLS over 50 finite-sampling data sets per gate |
| Figure 1 | [Simulations](simulation/README.md) | Relative Choi error (fraction, vertical axis) versus runtime (seconds, horizontal axis) for all three methods. Panel (a) shows the rotation and panel (b) shows CZ. |
| Table 2 | [QZ01 hardware](hardware/README.md) | Average gate fidelity, relative Choi distance and fitting RMSE for the channel reconstructed from each gate's measurement counts |
| Figure 2 | [QZ01 hardware](hardware/README.md) | Ideal, reconstructed and reconstructed-minus-ideal Pauli transfer matrices (top to bottom). The rotation is on the left and CZ is on the right. |

Each data set uses 64 product input states and 27 product Pauli measurement bases. Each of the 1,728 input–basis combinations has 4,096 shots and eight possible three-bit outcomes.

In the simulations, the ideal gate generates the counts, so the relative Choi distance measures reconstruction error. On QZ01, each gate has one measured data set. The hardware distance compares its reconstructed effective channel with the corresponding ideal gate.

## License

The data and documentation in this repository are licensed under Creative Commons Attribution 4.0 International (CC BY 4.0). [Full license text](LICENSE)
