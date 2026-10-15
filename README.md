# Extra-slot fitting for under-counted, correlated EEG sources

Code, pre-registrations and raw results for the paper

> *Extra-slot fitting reduces compromise locations when correlated EEG sources are under-counted*

When a least-squares multi-source fit (here: alternating projection, AP) is given one source fewer than are
active, it is often drawn to a *compromise* location between two correlated sources, because that location
explains the data better than either true source. The safeguard tested here fits one extra slot and keeps the
best subset of the assumed size **without re-fitting it**. All tests were pre-registered; all of them are
reported, including those in which the method did not help.

Every number in the paper can be regenerated from `results/` in about a minute (no simulation needed). Re-running
the simulations needs the input data listed below.

## Requirements

* Python ≥ 3.10, NumPy ≥ 2, SciPy, Matplotlib (`pip install -r requirements.txt`)
* MNE-Python, for the head models and for reading the recordings. The results were produced with the development
  version (commit `7ab5922`, 13 October 2026). With a released MNE version, `eeg2_setup.py` may need the montage
  name `standard_1005` instead of `fsaverage_1005`.

## Input data (only for re-running simulations)

Set these environment variables (or edit `paths.py`):

| Variable | What | Source |
|---|---|---|
| `MNE_TESTING_DATA` | MNE testing dataset (needs `MEG/sample/` and `subjects/sample/bem/`) | `mne.datasets.testing` or github.com/mne-tools/mne-testing-data |
| `EEGLAB_SET` | `eeglab_data.set` | github.com/sccn/eeglab, `sample_data/` |
| `LEADFIELD_DIR` | `Leadfield2.mat`, `voxelLocation2.mat` (70-channel lead field of Sotudeh et al. 2023) | from the authors on request |
| `DS003775_EDF` | `sub-015_ses-t1_task-resteyesc_eeg.edf` | OpenNeuro ds003775 (SRM resting-state EEG, CC0) |

## Quick start: regenerate the paper's numbers and figures from the stored results

```bash
cd results
python3 ../hybrid_analyse.py 'hy[0-9].txt'     # MEG test            (hybrid_result.txt)
python3 ../hybrid_analyse.py 'ee[0-9].txt'     # EEG test            (eeg_result.txt)
python3 ../hybrid_analyse.py 'hm[0-9].txt'     # head-model error    (step12_partA_result.txt)
python3 ../hybrid_analyse.py 'e2a[0-9].txt'    # EEGLAB, arm A       (eeg2_result.txt)
python3 ../hybrid_analyse.py 'e2b[0-9].txt'    # EEGLAB, arm B
python3 ../hybrid_analyse.py 'lf[0-9].txt'     # 70-ch lead field    (lf70_result.txt)
python3 ../hybrid_analyse.py 'lr[0-9].txt'     # 64-ch, real background (lf64r_result.txt)
python3 ../lf64r_arms.py                       # its descriptive arms (lf64r_arms.txt)
python3 ../step12_partB.py                     # Table 2 + reproduction check (step12_partB_result.txt)
python3 ../runtime_table.py                    # Table 3 (runtime_table.txt)
python3 ../mech_analyse.py                     # section 4.1 (mech_result.txt)
python3 ../theory_check.py                     # equations (3)-(5)
cd .. && python3 make_paper_figures.py         # Figures 1-3 -> figures/
```

## Re-running the simulations

Run everything from `results/` (setup scripts write their `.npz` inputs there). Run `hybrid_setup.py` before `eeg_setup.py`, which checks that both use the same sources and grid. `OPENBLAS_NUM_THREADS=1` was used
throughout; each test was split over two processes (`PART` = 0 and 1 of `NPARTS` = 2).

