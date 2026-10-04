# Nerve responses to high-frequency stimulation trains

Statistical analysis for an honours thesis (UNSW) by Silver Rose.

Trains of electrical pulses were delivered to the ulnar nerve. Three responses were
recorded to every pulse at the same time:

- **Motor:** the compound muscle action potential (CMAP) from abductor digiti minimi (ADM)
- **Sensory:** the sensory nerve action potential (SNAP)
- **Mixed nerve:** the response recorded at the elbow

The two analyses ask how these responses change during a train and recover after it:

- **AC analysis** (`ac_analysis.rmd`): the effect of stimulation frequency (10, 25, 50
  and 100 Hz).
- **DC analysis** (`dc_analysis.rmd`): the effect of a DC current applied during a 25 Hz
  train (depolarising, hyperpolarising or no DC). Only the motor and sensory channels
  are analysed.

## Repository layout

```
scripts/
  ac_analysis.rmd       AC analysis
  dc_analysis.rmd       DC analysis
outputs/data/           input data (and one output, see below)
figures/                main figures, written by the scripts (PNG and EMF)
```

## Running the analysis

1. Install R (written with R 4.6) and these packages:

   ```r
   install.packages(c("rmarkdown", "knitr", "ggplot2", "cowplot", "scales", "devEMF",
                      "lme4", "lmerTest", "emmeans", "car", "DHARMa",
                      "GLMMadaptive", "survival"))
   ```

2. Knit each script from the `scripts/` folder. The data paths are relative to it
   (`../outputs/data/`).

   ```r
   rmarkdown::render("scripts/ac_analysis.rmd")
   rmarkdown::render("scripts/dc_analysis.rmd")
   ```

   Each script knits to a Word document next to the script. It also writes its main
   figures to `figures/` and overwrites the files already there. The two scripts are
   independent and can be knitted in either order.

The setup chunk in each script clears the workspace, so chunks must be run in order if
you run them by hand.

## Analysis outline

Both scripts follow the same steps:

1. **Noise subtraction.** The response to the 2 mA step of each recording's recruitment
   curve is below threshold. It is taken as that recording's noise floor and subtracted
   from every pulse; values below zero are set to 0.
2. **Artefact blanking.** Pulses inside time windows marked as artefact in Spike2
   (`tagged areas.csv`, `dctaggedareas.csv`) are set to `NA` on the affected channels.
3. **Exclusions.** Participant S03 is excluded from both analyses. The AC script also
   excludes individual channels for some recordings.
4. **Normalisation.** Each response is divided by the mean of the single pulses delivered
   before the train, so 1 means no change from baseline.
5. **Figures and models:**
   - group-mean responses during the train and during recovery after it
   - the end-of-train response (mean of the last 5 s), modelled with
     `lmer(response ~ condition + (1 | subject))`
   - time to sustained recovery (the first of two consecutive probes above 0.9 of
     baseline). This is modelled as a censored log-normal mixed model (`GLMMadaptive`),
     because some recordings never recover within the window.

Within-train time is not recorded in the exports. It is reconstructed from the pulse
index as `count / frequency`.

## Data

All data are de-identified. Participants are identified only by study codes (S01–S18).

| File | Contents |
|---|---|
| `ACpulsedata.csv` | AC study: one row per stimulus, from the Spike2 exports. `measure` is `Singles` (pre-train), `Train`, `Recovery` or `Mmax`. |
| `RecruitmentCurves.csv` | AC study recruitment curves. Used only for the 2 mA noise floor. |
| `tagged areas.csv` | AC artefact windows (`start`, `end` in recording time) and the channel groups each affects. |
| `dc analysis.csv` | DC study: one row per stimulus. Adds `DC` (condition) and `DCamp`. |
| `dc recruitment curve.csv` | DC study recruitment curves. Used only for the 2 mA noise floor. |
| `dctaggedareas.csv` | DC artefact windows. |
| `noisereduceddcdata.csv` | **Output** of `dc_analysis.rmd`: the DC data after noise subtraction, artefact blanking and exclusions. Rewritten on every knit. |
| `ac_trace_*.txt`, `dc_trace_*.txt` | Spike2 text exports of raw waveforms, used for the raw-trace figures. The name gives the frequency or DC condition and whether it is the start or end of the train. |

Main columns in the pulse files:

| Column | Meaning |
|---|---|
| `filename` | Source Spike2 recording |
| `subjID` | Participant code |
| `freq` | Stimulation frequency (Hz) |
| `currAmp` | Stimulus current (mA) |
| `count` | Pulse index within the recording and `measure` |
| `ADMpk2pk` | Motor (ADM CMAP) peak-to-peak amplitude (mV) |
| `SNAPpk2pk`, `filtSNAPpk2pk` | Sensory (SNAP) peak-to-peak, unfiltered and filtered (mV) |
| `elbowpk2pk`, `filtelbowpk2pk` | Mixed-nerve (elbow) peak-to-peak, unfiltered and filtered (mV); AC study only |

## Figures

`figures/` holds each main figure as a PNG (for viewing) and an EMF (a Windows vector
format, for Word). Names follow `<ac|dc>_<figure>_<channel>`, for example
`ac_group_mean_train_motor.png`. The model diagnostic plots appear only in the knitted
Word documents.
