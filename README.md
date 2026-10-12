# EPS 2026 · Change Detection

An educational research project for detecting and classifying changes in co-registered aerial and satellite images captured before and after an event. The **BIT-CD** model has two heads: one detects changes, and the other predicts their type.

**Authors:** [@hubinmykyta](https://github.com/hubinmykyta), [@o-haid](https://github.com/o-haid), [@anbar-root](https://github.com/anbar-root).

| Class | Source of training labels |
|---|---|
| Destroyed / demolished object | xBD Global; FOTBCD-Instances `DEMOLISHED` |
| New crater | Synthetic generator |
| New fortification | Synthetic generator |
| New construction | FOTBCD-Instances `NEW` |

Outputs include a class mask, a probability map, an image with bounding boxes, and an object list with class labels, areas, and confidence scores. The Gradio demo supports downloading results as PNG and JSON files.

> Craters and fortifications are trained and evaluated on synthetic data. These results do not establish performance on real imagery of these objects. Performance on xBD-Ukraine is low; this project is a research prototype.

## Project files

| File | Purpose |
|---|---|
| [EPS_ChangeDetection_BITCD_Semantic_UA.ipynb](EPS_ChangeDetection_BITCD_Semantic_UA.ipynb) | Data preparation, synthetic data generation, data audits, training, validation / testing, and reporting |
| [EPS_ChangeDetection_Demo_Gradio_UA.ipynb](EPS_ChangeDetection_Demo_Gradio_UA.ipynb) | Interactive demo with a pretrained model |
| [requirements.txt](requirements.txt) | Dependencies for local execution |
| README.md | Project overview, setup, results, and references |

The notebooks and demo interface are in Ukrainian. Notebook outputs have been cleared to make changes easier to review. The saved experiment report is included below.

## Quick start: demo in Colab

1. Open [Google Colab](https://colab.research.google.com/) and upload `EPS_ChangeDetection_Demo_Gradio_UA.ipynb` through **File → Open notebook → Upload**, or use the **GitHub** tab to open it from the repository URL.
2. Select **Runtime → Change runtime type → T4 GPU**.
3. Keep `WEIGHTS_SOURCE='shared'` in the first code cell and select **Run all**.
4. In Gradio, upload a before / after image pair and run change detection. If known, enter the ground sampling distance (GSD) in meters per pixel.

The model and threshold are downloaded automatically from Google Drive, verified using SHA-256, and cached. The demo does not require a personal Drive, training data, or retraining.

| File on Drive | Purpose |
|---|---|
| [best.pt](https://drive.google.com/file/d/1jtceZDX4cNY9udOLS_6zsN8ptSNVQozj/view) | Trained model with a change-type head, epoch `29` |
| [threshold.json](https://drive.google.com/file/d/1RAjId11DcGvu1hq0K4JN4hJhDT32R1Bs/view) | Threshold `0.31` for this model |

[Shared artifacts folder](https://drive.google.com/drive/folders/1zWs2m9bG5eTe_NGnyp4ElNFQKbk0Ah4w). Download links and checksums are embedded in `SHARED_FILES` in each notebook; both notebooks include their own download helpers.

To use **your own model**, set `WEIGHTS_SOURCE='drive'`. In Colab, this mounts your personal Drive and expects `best.pt` and `threshold.json` in `MyDrive/EPS_2026/runs/bitcd_semantic_v7/`. Optional examples can be placed in `MyDrive/EPS_2026/demo_pairs/` as `<name>_pre.png`, `<name>_post.png`, and `<name>_label.png`. Without a custom threshold file, the demo uses `0.5`.

The demo runs with `share=True`, creating a temporary public Gradio link. To use a local interface, change this setting to `False` in the final code cell.

## Training and evaluation

Open the main notebook in Colab with a T4 GPU. For an experiment comparable to the saved report, place `FOTBCD-Instances.zip` or the `FOTBCD-Instances/` directory in `MyDrive/EPS_2026/`. In section **1.2 (Configuration)**, check the paths, `RUN_NAME`, and execution flags, then select **Run all**.

The main notebook uses [BIT_LEVIR_best_ckpt.pt](https://drive.google.com/file/d/1Qoh-0_Ub3xOdwRphfT_I4lXksm9nEZMK/view), the initial weights of a binary BIT-CD model trained on LEVIR-CD, for fine-tuning and baseline comparisons. The demo uses the trained `best.pt` and its `threshold.json`.

- Initial LEVIR weights and xBD Global are downloaded automatically. The xBD archive is approximately 8.4 GB; extracted data and caches require additional space.
- If FOTBCD-Instances is unavailable, `auto` mode falls back to FOTBCD-Binary, producing an experiment that differs from the saved run.
- Use `last.pt` to resume training. Use a new `RUN_NAME` for a new experiment.
- With `RUN_TRAIN=False` and `DOWNLOAD_SHARED_WEIGHTS=True`, a missing `best.pt` is downloaded from the shared link. Evaluation still requires data whose fingerprint matches the checkpoint.
- The threshold is selected on the validation set. The test set is evaluated once with `FORCE_RETEST=False`. Without the `val_hist.npz` cache, the main notebook recomputes validation statistics and the threshold.

## Results and limitations

The results below come from the saved experiment. Checkpoint: epoch `29`, threshold `0.31`, window size `256`, stride `192`, `TTA=False`, data fingerprint `622605ccb7ea0133`.

| Test set | Data type | Binary pixel F1 |
|---|---|---:|
| FOTBCD-Instances | Real | 0.764 |
| xBD-Global | Real | 0.642 |
| xBD-Ukraine | Real, different domain | 0.039 |
| SYN-ID | Synthetic | 0.754 |
| SYN-SHIFT | Synthetic, with modified generator parameters | 0.763 |
| FOTBCD + xBD | Real, combined | 0.713 |

On a Tesla T4, a `1024×1024` image pair takes **0.151 s** on average, with a p95 of **0.164 s** and a maximum of **0.165 s** across 20 measurements. This measures processing from arrays in memory to the class map and object list; file reading and interface overhead are excluded.

Images must be geometrically aligned; resizing in the demo does not perform image registration. Real-world destruction labels primarily describe buildings. Performance depends on the domain and image resolution; area estimates in m² depend on the supplied GSD. Confidence scores are not calibrated probabilities.

<details>
<summary>Full saved report: per-class and object-level metrics</summary>

### Summary: Change Detection (case 4.2)

- Model: BIT-CD (LEVIR initialization) + change-type head; checkpoint epoch 29
- Threshold (selected on validation): 0.31; inference: window size 256, stride 192, TTA=False
- Data: FOTBCD (instances), xBD Global (spatial split), synthetic data (ID + Shift), xBD-Ukraine (test only)
- Data fingerprint: 622605ccb7ea0133

#### Accuracy (TEST, required F1 ≥ 0.75)

| Source | Data type | Binary pixel F1 | Status |
|---|---|---|---|
| FOTBCD-Instances | Real | 0.764 (P 0.809, R 0.723) | PASS |
| xBD-Global | Real | 0.642 (P 0.662, R 0.623) | FAIL |
| xBD-Ukraine | Real, OOD | 0.039 (P 0.053, R 0.030) | FAIL |
| SYN-ID | Synthetic | 0.754 (P 0.717, R 0.796) | PASS |
| SYN-SHIFT | Synthetic, with a generator shift | 0.763 (P 0.739, R 0.789) | PASS |
| REAL (FOTBCD+xBD) | Real, combined | 0.713 (P 0.747, R 0.682) | FAIL |

| Source | Class | Pixel F1 | Object F1 | Status (pixel F1) |
|---|---|---|---|---|
| FOTBCD-Instances | Destroyed object | 0.635 | 0.235 | FAIL |
| FOTBCD-Instances | New construction (other change) | 0.791 | 0.672 | PASS |
| xBD-Global | Destroyed object | 0.644 | 0.605 | FAIL |
| xBD-Ukraine | Destroyed object | 0.050 | 0.024 | FAIL |
| SYN-ID | Destroyed object | 0.506 | 0.300 | FAIL |
| SYN-ID | New crater | 0.857 | 0.785 | PASS |
| SYN-ID | New fortification | 0.777 | 0.712 | PASS |
| SYN-ID | New construction (other change) | 0.716 | 0.576 | FAIL |
| SYN-SHIFT | Destroyed object | 0.573 | 0.457 | FAIL |
| SYN-SHIFT | New crater | 0.852 | 0.771 | PASS |
| SYN-SHIFT | New fortification | 0.728 | 0.719 | FAIL |
| SYN-SHIFT | New construction (other change) | 0.695 | 0.564 | FAIL |

**Synthetic shortcut test:** false positives on pre-existing objects: SYN-ID, 2.8% of objects / 1.3% of pixels; SYN-SHIFT, 6.8% of objects / 2.9% of pixels.

#### Runtime (required ≤ 2 s per 1024×1024 pair)

- Tesla T4: mean 0.151 s, p95 0.164 s, maximum 0.165 s → **PASS** (real test image pairs, 1024×1024)
- With TTA×4: maximum 0.578 s; PNG reading measured separately at approximately 0.086 s

#### Reference: zero-shot LEVIR BIT-CD (validation, threshold 0.5)

FOTBCD-Instances: F1 0.380, xBD-Global: F1 0.043, SYN-ID: F1 0.146, REAL (FOTBCD+xBD): F1 0.304

</details>

## Local setup

Colab is the primary execution environment. To use Jupyter locally:

```bash
python -m venv .venv
# PowerShell: .\.venv\Scripts\Activate.ps1
# Linux / macOS: source .venv/bin/activate
python -m pip install -r requirements.txt
jupyter lab
```

For GPU execution, install a compatible PyTorch / torchvision pair using the [official instructions](https://pytorch.org/get-started/locally/). `requirements.txt` lists direct dependencies; it is not an exact lock file for the experiment.

Outside Colab, `EPS_WORK` sets the working directory. In the demo's `shared` mode, weights are cached in `WORK/assets/runs/bitcd_semantic_v7/`. In `drive` mode, `EPS_DRIVE_ROOT` sets the directory containing your own files. Running the full training pipeline on Windows may require adapting multiprocessing code.

## References and usage terms

- **BIT-CD:** Hao Chen, Zipeng Qi, Zhenwei Shi, *Remote Sensing Image Change Detection with Transformers*, 2021, [DOI](https://doi.org/10.1109/TGRS.2021.3095166), [code and initial LEVIR weights](https://github.com/justchenhao/BIT_CD). The original [License section](https://github.com/justchenhao/BIT_CD#license) specifies non-commercial and research use.
- **FOTBCD:** Abdelrrahman Moubane, 2026, [paper](https://arxiv.org/abs/2601.22596), [official repository and data](https://github.com/abdelpy/FOTBCD-datasets), [FOTBCD-Binary](https://huggingface.co/datasets/retgenai/FOTBCD-Binary). FOTBCD-Binary / Instances are listed under CC BY-NC-SA 4.0.
- **xBD:** [original source](https://xview2.org/dataset), [xBD Global mirror used in this project](https://huggingface.co/datasets/DrNerd/xBD-Training-Data), [xBD-Ukraine](https://huggingface.co/datasets/sda-kr/xbd-ukraine). xBD-Ukraine is used only to evaluate transfer to a different domain.

Synthetic objects are generated by the notebook on backgrounds from the corresponding datasets. Third-party data, weights, and libraries are subject to their own usage terms.
