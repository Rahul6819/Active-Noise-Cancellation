# Adaptive Noise Cancellation (ANC)

**EE2800 – Course Project**
Indian Institute of Technology Hyderabad

**Team (Group 14):**
- Revulagadda Rahul — EE23BTECH11053
- Sai Krishna Shanigarapu — EE23BTECH11054
- Sammeta Sai Poorna — EE23BTECH11055

## Overview

This project implements an **Active Noise Cancellation (ANC)** system in MATLAB that removes unwanted noise `v(n)` from a noisy speech signal `s(n) + v(n)`, using a reference noise signal `w(n)` captured by a second microphone. The system supports two modes of operation:

- **Full Suppression (`mode = 1`)** — Uses a conventional **RLS (Recursive Least Squares)** adaptive filter directly on the reference noise to estimate and cancel the noise component.
- **Partial Suppression (`mode = 0`)** — Passes the reference noise through a **notch filter** (centered at a target frequency) before applying RLS adaptation, resulting in a milder/partial cancellation.

### Block Diagram

The reference noise `w(n)` is optionally passed through a notch filter (`mode = 0`) or fed directly (`mode = 1`) into the RLS adaptive filter, which produces an estimate `v'(n)` of the noise. This estimate is subtracted from the noisy speech `s(n) + v(n)` to produce the cleaned output `s(n) + v(n) - v'(n)`.

```
                    ┌──────────────┐
  w(n) ──mode=0────►│ Notch Filter │──┐
        │            └──────────────┘  │
        └─mode=1──────────────────────►├──► RLS ──► v'(n)
                                        │
noisy speech: s(n)+v(n) ───────────────┴──►  (+) ──► s(n)+v(n)-v'(n)
```

## Repository Contents

| File | Description |
|---|---|
| `final.m` | Main MATLAB script implementing both suppression modes, evaluation metrics, and plots |
| `noisy_speech.txt` | Input: speech corrupted with noise, `s(n) + v(n)` |
| `external_noise.txt` | Reference noise signal, `w(n)` |
| `clean_speech.txt` | Ground-truth clean speech, `s(n)` (used for SNR evaluation) |

> **Note:** The `.txt` data files are not included in this repository by default — add your own recordings/signals with matching filenames, or update the `load(...)` paths in `final.m`.

## How It Works

### 1. Full Suppression Mode (`mode = 1`)
A 5-tap conventional RLS adaptive filter (as described in Diniz, *Adaptive Filtering: Algorithms and Practical Implementation*, Ch. 5) is trained directly on the external noise reference `w(n)` to predict and subtract the noise component from the noisy speech.

### 2. Partial Suppression Mode (`mode = 0`)
The reference noise is first passed through a 2nd-order IIR **notch filter** (default center frequency: 1000 Hz, pole radius `r = 0.99`) before being fed into the RLS filter. Since the notch filter is not ideal, frequencies near the target frequency are also attenuated, giving a "partial" cancellation effect with reduced signal quality near that band.

### RLS Parameters
| Parameter | Value |
|---|---|
| Filter order (`N` / `M_RLS`) | 5 |
| Forgetting factor (`λ`) | 0.998 |
| Regularization (`δ`) | 0.01 |

## Outputs

Running `final.m` will:
1. Save the cleaned signal to `cleaned_speech_full.txt` or `cleaned_speech_partial.txt`
2. Play back the cleaned audio (`sound(residual, fs)`)
3. Generate the following plots:
   - Spectrograms (before / after cancellation)
   - FFT magnitude comparison (noisy vs. residual)
   - Time-domain comparison (noisy vs. residual)
4. Print **SNR before**, **SNR after**, and **SNR gain** (in dB) to the console

## Usage

1. Open MATLAB and navigate to the project directory.
2. Ensure `noisy_speech.txt`, `external_noise.txt`, and `clean_speech.txt` are present (sampled at `fs = 44100` Hz).
3. Set the desired mode at the top of `final.m`:
   ```matlab
   mode = 1; % 1 = Full Suppression, 0 = Partial Suppression
   ```
4. Run the script:
   ```matlab
   final
   ```

## Design Choices & Trade-offs

- **RLS over LMS:** RLS was chosen for full suppression due to its faster convergence rate and better stability compared to LMS, at the cost of higher computational complexity (`O(N²)` per iteration).
- **Notch Filter for Partial Suppression:** A simple, low-complexity approach for scenarios where full adaptive cancellation isn't necessary, but it is not ideal — it also attenuates speech content near the notch frequency.
- **Mode Switch:** A single `mode` flag lets the user toggle between full and partial suppression without changing the rest of the pipeline.

## Pros and Cons

**Pros**
- RLS offers strong convergence speed and stability for noise estimation.
- Modular design allows easy switching between suppression strategies.

**Cons**
- RLS has `O(N²)` time complexity, making it computationally expensive for large filter orders.
- The non-ideal notch filter removes useful signal content near the target frequency, reducing output quality in partial suppression mode.

## References

1. P. Diniz, *Adaptive Filtering: Algorithms and Practical Implementation*, pp. 209–212. Springer, 2008.
2. Y. He, H. He, L. Li, Y. Wu, and H. Pan, "The applications and simulation of adaptive filter in noise canceling," in *2008 International Conference on Computer Science and Software Engineering*, vol. 4, pp. 1–4, 2008.
3. H. Kaur and R. Talwar, "Performance comparison of adaptive filter algorithms for noise cancellation," in *2013 International Conference on Emerging Trends in Communication, Control, Signal Processing and Computing Applications (C2SPCA)*, pp. 1–5, 2013.
4. V. Djigan, "Low complexity RLS adaptive filters," in *2022 24th International Conference on Digital Signal Processing and its Applications (DSPA)*, pp. 1–5, 2022.
