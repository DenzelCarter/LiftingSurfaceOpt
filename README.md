Repo Structure:
├─ Experiment/
│  ├─ data/
│  │  ├─ hover/{raw, tare, comsol} # contains automatic testing script for 1585 flight stand with discrete stepping, link 1585 thrust stand to the raw folder and dump files here, put no-load run in tare folder
│  │  ├─ cruise/{raw, tare, comsol} # comsol data only so far
│  │  └─ processed/          # master parquet
│  ├─ outputs/
│  │  ├─ models/             # .pkl pipelines
│  │  ├─ plots/              # PDFs (all seen in paper)
│  │  └─ tables/             # CSVs (diagnostic and used by scripts)
│  └─ doe/
│     ├─ doe_initial.csv     # design-of-experiments geometries (hover/cruise)
│     └─ doe_optima.csv      # (optional) active-learning additions
├─ Scripts/
│  ├─ 01_process_data.py     # INPUT: data/hover/raw data from 1585 flight stand and cruise comsol files, use no-load tare in data/hover/tare, perform step-level estimator selection to balance efficiency and robustness, OUTPUT: master_dataset.parquet with per-step performance and variance, diagnostics csv for noise characterization 
│  ├─ 02_train_models.py     # INPUT: master_dataset.parquet from 01, trains two GPR models on hover and cruise separately using geometry and operation features to predict performance, variance fed into GPR per step to model heteroscedasticity, OUTPUT: two GPR models for predictions of performance and forming CIs with model variance
│  ├─ 03_validate_models.py  # INPUT: GPR models, performs LOOCV on geometry specifically to be fair since operation speeds of the same geometry are too similar, singles out a hover and cruise unique combo of geometry and operation as a case study (picks ones with most steps so you can get a nice curve of predicted vs measured), OUTPUT: csvs with predictions, interval calibration for confidence intervals (CIs), metrics for performance, diagnostics
│  ├─ 04_ucb_pareto_search.py # INPUT: GPR models, interval calibrations, uses Upper Confidence Bound (UCB) objective function to balance exploitation and exploration of the design space, creates pareto front of optimal designs to test in future active learning iterations, OUTPUT: csvs with lots of outputs related to UCB and diagnostics
│  ├─ 05_reoperate_baselines.py # INPUT: Reoperates baseline designs to see if UCB is actually better by geometry as well and not just by operation, OUTPUT: baseline performances
│  └─ 99_build_figs.py        # INPUT: csvs from all the previous scripts, contains many plotting functions to produce all plots used in the paper for future diagnostics of iterative design, OUTPUT: pdf plots in the outputs/plots folder
│  └─  config.yaml            # INPUT: configure geometry accordingly, tune optimization parameters, add model features to train on, adjust bounds to optimize over, adjust dynamic kappa scheduling
└─ README.md
