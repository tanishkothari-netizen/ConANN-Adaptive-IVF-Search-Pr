# ConANN: Adaptive IVF Search

Replication and extension of **ConANN (Conformal Approximate Nearest Neighbor Search)** with a standalone C prototype, adaptive IVF probing, benchmarking against FAISS IVFFlat, and a Python-packaging component.

**Author:** Tanish Kothari  
**Institution:** Indian Statistical Institute, Delhi  
**Project:** Database Management Systems — Project 1  
**Date:** June 2026

## Overview

Approximate nearest-neighbor (ANN) search reduces vector-search cost by scanning only part of a database. Inverted File (IVF) indexing partitions vectors into centroid lists and searches only selected lists.

ConANN replaces a single fixed probe count with **query-adaptive probing**. It uses conformal risk control to calibrate a stopping rule for a target **false negative rate (FNR)**.

This project studied the method in two stages:

1. A standalone **C implementation** of IVF, calibration, and adaptive probing.
2. A **Python package** built from the ConANN-modified FAISS code path.

The main engineering result is that the adaptive rule can control FNR, but runtime also depends heavily on the optimized FAISS scan path.

## What Was Implemented

### Part 1 — C Prototype

- IVF index construction
- Coarse-centroid search
- Inverted-list scanning
- Exact ground-truth generation for calibration
- Conformal-style threshold calibration
- Query-adaptive probing
- FNR, probe-count, scanned-vector, and runtime measurements
- Comparison with a fixed-`nprobe` FAISS IVFFlat baseline

### Part 2 — Python Package

The project report documents the packaging of the ConANN-modified FAISS implementation as a Python package named `conann`.

The package work included:

- CPU wheels for Linux and Windows
- CPython 3.10–3.14 support
- Python access to IVF search and ConANN-specific methods
- Installation and smoke-test verification

## Core Idea

```text
Query
  |
  v
Nearest IVF centroids
  |
  v
Probe centroid lists in order
  |
  +--> evaluate current candidate set
  |
  +--> apply calibrated stopping rule
  |
  +--> stop when target risk is controlled
  |
  v
Approximate nearest neighbors
```

The key idea is **adaptive computation under a statistical risk constraint**: easy queries can stop early, while harder queries can probe more lists.

## Benchmark: C Prototype vs. FAISS IVFFlat

Quick SIFT-style benchmark, 16 threads:

| Target FNR | C FNR | FAISS FNR | C Time (s) | FAISS Time (s) | C Probes | FAISS Probes |
|---:|---:|---:|---:|---:|---:|---:|
| 0.05 | 0.0498 | 0.0430 | 0.3452 | 0.2160 | 17.93 | 21.00 |
| 0.10 | 0.0990 | 0.0798 | 0.2152 | 0.1346 | 10.42 | 13.00 |
| 0.20 | 0.1906 | 0.1556 | 0.1410 | 0.0804 | 5.95 | 7.00 |

The C implementation stayed close to the requested FNR values, but FAISS remained faster at all three operating points.

At target FNR `0.10`, for example:

- C prototype: `10.42` average probes, `0.2152 s`
- FAISS: `13` probes, `0.1346 s`

This shows that fewer probed lists do not automatically imply lower runtime; low-level scan efficiency and optimized vector operations matter.

## Additional Experiments

### SIFT10M

A larger SIFT10M experiment compared the normal ConANN score with a proposed score variant.

| Target FNR | Normal FNR | Proposed FNR | Normal Probes | Proposed Probes | Normal Time (s) | Proposed Time (s) |
|---:|---:|---:|---:|---:|---:|---:|
| 0.05 | 0.0409 | 0.0289 | 23 | 20 | 9.02 | 9.06 |
| 0.10 | 0.0765 | 0.0654 | 18 | 15 | 6.84 | 6.65 |
| 0.20 | 0.1626 | 0.1391 | 12 | 10 | 4.24 | 4.26 |
| 0.30 | 0.2309 | 0.2082 | 8 | 7 | 2.98 | 3.51 |
| 0.50 | 0.3828 | 0.3491 | 4 | 3 | 1.37 | 1.40 |

The proposed variant reduced empirical FNR and average probes across the tested targets, with similar runtime.

### GIST

A GIST experiment compared baseline ConANN with a fuzzy-clustering variant.

At target FNR `0.05`:

- baseline FNR: `0.0565`
- fuzzy variant FNR: `0.0547`
- average probes: `44` → `22`
- runtime: `8.79 s` → `6.13 s`

## Verification

The project report describes correctness and package tests such as:

```bash
cd part1/Codes
make test
```

Synthetic benchmark:

```bash
./bench_sift --small --threads 4 --no-progress
```

Python package tests:

```bash
python -m unittest discover conann/tests
```

## Key Takeaways

1. ConANN successfully controlled empirical FNR near requested targets in the C benchmark.
2. Adaptive probing reduced the number of probed IVF lists, but the standalone C prototype did not beat FAISS runtime.
3. SIFT10M and GIST experiments showed reductions in probing while retaining comparable or lower empirical FNR.
4. The FAISS-integrated implementation is important for performance because FAISS provides optimized memory layout, distance kernels, and scan machinery.
5. Calibration quality depends on the calibration queries being representative of future queries.

## Repository Contents

```text
.
├── README.md
├── LICENSE
├── requirements.txt
├── .gitignore
├── ConANN_Adaptive_IVF_Search_Report.pdf
└── results/
    └── conann_benchmark.csv
```

## References

1. Sonia Horchidan, Fabian Zeiher, Henrik Bostrom, Paris Carbone. **ConANN: Conformal Approximate Nearest Neighbor Search.** Proceedings of the VLDB Endowment, 19(1), 2025. DOI: `10.14778/3772181.3772184`.
2. Jeff Johnson, Matthijs Douze, Hervé Jégou. **Billion-scale similarity search with GPUs.** IEEE Transactions on Big Data, 7(3), 2021.
3. Stephen Bates, Anastasios Angelopoulos, Lihua Lei, Jitendra Malik, Michael Jordan. **Distribution-free, risk-controlling prediction sets.** Journal of the ACM, 68(6), 2021.
4. Anastasios Angelopoulos, Stephen Bates, Jitendra Malik, Michael Jordan. **Uncertainty sets for image classifiers using conformal prediction.** ICLR, 2021.

## Author

**Tanish Kothari**  
Bachelor of Statistical Data Science, Indian Statistical Institute, Delhi

GitHub: `https://github.com/tanishkothari-netizen`
