# EEG Suppression Annotation UI

A desktop application for automatic EEG suppression detection, visual review, manual correction, and batch annotation. The calibrated model is included and loads automatically.

> [!IMPORTANT]
> This is research software. It is not intended for clinical diagnosis, treatment, or patient monitoring.

## Start with the defaults

No configuration is required for a first run. Load a compatible recording and the application will use the bundled model, all four algorithms, and their calibrated thresholds.

| Setting | Default |
| --- | --- |
| Display window | `10 s` |
| Y limits | `-100` to `100 µV` |
| EEG scaling factor | `1` |
| Minimum suppression duration | `1 s` |
| Algorithms | CNN, Coastline, Westover, OpenIBIS |
| Thresholds | CNN `0.385`, Coastline `188`, Westover `14.6053`, OpenIBIS `11` |

Use the defaults when your EEG is already on the expected amplitude scale. If your data uses a different scale or you need different detection behavior, every relevant setting can be changed inside the application.

## Input data

The application expects one MATLAB `.mat` file per recording.

| Field | Required content |
| --- | --- |
| `EEG` | Top-level scalar MATLAB structure |
| `EEG.data` | Single-channel numeric sample vector; row or column vectors are accepted |
| `EEG.srate` | Numeric scalar equal to `250` Hz |
| Recording length | At least `1` second |

Example MATLAB export:

```matlab
EEG.data = single(eeg_samples);
EEG.srate = 250;
save("patient.mat", "EEG");
```

The default scale factor is `1`, so values are used as supplied and displayed as µV. If the numerical scale differs, adjust **Settings → Signal → EEG scaling factor**. For example, data stored in volts can be multiplied by `1,000,000` to work on a microvolt scale. The factor is applied before both visualization and detection.

## Choose a workflow

| Single recording | Batch processing |
| --- | --- |
| Visualize the EEG and every active detector | Process several recordings without opening plots |
| Inspect raw algorithm values | Review settings once and apply them to every file |
| Manually add or remove suppression intervals | Save annotations automatically in the background |
| Save after reviewing the final annotation | Follow progress in a simple batch dialog |

## Review one recording

1. Select **Load single file…** and choose a compatible `.mat` file.
2. Wait while the loading and prediction popup is visible. It closes automatically when processing finishes.
3. Review the EEG in the upper plot and the full-recording overview below it.
4. Inspect, edit, or navigate through the result.
5. Select **Save annotation** when the annotation is ready.

Only selected algorithms are calculated and displayed. Their masks are combined with a logical OR to create the automatic annotation.

### Navigate and inspect

- Use **Previous**, **Next**, or the left and right arrow keys to move through the EEG.
- Click the lower overview to center the EEG view at that location.
- In **Navigation mode**, hover over the upper EEG plot to see the active algorithms' raw values for the nearest one-second window.
- The hover readout uses values already calculated during prediction and does not run the algorithms again.

### Correct the annotation

- Select **Select suppression**, then click two boundaries in the upper EEG plot to add that interval.
- Select **Deselect suppression**, then click two boundaries to remove that interval.
- Select **Navigation mode** or press Escape to stop editing.

Manual additions remain when predictions are run again. Manual deselections are cleared when a new prediction replaces the previous result.

## Adapt the settings

Open **Settings** to reveal a compact panel beside the recording. Signal and algorithm settings have separate **Restore defaults** and **Done** buttons.

### Signal settings

| Setting | Effect |
| --- | --- |
| EEG window | Changes how many seconds are visible in the upper plot |
| Y limits | Fixes the vertical display range |
| EEG scaling factor | Multiplies the EEG before visualization and every detector |

Changing the display window or Y limits does not alter predictions. Changing the scaling factor starts a new prediction because the algorithms receive different values.

### Algorithm settings

| Setting | Effect |
| --- | --- |
| Minimum duration | Removes shorter automatic detections; manual edits are unaffected |
| Use | Selects which algorithms are calculated, displayed, and combined |
| Threshold | Overrides the calibrated threshold for that algorithm |

The threshold rules are:

| Algorithm | Suppression decision |
| --- | --- |
| CNN | score `>=` threshold |
| Coastline | value `<=` threshold |
| Westover | value `<` threshold |
| OpenIBIS | value `<=` threshold |

Select **Done** to apply algorithm changes. A new prediction starts automatically when processing settings changed. **Restore algorithm defaults** selects all algorithms, restores the one-second minimum duration, and reloads the thresholds from the bundled model.

Settings remain unchanged when another recording is loaded during the same application session. Closing the application starts a new session with the defaults.

## Process a batch

Batch processing applies one reviewed configuration to every selected recording and does not open individual visualizations.

1. Select **Load and run batch…**.
2. Select all recordings to process.
3. Review the Signal settings and select **Done**.
4. Review the selected algorithms and thresholds, then select **Done** to start.
5. Follow progress in the batch dialog. **Cancel** stops after the current recording finishes.

For each processed recording, the application saves `Annotation_<recording>.csv` beside its source file. It also writes one `Batch_settings.txt` beside the first selected recording containing the scale factor, minimum duration, and selected algorithm thresholds.

## Output files

Annotation files contain:

| Column | Meaning |
| --- | --- |
| `Start` | Zero-based, inclusive sample index |
| `Stop` | Zero-based, exclusive sample index |
| `Label` | `Suppression` |

Example:

```csv
Start,Stop,Label
250,750,Suppression
```

Intervals are sorted and consolidated. A recording with no detected or manually selected suppression produces a header-only CSV. Divide a sample index by `250` to convert it to seconds.

Existing `Annotation_<recording>.csv` and `Batch_settings.txt` files with the same names are replaced.

## Run the application

### Packaged application

The packaged application includes Python, the model, and all required libraries.

- **Windows:** extract the complete archive and open `SuppressionDetection.exe`. Keep the `_internal` folder beside the executable.
- **macOS:** extract the archive and open `SuppressionDetection.app`.

Development builds may trigger Windows SmartScreen or macOS Gatekeeper because they are unsigned.


## Troubleshooting

- **The file does not load:** confirm that the top-level variable is named `EEG` and contains `EEG.data` and `EEG.srate`.
- **Sampling-rate mismatch:** recordings must use `250` Hz.
- **The recording is rejected as too short:** provide at least one second of EEG.
- **The trace is too small, too large, or outside the plot:** adjust the scaling factor and Y limits under Signal settings.
- **The results need a different sensitivity:** review the active algorithms, minimum duration, and threshold values, or restore the calibrated defaults.

All EEG processing is performed locally. The application does not upload recordings.
