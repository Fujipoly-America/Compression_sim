# Rate-Dependent Thermal Gap Pad and PCB Stack Calculator

An interactive Jupyter Notebook for estimating how an imposed heat-sink motion is divided between compression of a thermal gap pad and elastic deflection of a printed circuit board (PCB).

The calculator uses measured pad force–compression curves collected at multiple compression speeds to create a rate-dependent pad surface. It combines that surface with a rate-independent PCB force–deflection curve and solves the pad/PCB stack one time step at a time.

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/Fujipoly-America/Compression_sim/blob/main/Rate_Dependent_PAD_PCB_Stack_Calculator.ipynb)

## What the notebook calculates

For a specified stack displacement and heat-sink speed, the notebook estimates:

- Pad compression
- Actual pad compression rate
- PCB deflection
- Common force carried by the pad and PCB
- Total stack displacement
- Remaining compressed pad thickness
- The portion of the stack motion absorbed by each component

It also creates:

- A 3D pad surface of compression, compression speed, and force
- Component and combined-response plots
- Displacement-sharing plots
- An optional animated view of the heat sink, pad, and PCB
- An optional GIF export

## Quick start

1. Click **Open in Colab** above.
2. Run the notebook from top to bottom using **Runtime → Run all**.
3. Leave `USE_EXAMPLE_DATA = True` for the first run.
4. Review the example pad surface, results, plots, and animation.
5. Set `USE_EXAMPLE_DATA = False`, upload your measured CSV files, and run the notebook again.

No local Python installation is required when using Google Colab.

## Run from an IDE or terminal

Customers who prefer VS Code, PyCharm, Spyder, or another Python IDE can use the standalone script:

```text
Rate_Dependent_PAD_PCB_Stack_Calculator.py
```

Edit the **CUSTOMER INPUTS** section at the top of the file, place the measured CSV files in the same folder (or enter their paths), and run the script. From a terminal:

```bash
python Rate_Dependent_PAD_PCB_Stack_Calculator.py
```

The Python script uses regular Matplotlib windows and does not require Jupyter or IPython.

## Customer inputs

The main settings are in the customer-input cell near the beginning of the notebook. They include:

```python
USE_EXAMPLE_DATA = True

PAD_RATE_DATA_FILE = Path("pad_rate_curves.csv")
PCB_DATA_FILE = Path("pcb_curve.csv")

PAD_ORIGINAL_THICKNESS_MM = 1.0
TARGET_TOTAL_DISPLACEMENT_MM = 0.70
STACK_SPEED_MM_MIN = 5.0
TIME_STEP_SECONDS = 0.10

RUN_ANIMATION = False
SAVE_GIF = False
```

Set `USE_EXAMPLE_DATA = False` to use measured data. In Colab, upload both CSV files to the notebook session before running the data-loading cell. Colab session files are temporary and must be uploaded again after starting a new runtime.

## Required pad data

The pad CSV must contain a complete force–compression curve for each measured compression speed. Use these exact column headings:

```csv
Rate_mm_min,Disp_mm,Force_N
1.0,0.00,0.00
1.0,0.10,2.10
1.0,0.20,5.20
5.0,0.00,0.00
5.0,0.10,2.80
5.0,0.20,6.70
```

| Column | Meaning | Unit |
|---|---|---|
| `Rate_mm_min` | Measured pad compression speed | mm/min |
| `Disp_mm` | Pad compression | mm |
| `Force_N` | Measured pad force | N |

Provide at least two displacement points for every rate. More measured points and more test speeds generally produce a better representation of the pad response.

The notebook uses the measured curves directly. It does **not** apply a force multiplier to convert one reference curve into the other speeds.

## Required PCB data

The PCB CSV must contain these exact column headings:

```csv
Disp_mm,Force_N
0.00,0.00
0.05,2.50
0.10,5.00
0.20,10.00
```

| Column | Meaning | Unit |
|---|---|---|
| `Disp_mm` | PCB deflection | mm |
| `Force_N` | PCB reaction force | N |

