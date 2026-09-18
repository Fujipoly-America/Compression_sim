# Thermal Gap Filler and PCB Deflection Calculator

An interactive Jupyter Notebook for estimating the mechanical interaction between a thermal gap-filler pad and a printed circuit board (PCB). The calculator combines measured pad and PCB force–displacement data to estimate the initial equilibrium force, pad compression, compressed pad thickness, and PCB deflection.

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/Fujipoly-America/Compression_sim/blob/main/Compression_Simulation_with_Plots_and_Animation.ipynb)

## What the notebook does

The notebook:

- Loads built-in example data or user-provided CSV files.
- Interpolates the pad and PCB responses using shape-preserving PCHIP interpolation.
- Adds pad compression and PCB deflection at a common force because the components act mechanically in series.
- Solves for the equilibrium corresponding to a specified total assembly displacement.
- Reports force, pad compression, compressed pad thickness, and PCB deflection.
- Produces four plots describing the individual and combined responses.
- Creates an optional animation of the pad, PCB, and heat-sink interaction.
- Can export the animation as a GIF.

## Quick start

1. Select **Open in Colab** above.
2. Run the notebook from top to bottom using **Runtime → Run all**.
3. Begin with `USE_EXAMPLE_DATA = True` to confirm that the notebook runs correctly.
4. Enter the desired total assembly displacement and original pad thickness in the customer-input cell.
5. Review the calculated results, plots, and animation.

No local Python installation is required when using Google Colab.

## Using your own data

Change the following setting:

```python
USE_EXAMPLE_DATA = False
```

Then provide the filenames for the pad and PCB data:

```python
PAD_DATA_FILE = "pad_curve.csv"
PCB_DATA_FILE = "pcb_curve.csv"
```

Upload both CSV files to the notebook session before running the data-loading cell. When using Colab, files uploaded to the session are temporary and must be uploaded again when a new runtime is started.

## CSV format

Both CSV files must contain these exact column headings:

```csv
Disp_mm,Force_N
0.00,0.00
0.10,5.00
0.20,12.00
```

### Pad file

`Disp_mm` is the pad compression in millimeters at the corresponding measured force.

### PCB file

`Disp_mm` is the PCB deflection in millimeters at the corresponding measured force.

For both files:

- Use force in newtons and displacement in millimeters.
- Enter nonnegative numeric values.
- Include at least two valid data points.
- Cover the force range containing the expected equilibrium.
- Avoid duplicate force values.
- Do not use data outside the measured range for design decisions.

## Calculation method

At mechanical equilibrium, the pad and PCB carry the same force:

$$
F_{pad}=F_{PCB}=F
$$

Because the pad and PCB act in series, their displacements are added:

$$
x_{total}(F)=x_{pad}(F)+x_{PCB}(F)
$$

The notebook finds the force where the combined displacement equals the specified assembly displacement:

$$
x_{pad}(F)+x_{PCB}(F)=x_{target}
$$

PCHIP interpolation is used between measured points to preserve the shape of each response without introducing excessive oscillation.

## Results and visualizations

The notebook reports:

- Equilibrium force
- Pad compression
- Compressed pad thickness
- PCB deflection
- Total imposed displacement

It also displays:

1. Measured pad and PCB component responses
2. Combined series response and selected equilibrium
3. Pad and PCB displacement at a common force
4. Each component's share of total displacement
5. An optional animated stack and force–displacement plot

## Important input considerations

Gap-filler force–compression behavior is nonlinear and can depend on compression rate, temperature, specimen geometry, contact area, and test method. Use pad data measured under conditions representative of the intended assembly.

PCB response depends on board dimensions, thickness, material construction, support locations, component placement, and loading area. Use measured or simulated PCB force–deflection data that represent the actual design as closely as practical.

## Assumptions and limitations

- The pad and PCB are treated as components acting mechanically in series.
- The pad curve represents the loading rate and conditions used during measurement.
- Interpolation is limited to the overlapping measured force range.
- The PCB input can be linear or nonlinear.
- The example data are illustrative and do not represent a specific Fujipoly product or customer assembly.
- The model calculates initial mechanical equilibrium only.
- Stress relaxation, creep, compression set, permanent deformation, thermal cycling, and material aging are not modeled.
- Local board strain and component-level stress are not calculated.

## Engineering use

This notebook is intended for education, preliminary analysis, and design comparison. It does not replace mechanical validation of the actual assembly. Confirm material selection, compression, PCB strain, and component loading through appropriate testing and engineering review.

## Requirements for local use

If running the notebook locally, install:

```text
numpy
pandas
scipy
matplotlib
pillow
jupyter
```

## Repository contents

```text
Compression_sim/
├── README.md
├── Compression_Simulation_with_Plots_and_Animation.ipynb
└── sample_data/
    ├── pad_curve.csv
    └── pcb_curve.csv
```

The `sample_data` directory is optional but recommended for demonstrating the required file format.

## Support

For questions about the calculator, input data, or Fujipoly thermal interface materials, contact your Fujipoly America representative.