| Test (pre-registration) | Commands |
|---|---|
| MEG, sample subject (`PREREGISTRATION.md`) | `../hybrid_setup.py`; `../hybrid_run.py PART 2 15 > hyPART.txt` |
| EEG, sample subject (`PREREGISTRATION_EEG.md`) | `../eeg_setup.py`; `../eeg_run.py PART 2 15 > eePART.txt` |
| EEGLAB recording (`PREREGISTRATION_EEG_REPLICATION.md`) | `../eeg2_setup.py`; `../eeg2_run.py PART 2 15 > e2aPART.txt`; `SNRS=2,8 ../eeg2_run.py PART 2 15 850000 > e2bPART.txt` |
| 70-ch lead field, simulated background (`PREREGISTRATION_LF70.md`) | `../lf70_run.py diag`; `../lf70_run.py PART 2 15 2,8 > lfPART.txt` |
| Head-model error (`PREREGISTRATION_STEP12.md`, part A) | `../make_bem_skull0125.py`; `../hme_setup.py`; `../step12_run.py hme-diag`; `../step12_run.py hme PART 2 15 2,8 1000000 > hmPART.txt` |
| Baselines and timing (`PREREGISTRATION_STEP12.md`, part B) | `../step12_run.py eeg PART 2 15 0.5,2 700000 > bePART.txt`; `../step12_run.py lf70 PART 2 15 2,8 900000 > blPART.txt`; `../retime_baselines.py eeg PART 2 'be[0-9].txt' > rt_bePART.txt` (likewise `lf70`/`bl`, `hme`/`hm`) |
| 64-ch lead field, real background (`PREREGISTRATION_LF64REAL.md`) | `../lf64r_run.py diag`; `LF64_MAP=refined ../step12_run.py lf64r PART 2 15 2,8 1100000 > lrPART.txt`; arms: `LF64_MAP=rows1-64` → `lrA`, `LF64_MAP=recovered` → `lrB` |
| DCBF, post hoc | `../dcbf_eeg.py 0 1 10 1000 > dcbf_eeg.txt` |
| Mechanism (section 4.1) | `../mechanism.py eeg > mech_eeg.txt`; `../mechanism.py meg > mech_meg.txt`; `../mech_compare.py eeg` (likewise `meg`, `lf70`, `lf64r`) |
| Fixed-orientation floor (section 4.6) | `../grid_floor.py` |
| Baseline sanity check | `../baseline_sanity.py` |
| Channel order of the 70-row lead field | `../chancheck/validate_tests.py`; `../chancheck/recover_order.py`; `../chancheck/refine_and_inspect.py` → `recovered_order.npy`, `refined_order.npy` |

A re-run reproduces the stored results exactly (same seeds; checked for the EEG and real-background settings).
Timings depend on the machine.

## Pre-registrations

Each document was written before its test was run and fingerprinted with SHA-256 and a UTC time stamp
(`preregistration/*.sha256`; documents with amendments list one line per version). They were kept with the code,
not deposited in a public registry, so the fingerprints are what allow later changes to be detected.

| Document | Test | Final SHA-256 (first 12) | Time stamp (UTC) |
|---|---|---|---|
| `PREREGISTRATION.md` | MEG, sample subject | `03ab5cadbcab` | 2026-10-15 03:21 |
| `PREREGISTRATION_EEG.md` | EEG, sample subject | `4e43edb74ac9` | 2026-10-15 03:34 |
| `PREREGISTRATION_EEG_REPLICATION.md` | EEGLAB recording | `6fbc423d3ac7` | 2026-10-15 03:43 |
| `PREREGISTRATION_LF70.md` | 70-ch lead field | `bc0ce50c228b` | 2026-10-15 03:54 |
| `PREREGISTRATION_STEP12.md` | head-model error; baselines and timing | `1643e9a671a2` | 2026-10-15 04:10 |
| `PREREGISTRATION_LF64REAL.md` | 64-ch lead field, real background | `8821f03a850f` | 2026-10-15 05:32 |

## Files

| File | Purpose |
|---|---|
| `core/modelorder.py`, `core/lsq_generic.py`, `core/lsq_methods.py` | AP, restarts, extra-slot fits, scoring |
| `core/dcbf.py` | dual-core beamformer and its Powell search |
| `baselines.py` | TRAP-MUSIC and LCMV peaks (`baselines_v1.py`: first, slower version; identical picks) |
| `*_setup.py`, `*_run.py`, `step12_run.py` | head models, simulations and runners |
| `hybrid_analyse.py` | pre-registered decision rule for any test |
| `make_paper_figures.py` | Figures 1-3 |
| `results/` | raw per-realisation outputs (one JSON line each), summaries, `STEP3_THEORY.md` |

## Licence

MIT