The PCB response is treated as elastic and rate-independent. The input curve may be linear or nonlinear, although a linear curve is commonly used for this model.

## How the model works

The pad and PCB are treated as mechanical elements in series. Therefore, they carry the same force:

$$
F_{pad}=F_{PCB}
$$

Their displacements add to the imposed stack displacement:

$$
x_{stack}=x_{pad}+x_{PCB}
$$

Their instantaneous displacement rates also add to the imposed stack speed:

$$
v_{stack}=v_{pad}+v_{PCB}
$$

The pad force depends on both its current compression and its actual compression rate:

$$
F_{pad}=F(x_{pad},v_{pad})
$$

At every time step, the notebook:

1. Advances the total stack displacement using the specified heat-sink speed.
2. Estimates a new pad compression.
3. Calculates PCB deflection from the remaining stack displacement.
4. Calculates the pad compression rate from the change in pad compression over the time step.
5. Evaluates pad force from the measured rate-dependent surface.
6. Evaluates PCB force from the PCB curve.
7. Adjusts pad compression until the pad and PCB forces match.

This is why the pad compression speed can be lower than the heat-sink speed: some of the imposed motion is used to deflect the PCB.

## Interpolation and measured range

- Shape-preserving PCHIP interpolation is used along each measured force–compression curve.
- Values between measured pad speeds are interpolated in logarithmic speed space.
- Imported curves remain independent measured curves; they are not replaced by a fitted speed multiplier.
- The calculation stays within the overlapping measured data range rather than extrapolating beyond the available test data.
- A result at a range boundary should be reviewed because it may indicate that additional test coverage is needed.

Use pad data collected with representative specimen geometry, contact area, temperature, test method, and loading direction.

## Main data frames

The notebook creates the following principal pandas DataFrames:

| DataFrame | Purpose |
|---|---|
| `pad_rate_data` | Imported or example pad measurements at multiple rates |
| `pcb_curve` | Imported or example PCB force–deflection data |
| `stack` | Complete time-step solution for the combined stack |
| `results` | Customer-facing result table |

The 3D pad surface is stored in NumPy arrays rather than another DataFrame. `equilibrium` is the final selected row from the solved stack history. Important result columns include `Force_N`, `Pad_Disp_mm`, `PCB_Disp_mm`, `Total_Disp_mm`, and `Pad_Rate_mm_min`.

## Example data

Example mode contains separate illustrative pad curves at several compression speeds and an illustrative PCB curve. These values are provided only to demonstrate the workflow. They do not represent a specific Fujipoly product or customer assembly.

## Assumptions and limitations

- The pad and PCB are modeled as components in series.
- The PCB is elastic and rate-independent.
- The pad loading response is rate-dependent and is defined by the imported measurements.
- Input data should represent the actual pad area, PCB support condition, and loading geometry as closely as practical.
- The model describes the initial loading process only.
- Stress relaxation at a fixed gap is not modeled.
- Creep, unloading, reloading, hysteresis, compression set, permanent deformation, thermal cycling, and aging are not modeled.
- Local PCB strain and component-level stress are not calculated.
- Interpolation cannot substitute for missing test data far outside the measured conditions.

## Engineering use

This notebook is intended for education, preliminary analysis, and design comparison. It does not replace mechanical validation of the actual assembly. Confirm material selection, pad compression, PCB strain, component loading, and long-term behavior through appropriate testing and engineering review.

## Requirements for local use

Install the following packages to run the notebook or Python script locally:

```bash
pip install numpy pandas scipy matplotlib pillow
```

Install `jupyter` as well only when using the notebook locally.

## Suggested repository structure

```text
Compression_sim/
├── README.md
├── Rate_Dependent_PAD_PCB_Stack_Calculator.ipynb
├── Rate_Dependent_PAD_PCB_Stack_Calculator.py
└── sample_data/
    ├── pad_rate_curves.csv
    └── pcb_curve.csv
```

The `sample_data` directory is optional but recommended because it shows customers the required file formats.

## Support

For questions about the calculator, input data, or Fujipoly thermal interface materials, contact your Fujipoly America representative.
