# ConANN: Adaptive IVF Search

A research project on **ConANN (Conformal Approximate Nearest Neighbor Search)**, studying query-adaptive IVF search under a controlled false-negative-rate (FNR) target.

**Co-authors:** Raj Lohar, Bhagavath Chukka, Tanish Kothari, Madhumita Das, Suman Polley  
**Institution:** Indian Statistical Institute  
**Project:** Database Management Systems — Project 1  
**Date:** June 2026

---

## Overview

Approximate nearest-neighbor (ANN) search reduces vector-search cost by scanning only a subset of a vector database.

This project studies **Inverted File (IVF)** search, where vectors are partitioned into centroid lists. The usual trade-off is controlled by `nprobe`:

- lower `nprobe` → faster search, potentially lower recall;
- higher `nprobe` → better recall, higher computation.

**ConANN** replaces a fixed probe count with a query-adaptive stopping rule. The method uses **conformal risk control** to calibrate search so that the empirical false negative rate can be controlled around a requested target.

---

## What We Did

The project had two connected components.

### 1. Standalone C Prototype

We implemented and evaluated:

- IVF indexing;
- centroid assignment;
- inverted-list search;
- exact nearest-neighbor ground truth;
- calibration for target FNR;
- adaptive probing;
- FNR measurement;
- average probe-count measurement;
- runtime benchmarking;
- comparison against FAISS IVFFlat.

### 2. Python Package

We also worked with the **ConANN-modified FAISS implementation** and packaged the implementation for Python use.

The resulting Python module is available here:

**[ConANN Python Package](https://github.com/smoothy100x/conann)**

The package provides ConANN-enabled IVF functionality through Python and was built with CPU support for Linux and Windows.

---

## Core Idea

```text
                 Query
                   |
                   v
          Find nearest IVF centroids
                   |
                   v
       Probe centroid lists sequentially
                   |
                   v
          Evaluate current candidates
                   |
                   v
        Apply calibrated stopping rule
              /                      STOP           CONTINUE
            |                |
            v                v
       Return ANN       Probe more lists
```

The important idea is **adaptive computation under a statistical risk constraint**: different queries can require different amounts of search.

---

## Benchmark: C Prototype vs FAISS IVFFlat

The quick SIFT-style benchmark compared the adaptive C implementation with a fixed-`nprobe` FAISS IVFFlat baseline.

| Target FNR | C FNR | FAISS FNR | C Time (s) | FAISS Time (s) | C Probes | FAISS Probes |
|---:|---:|---:|---:|---:|---:|---:|
| 0.05 | 0.0498 | 0.0430 | 0.3452 | 0.2160 | 17.93 | 21.00 |
| 0.10 | 0.0990 | 0.0798 | 0.2152 | 0.1346 | 10.42 | 13.00 |
| 0.20 | 0.1906 | 0.1556 | 0.1410 | 0.0804 | 5.95 | 7.00 |

The C prototype stayed close to the requested FNR targets:

- target `0.05` → measured `0.0498`;
- target `0.10` → measured `0.0990`;
- target `0.20` → measured `0.1906`.

However, the standalone C implementation did **not** beat FAISS in runtime.

This is an important systems-level observation: probing fewer IVF lists does not necessarily mean lower runtime. FAISS benefits from optimized memory layout, distance computation, and list-scanning infrastructure.

---

## SIFT10M Experiment

A larger SIFT10M experiment compared the standard ConANN score with a proposed score variant.

| Target FNR | Normal FNR | Proposed FNR | Normal Probes | Proposed Probes | Normal Time (s) | Proposed Time (s) |
|---:|---:|---:|---:|---:|---:|---:|
| 0.05 | 0.0409 | 0.0289 | 23 | 20 | 9.02 | 9.06 |
| 0.10 | 0.0765 | 0.0654 | 18 | 15 | 6.84 | 6.65 |
| 0.20 | 0.1626 | 0.1391 | 12 | 10 | 4.24 | 4.26 |
| 0.30 | 0.2309 | 0.2082 | 8 | 7 | 2.98 | 3.51 |
| 0.50 | 0.3828 | 0.3491 | 4 | 3 | 1.37 | 1.40 |

Across the tested targets, the proposed variant reduced empirical FNR and the average number of probed clusters while maintaining similar runtime.

---

## GIST Experiment

We also evaluated a fuzzy-clustering variant on GIST.

At target FNR `0.05`:

| Metric | Baseline | Fuzzy Variant |
|---|---:|---:|
| FNR | 0.0565 | 0.0547 |
| Average probes | 44 | 22 |
| Runtime (s) | 8.79 | 6.13 |

The experiment showed a substantial reduction in the number of probed clusters while maintaining a comparable empirical FNR.

---

## Key Takeaways

1. **Adaptive probing can control empirical FNR near a requested target.**
2. **Query-adaptive search can reduce the number of IVF lists probed.**
3. **A standalone implementation is not automatically faster than FAISS**, even when it performs fewer probes.
4. **Integration with optimized FAISS internals is important for practical performance.**
5. The quality of the calibration depends on the calibration queries being representative of future queries.

---

## Python Implementation

The Python implementation developed as part of the project is available here:

### [ConANN — Python Package](https://github.com/smoothy100x/conann)

The project report describes Python APIs including:

```python
import conann

index.calibrate_conann(...)
index.search_conann(...)
index.evaluate_conann(...)
index.conann_time_report(...)
```

The package provides a Python interface to the ConANN-enabled IVF implementation.

---

## References

1. Sonia Horchidan, Fabian Zeiher, Henrik Bostrom, Paris Carbone. **ConANN: Conformal Approximate Nearest Neighbor Search.** Proceedings of the VLDB Endowment, 19(1), 2025. DOI: `10.14778/3772181.3772184`.
2. Jeff Johnson, Matthijs Douze, Hervé Jégou. **Billion-scale similarity search with GPUs.** IEEE Transactions on Big Data, 7(3), 2021.
3. Stephen Bates, Anastasios Angelopoulos, Lihua Lei, Jitendra Malik, Michael Jordan. **Distribution-free, risk-controlling prediction sets.** Journal of the ACM, 68(6), 2021.
4. Anastasios Angelopoulos, Stephen Bates, Jitendra Malik, Michael Jordan. **Uncertainty sets for image classifiers using conformal prediction.** ICLR, 2021.

---

## Authors

**Raj Lohar**  
**Bhagavath Chukka**  
**Tanish Kothari**  
**Madhumita Das**  
**Suman Polley**

Indian Statistical Institute

