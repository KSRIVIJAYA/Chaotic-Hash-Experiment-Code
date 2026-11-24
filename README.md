# Chaotic Hash Integrity Experiments

Reproducible Python experiments for the synthetic dataset **SMI-80-2026** and the manuscript:

*A Dynamic Chaotic Hash-Based Integrity Verification Mechanism for Secure Cloud Multimedia Data.*

This repository implements the manuscript's stated Algorithm 1 and generates new, independently reproducible measurements. It does **not** reconstruct the manuscript's unavailable historical 80-file pilot measurements.

## 1. Repository layout

```text
chaotic-hash-experiments/
├── src/
│   ├── chaotic_hash.py
│   └── run_experiments.py
├── tests/
│   └── test_algorithm.py
├── .gitignore
├── CITATION.cff
├── LICENSE
├── requirements.txt
└── README.md
```

Keep the dataset archive in a separate GitHub release or external storage if repository size limits are a concern. Do not commit private keys or sensitive research data.

## 2. Requirements

- Python 3.10 or newer
- The core implementation uses only the Python standard library.
- Optional plotting: `matplotlib` (not required to calculate CSV results).

```bash
python --version
```

## 3. Prepare the dataset

Download and extract `SMI-80-2026_synthetic_multimedia_dataset.zip`. The extracted folder must contain `dataset_manifest.csv`, `tamper_manifest.csv`, `originals/`, and `tampered/`.

Example:

```text
project/
├── chaotic-hash-experiments/
└── SMI-80-2026/
    ├── dataset_manifest.csv
    ├── tamper_manifest.csv
    ├── originals/
    └── tampered/
```

## 4. Run a quick test first

From the repository root:

```bash
python src/run_experiments.py --dataset ../SMI-80-2026 --output results_quick --max-files 4 --max-bytes 524288
```

This validates the manifest and runs the diagnostic experiments plus the selected small dataset files.

## 5. Run the complete experiment

```bash
python src/run_experiments.py --dataset ../SMI-80-2026 --output results_full --repeats 5
```

The full dataset timing includes 80 files up to 60 MiB. The proposed chaotic transform uses a sequential logistic-map iteration for each byte and is therefore computationally expensive for large files. Run the quick test first, then consider using `--max-bytes` to stage the experiment:

```bash
python src/run_experiments.py --dataset ../SMI-80-2026 --output results_upto_10MiB --max-bytes 10485760 --repeats 5
```

For a faster baseline-only dataset timing run, use:

```bash
python src/run_experiments.py --dataset ../SMI-80-2026 --output results_baselines --skip-chaotic-timing
```

Note: `--skip-chaotic-timing` skips proposed-method dataset timing but the script still computes proposed digests for tamper tests. The six-vector diagnostic timing section is skipped only with `--skip-diagnostics`.

## 6. Algorithm and experimental metrics

The proposed digest follows Algorithm 1 in the manuscript:

1. Require a 32-byte secret key and compute SHA-256(K).
2. Derive `x0` and `mu` from the first and second 8-byte key-hash segments.
3. Generate the logistic sequence in binary64 arithmetic and discard 256 iterations.
4. Derive one byte `q_i = floor(256*x_i) mod 256` per input byte.
5. Derive `Kd = SHA256(K || Q || Enc(L))`.
6. Transform each input byte using `c_i = m_i XOR q_i XOR Kd[i mod 32]`.
7. Return `SHA256(C || Kd || Enc(L))`, with `Enc(L)` as unsigned 8-byte big-endian.

The experiment runner calculates:

- **Manifest verification:** exact file length and SHA-256 check against the supplied manifest.
- **Processing latency and throughput:** proposed composition, SHA-256, SHA3-256 and HMAC-SHA-256 on matched deterministic vectors and selected dataset files.
- **Avalanche effect:** 256 deterministic 2,048-byte inputs, each paired with a one-bit input change; digest Hamming distance is reported as a percentage.
- **Key sensitivity:** 256 one-bit key variants, each compared with the baseline digest for a fixed 2,048-byte message.
- **Digest-byte entropy:** the raw observation CSVs allow pooled digest-byte entropy to be calculated. Do not calculate 8-bit entropy from a single 32-byte digest and interpret it as an 8-bit sample.
- **Empirical duplicate digest check:** records any duplicate digest within the selected dataset for each method. No finite sample proves collision resistance.
- **Controlled tamper detection:** checks the 12 supplied one-bit-altered copies against original digests.

## 7. Output files

The `results/` directory contains CSV/JSON files, including:

- `manifest_validation.csv`
- `timing_observations.csv`, `timing_summary.csv`
- `dataset_timing_observations.csv`, `dataset_timing_summary.csv`
- `avalanche_observations.csv`, `avalanche_summary.csv`
- `key_sensitivity_observations.csv`, `key_sensitivity_summary.csv`
- `tamper_detection.csv`
- `empirical_collision_matches.csv`
- `environment.json`, `run_summary.json`

Retain the raw CSV files and environment record. Report means, standard deviations, sample counts, and the exact host/software details. Do not copy the manuscript's old reported means into these new results.

## 8. Key handling

By default, a deterministic demonstration key is generated from a fixed seed to make runs reproducible. This is strictly for experiments. For a separate controlled run, pass a 32-byte key encoded as 64 hexadecimal characters:

```bash
python src/run_experiments.py --dataset ../SMI-80-2026 --output results_keyed --key-hex YOUR_64_HEX_CHAR_KEY
```

Do not publish the actual key. The program records only a SHA-256 fingerprint of the key in `environment.json`. Never use the demonstration key for production protection.

## 9. Interpretation and limitations

- SMI-80-2026 is a synthetic byte-level benchmark, not the historical pilot data and not a representative sample of real multimedia usage.
- The media are not decoded for semantic analysis. The algorithm processes raw bytes; file-format differences should not be interpreted independently of file size.
- Some files use size padding to meet the target byte ranges. Validate media decoder compatibility before making container-conformance claims.
- The proposed construction is a deterministic keyed preprocessing layer followed by SHA-256. This implementation does not establish security improvement over SHA-256 or HMAC-SHA-256.
- Avalanche effect, key sensitivity, byte entropy, no observed duplicates, and tamper detection are empirical diagnostics, not proofs of cryptographic security.
- The experiment is local. It does not simulate live cloud retrieval, remote auditing, adversarial cloud behavior, chosen-message forgery, related-key attacks, or key recovery.
- Benchmark timing is machine-, interpreter-, and implementation-dependent. Compare methods only within the same execution environment and matched input workload.

## 10. Reproducibility checklist

Before reporting results:

1. Archive the exact dataset ZIP and its SHA-256 checksum.
2. Record the Git commit hash and Python version.
3. Preserve the exact command line, environment JSON and raw CSV outputs.
4. Use the same key for matched method comparisons; protect it separately.
5. Repeat the timing run on the same host and report the number of repetitions.
6. State clearly that the new dataset experiment is separate from the historical pilot.

## 11. License and citation

The code is released under the MIT License. The synthetic dataset is generated for research experimentation and is described separately in its own README. Cite the manuscript and identify the dataset as SMI-80-2026 when reporting results.
