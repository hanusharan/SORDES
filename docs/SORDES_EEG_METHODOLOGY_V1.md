# SORDES EEG Decoding — Research Methodology Specification v1

2026-09-17 · @Someone

This methodology is pre-registered and frozen before experimentation begins: every definition and threshold below is locked and will not be revised after results are seen. Actual measured values will be reported even where a criterion is not met.

## 1. Neural Class Vocabulary

The EEG decoder predicts exactly 4 motor-imagery classes — nothing else:

| Class | Meaning |
| --- | --- |
| `MI_LEFT` | Left-hand motor imagery |
| `MI_RIGHT` | Right-hand motor imagery |
| `MI_FEET` | Feet motor imagery |
| `MI_TONGUE` | Tongue motor imagery |

These are **decoder outputs, not SORDES commands**. A separate, deterministic context/mapping layer converts a decoded class plus the current system context (locomotion mode vs. manipulation mode) into an actual command such as `WALK`, `STOP`, `TURN_LEFT`, `TURN_RIGHT`, or a `GRASP_OBJECT_x` — the decoder is never trained to predict commands directly, since those aren't the motor-imagery classes the public data actually contains.

No `IDLE`/rest class is included. Uncertainty is handled entirely by the confidence/safety gate (Blender spec, already locked) rather than a 5th trained class — consistent with BCI IV-2a's cued paradigm, which has no continuous no-intent period to train a rest class against anyway.

## 2. Dataset & Session Structure

**Primary dataset: BCI Competition IV, Dataset 2a.** Chosen because its 4 native classes (left hand, right hand, feet, tongue) match the locked vocabulary exactly, and it provides two recording sessions per subject on different days — directly aligned with the cross-session research question.

| Property | Value |
| --- | --- |
| Subjects | 9 |
| Classes | 4 (matches locked vocabulary) |
| EEG channels (decoder input) | 22 |
| EOG channels | 3 — artifact/reference only, never a decoder input |
| Sessions per subject | 2, on different days: **T** (training) and **E** (evaluation) |
| Trials per session | 288 (72 per class) |

The **T/E split is the backbone of the whole methodology**: a model trained on session T and evaluated on session E is, by construction, a cross-session evaluation — the exact real-world scenario (put the headset on a different day) that the research question is about.

PhysioNet EEGMMIDB is explicitly **not** part of the primary methodology — it may be added later as an external generalization check if time permits, but is out of scope for the pre-registered experiment.

## 3. Baseline & Adaptive Conditions

Three precisely defined reference points, all using the same preprocessing and evaluation protocol so they're comparable:

| Condition | Definition |
| --- | --- |
| **Cross-session baseline** | Model trained only on session T, evaluated directly on session E, with zero adaptation. Quantifies the raw performance drop caused by the session shift. |
| **Full-retrain (upper reference)** | The same model architecture trained from scratch using 100% of session E's own training data. Represents best-case, unlimited-calibration performance — the ceiling the adaptive condition is measured against. |
| **Adaptive (recalibrated)** | The T-trained model adapted using a limited fraction of session E's data — swept at 5%, 10%, 20%, 50%, 100% per the calibration efficiency curve below. |

These three conditions are what the recovery-efficiency formula in the next section directly compares.

## 4. Primary Success Criterion — Recovery Efficiency

```text
Recovery = (Adaptive − Cross-session baseline) / (Full-retrain − Cross-session baseline)
```

**Pre-registered threshold:** the adaptive model must achieve **Recovery ≥ 0.50** using **≤20%** of the target-session (session E) calibration data that full retraining would require.

Recovery can fall outside [0, 1] — a value above 1 means adaptation outperformed even the full-retrain ceiling (plausible, since adaptation also benefits from source-session knowledge the full-retrain condition discards); a value below 0 means adaptation performed worse than doing nothing. Both outcomes will be reported exactly as measured, not treated as invalid results.

## 5. Calibration Efficiency Curve

Recovery (and the underlying accuracy/F1) is measured at six calibration-data fractions: **0%, 5%, 10%, 20%, 50%, 100%** of available target-session (session E) training data.

The key reported result is the **full performance-vs-calibration curve** across all six points — not a single pass/fail number at 20%. The 20% point is what the primary threshold in Section 4 is checked against, but the curve as a whole is what shows how adaptation actually behaves as more calibration data becomes available.

## 6. Safety Criterion — False Activation & Command Acceptance

**False activation, defined specifically for this 2a-based evaluation:** a wrong-class prediction that passes the confidence gate and is therefore accepted as an executable command. This is deliberately **not** a claim of measuring asynchronous/continuous no-intent false activations — the 2a paradigm is cued and forced-choice, so it provides no continuous no-intent period to measure that kind of false activation against.

**Pre-registered thresholds, both must hold at the chosen gate threshold:**

- Confidence gating reduces this false-activation rate by **≥50%** relative to the ungated decoder
- While retaining **≥90%** of correctly detected commands (≤10% loss in true-command acceptance)

The operating confidence threshold is chosen from a sweep to satisfy both conditions simultaneously — it is not a single arbitrarily-picked cutoff.

## 7. Responsiveness — Latency Reporting

**Headline metric:** end-to-end system latency, reported as measured with no invented universal pass/fail threshold on this composite number.

**Reported separately, as components of that same end-to-end number** (not excluded from it):

- EEG acquisition/window duration
- Preprocessing time
- Inference time
- Command transmission/bridge time

**The ≤1 s target applies only to the post-window portion** — preprocessing + inference + transmission combined — and explicitly **excludes** the EEG acquisition window itself. The acquisition window is real, unavoidable end-to-end latency for a motor-imagery system and will be reported honestly as part of the full latency figure, not minimized or hidden to make the ≤1 s target look better than the system actually is.

## 8. Pre-registration Commitment

Every definition and threshold in this document — the recovery formula and its 0.50/20% threshold, the calibration sweep points, the false-activation definition and its 50%/90% thresholds, and the latency scoping — is **frozen as of this lock date** and will not be revised after experimental results are seen.

Actual measured values are reported in full even where SORDES fails one or more criteria. Nothing here is adjusted post hoc to make results look more favorable.
