# Groth16 in Python

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/DeveloperMarwan/RareSkills_ZK/blob/main/Groth16_Final.ipynb)

A Python implementation of the Groth16 zk-SNARK proving system over the BN128 (alt_bn128) curve. I wrote this notebook from scratch as the capstone of the RareSkills Zero Knowledge Bootcamp (August to September 2023 cohort), adding a new section each week of the six-week program. It follows the RareSkills article [Groth16](https://rareskills.io/post/groth16). Curve and field arithmetic come from `py_ecc` and `galois`; the Groth16 logic itself is my own code.

The notebook runs the whole pipeline end to end on a small example circuit, and the final pairing check confirms that the proof verifies.

## What it implements

1. **R1CS.** The circuit is expressed as three matrices (L, R, O) over the BN128 scalar field. The notebook asserts that the witness satisfies `(L·w) ∘ (R·w) = O·w`.
2. **R1CS to QAP.** Each matrix column is interpolated into a polynomial with Lagrange interpolation. The notebook computes the target polynomial `t(x)` and the quotient `h(x) = (U(x)·V(x) − W(x)) / t(x)`, and asserts that the division has no remainder.
3. **Trusted setup.** The notebook builds the structured reference string as elliptic curve points: powers of tau in G1 and G2, the encrypted `u`, `v`, and `w` polynomial evaluations, the `h(τ)·t(τ)/δ` terms, and the alpha, beta, gamma, and delta elements. Public and private terms of the C polynomial are divided by gamma and delta respectively.
4. **Proof generation.** The proof consists of three elements, A (G1), B (G2), and C (G1). The prover uses the blinding factors `r` and `s` and computes B in both G1 and G2, as the C term requires.
5. **Verification.** The verifier checks the pairing equation `e(A, B) = e(α, β) · e(public input, γ) · e(C, δ)`. It is evaluated as a product of pairings followed by the final exponentiation, which must equal 1 in FQ12.

## The example circuit

The prover demonstrates knowledge of private inputs `x` and `y` for a public output `out`, where:

```
out = 5x³ + x² + 13y² − 10y − 4x²y²
```

The circuit has 4 constraints and 7 witness variables: `[1, out, x, y, v1, v2, v3]`, where `v1 = x²`, `v2 = 5x³`, and `v3 = y²`. The public inputs are `1` and `out`. The private inputs are `x`, `y`, and the intermediate values. The example uses `x = 29` and `y = 11`.

## Running it

**In the browser:** click the Colab badge above and choose *Runtime → Run all*.

**Locally:**

```bash
pip install py_ecc==6.0.0 galois==0.3.6 jupyter
jupyter notebook Groth16_Final.ipynb
```

The notebook was run with Python 3.10 and `numpy` below 1.26 (a `galois` requirement).

**Expected result:** the cell prints the pairing values, and the last line shows `final_result` equal to `(1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0)`, which is the multiplicative identity in FQ12. The final `assert` passes without error.

## Libraries

- [`py_ecc`](https://github.com/ethereum/py_ecc) for BN128 curve arithmetic and pairings
- [`galois`](https://github.com/mhostetter/galois) for finite field and polynomial arithmetic
- `numpy` for the constraint matrices

## Limitations

This is an educational implementation, not a production prover.

- The trusted setup values (alpha, beta, tau, delta, gamma) are hard-coded, so the "toxic waste" is known. A real deployment would generate them in a multi-party ceremony.
- The blinding factors `r` and `s` are fixed values, not random.
- The circuit and its matrices are written by hand. There is no circuit compiler.
- There is no serialization of the proving or verification keys.
- The notebook demonstrates the success case only. It does not include a test showing that a tampered witness or proof fails verification.

## Reference

- RareSkills, [Groth16](https://rareskills.io/post/groth16)
