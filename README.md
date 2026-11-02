# Reproducibility

Numerical data, model weights, simulation/analysis code and result plots. Download reproducibility.zip from Releases and extract it before following the instructions below.

## Contents

- `code/`: simulator, constrained network operation, recurrent multi-agent learning, evaluation, repair replay, statistics and plotting.
- `data/`: physical configuration, scenarios, input temperatures, training parameters and 20 selected model weights.
- `results/`: 10,800 evaluation outcomes, 96,158 repair-completion effects, training curves, plotting tables and numerical summaries.

Original numerical arrays and model tensors are preserved. File layout and metadata were exported for portability. Model files contain inference weights and architecture information; optimiser and training-resume state are omitted.

## Run

Use Python 3.12. Tested with Python 3.12.9 on macOS arm64. CPU execution is supported; no GPU is required. Install the dependencies in a separate environment:

```bash
python -m pip install -r requirements.txt
python code/reproduce.py analyze
python code/reproduce.py plot
python code/reproduce.py check-models
```

Analysis recomputes aggregate service, paired contrasts and weighted repair effects, then checks them against the saved results. Add `--bootstrap-draws 10000` to regenerate paired bootstrap intervals. Analysis takes approximately 3–5 seconds and model loading approximately 10 seconds on the tested machine; installation time depends on downloads. Outputs are written to `generated/`.

Run one complete 336-hour evaluation and compare its service trajectory with the saved outcome:

```bash
python code/reproduce.py simulate --condition full --seed 20260821 --scenario 2090
```

Use `--all-cases` to run all 400 test scenarios for that condition/seed. Use `--repair-replay` to compute same-state repair-completion effects. Available evaluation conditions are `full`, `fixed_parameters`, `no_preparation`, `no_restoration`, `service`, `no_intervention` and `heuristic`. The five learned-policy seeds are 20260821–20260825.

For training from scratch, use `python code/reproduce.py train --condition full --seed 20260821`. Repeat for the four learned conditions and five seeds. `data/training.yaml` records the 500-update configuration. `--updates` permits a shorter development run and does not reproduce a completed 500-update training run. Selected model updates are listed in `data/models.json`; selection uses the minimum full-validation objective, breaking ties by earlier update. Training and full evaluation are much more expensive than offline analysis. They have not all been repeated for this export.

## Data conventions

`results/episodes/*.npz` can be read with `numpy.load(..., allow_pickle=False)`. Each row is one episode, identified by `training_seed` and `scenario_id`. `service` has 336 hourly step-end values. `q_trajectories__*`, `total_loss_trajectories__*` and `category_trajectories__*` retain system losses and category service. `metric_names` labels the columns of `metrics`. `stratum`, `policy_seed` and `environment_seed` preserve grouping and deterministic replay inputs.

Service index is the mean service fraction; service loss-hours are the sum of `1-service`. Sustained recovery requires service to remain above the threshold to the end. Unrecovered episodes use 337 h for restricted-recovery comparisons. Repair gains are one-hour, same-state effects. Detailed historical replay trace files are omitted; event-level values needed for the saved population analysis are included, and the recorder can regenerate individual traces.

`full` uses preparedness and restoration; `fixed_parameters` disables preparedness while retaining the full policy weights; `no_preparation` and `no_restoration` are separately trained; `service` uses the service-augmented objective. Source plotting tables also use `infra/full/F`, `frozen/M`, `retrained/R`, `infra_no_c/N` and `service/S` for these conditions.

Plots reproduce numerical results in a simple layout. Editorial artwork is not included. Files are verified by `checksums.sha256`; the file list contains only paths within this package.

## Terms and input sources

Original materials are provided under the custom [Academic Reproduction License](LICENSE). No individual application is needed to download, run, modify or retrain them solely for non-commercial academic reproduction and verification of the methods and findings supported by this package. Report verification findings with attribution to the source and version used. Commercial use, other research uses, and redistribution of the original or modified materials require separate written permission. This is a restricted-use license, not an open-source license.

Third-party materials are excluded from this license and retain their own terms. Hazard inputs originate from the Hong Kong Observatory. Infrastructure-derived inputs originate from OpenStreetMap contributors under the Open Database Licence, including where they appear alongside original model configuration. Existing third-party attribution and reuse obligations continue to apply. Package binaries for dependencies and map tiles are not included.
