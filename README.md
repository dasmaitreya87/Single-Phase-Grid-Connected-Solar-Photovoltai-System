# Single-Phase Grid-Connected Solar PV System

A MATLAB/Simulink model of a **single-phase solar photovoltaic (PV) system connected to the utility grid**. The model brings together a PV plant, maximum power point tracking (MPPT), grid synchronisation, current control, an output filter, and protection logic to study how solar power can be delivered to the grid.

The project follows the architecture shown in the system diagram and is based on the MathWorks example: [Single-Phase Grid-Connected Solar Photovoltaic System](https://in.mathworks.com/help/simscape-electrical/ug/single-phase-grid-connected-in-pv-system.html).

> **Note:** Simulation results depend on the model file, component parameters, irradiance, temperature, grid conditions, and solver settings. Use the parameter values in your local model as the source of truth.

## Objectives

- Model the power flow from a PV array to a single-phase utility grid.
- Track the PV array's maximum power point as operating conditions change.
- Synchronise the control system with the grid voltage using a phase-locked loop (PLL).
- Regulate the current injected into the grid.
- Monitor PV voltage/current, grid current, and power during simulation.
- Study protection and grid-disconnection behaviour.
- Optionally linearise the control system for open-loop frequency-response analysis.

## System Architecture

The main signal and power-flow path is:

```text
Irradiance
    │
    ▼
PV Solar Plant ── DC-link capacitor (C1) ── Power Circuit ── L filter ── Circuit Breaker ── Grid
    │                                           ▲                 │              ▲
    │                                           │                 │              │
    ├── Vpv, Ipv ──► MPPT ──► reference ──► Current Controller ──┘              │
    │                                                                         │
    └────────────────────► PLL and Protection ── angle / frequency / TripSig ──┘
```

The diagram is a simplified conceptual view; follow the actual signal connections in the Simulink model for implementation details.

### Main Subsystems

| Subsystem | Purpose |
|---|---|
| **Solar Plant** | Represents the PV array and converts the irradiance input into DC electrical power. PV voltage (`Vpv`) and current (`Ipv`) are measured for control and monitoring. |
| **DC-link capacitor (`C1`)** | Helps smooth the DC-side voltage and buffers short-term differences between PV power and power delivered to the grid. |
| **MPPT** | Determines an operating/reference signal that allows the PV plant to operate near its maximum-power point. The MathWorks example supports Perturbation and Observation (P&O) and Incremental Conductance methods. |
| **Power Circuit** | Represents the power-conversion stage connecting the DC-side PV plant to the grid-side circuit. The MathWorks example provides average, continuous, and discrete inverter options. |
| **Output inductor (`L`)** | Filters switching-related current ripple and helps shape the current injected into the grid. |
| **PLL and Protection** | Uses measured grid/PV signals to estimate grid angle and frequency, and generates a trip signal for protection. |
| **Current Controller** | Uses the reference current and synchronisation signals to control the grid-side current. Its output is passed toward the converter's modulation/control input. |
| **Circuit breaker** | Connects or disconnects the PV system from the grid in response to the protection logic. In the supplied diagram, green indicates closed and red indicates open. |
| **Scopes and measurement blocks** | Display signals such as supply current, PV voltage, PV current, and power. Linearization I/O points can be used for control-system analysis. |

The referenced MathWorks model is a **boostless** grid-connected PV configuration, meaning it does not use an intermediate DC–DC converter between the PV plant and inverter. The example is designed to inject solar power at unity power factor under its intended operating conditions.

## Signals to Monitor

| Signal | Meaning / Use |
|---|---|
| `Vpv` | PV array terminal voltage; useful for observing MPPT behaviour. |
| `Ipv` | PV array output current. |
| `Power` | Power signal used by the MPPT/control and monitoring paths. Check the model's units and scaling before interpreting plotted values. |
| `Vac` | Measured AC/grid voltage used by the synchronisation subsystem. |
| `Iac` | Grid-side current; inspect its waveform and phase relationship with the grid voltage. |
| `angle` | Estimated grid phase angle from the PLL. |
| `Angfreq` | Estimated grid angular frequency. |
| `Vsd` | Internal signal passed between the synchronisation, MPPT, and controller blocks in the shown model. Its exact definition should be confirmed from the subsystem implementation. |
| `TripSig` | Protection/trip signal used to command grid disconnection. |
| `m` | Controller/modulation-related output shown after the linearization I/O block. Confirm its interpretation in the model before using it as a physical measurement. |

## Requirements

- MATLAB with **Simulink**.
- **Simscape Electrical** for the electrical plant and power-system components.
- **Simulink Control Design** may be required for the model's linearization and frequency-response workflow.
- A compatible version of the referenced model and its parameterization script.

Block availability and solver behaviour can differ between MATLAB releases. Check the MathWorks documentation for the release installed on your machine.

## Getting Started

1. Open the [MathWorks example page](https://in.mathworks.com/help/simscape-electrical/ug/single-phase-grid-connected-in-pv-system.html) and open or download the model using the options provided there.
2. If using the documented example, inspect its parameterization script from the MATLAB Command Window:

   ```matlab
   edit('SinglePhaseGridSolarPVData')
   ```

3. Review the PV module data, array sizing, grid settings, component values, and solver configuration before running the model.
4. Open the Simulink model and run the simulation.
5. Inspect the scopes for `Vpv`, `Ipv`, grid current, and power. Compare the current waveform with the grid voltage and check whether the breaker/protection logic behaves as expected.

If this repository contains a local `.slx` model and supporting `.m` files, open the supplied model and follow its accompanying setup instructions. The exact local filename is intentionally not assumed here.

## Configuration Options in the MathWorks Example

The linked example documents the following options.

### MPPT Method

The example supports two MPPT methods through the workspace variant variable `MPPT`:

- `MPPT = 0`: Perturbation and Observation (P&O).
- `MPPT = 1`: Incremental Conductance.

### Inverter Model

The example provides average, continuous, and discrete inverter variants. To select the average inverter model, the documentation specifies:

```matlab
powerCircuit = 0;
```

Use the option appropriate for your analysis: an average model is generally useful for control-oriented studies, while more detailed switching representations can capture additional waveform behaviour at higher simulation cost.

### Open-Loop Linearization

For the documented open-loop Bode-plot workflow, the MathWorks example instructs users to set `closeLoop = 0` and use the average inverter model before linearizing. Consult the linked documentation and the local model's parameterization script for the full procedure.

## What to Check in a Simulation

Use the following as a qualitative validation checklist rather than assuming particular numerical results:

- **PV response:** `Vpv` and `Ipv` respond to irradiance and temperature changes.
- **MPPT response:** The PV operating point moves toward the maximum-power point after conditions change.
- **PLL response:** The estimated angle/frequency follows the grid signal and settles without sustained oscillation.
- **Current regulation:** Grid current follows the controller's reference and has the intended phase relationship with grid voltage.
- **Power flow:** The measured power is consistent with the PV operating condition and the chosen signal scaling.
- **Protection:** A trip condition opens the circuit breaker according to the model's protection logic.
- **Numerical stability:** Solver warnings, discontinuities, and time-step settings are reviewed when detailed power-electronics switching models are used.

Do not claim a specific efficiency, harmonic-distortion value, settling time, or power rating unless it has been measured from the configured model and documented simulation results.

## Troubleshooting

- **Missing blocks or libraries:** Verify that Simulink and Simscape Electrical are installed and that the model release is compatible with your MATLAB release.
- **Undefined workspace variables:** Run the model's parameterization/setup workflow before starting the simulation.
- **Unexpected MPPT behaviour:** Check the selected MPPT variant, PV parameters, irradiance/temperature inputs, and the definitions/scaling of `Vpv`, `Ipv`, and power signals.
- **Breaker remains open:** Inspect `TripSig`, protection thresholds, initial conditions, and the breaker control input.
- **Slow simulation:** Consider whether a different documented inverter variant is suitable for the study and review solver settings before changing physical parameters.
- **Linearization errors:** Confirm the required toolbox is available, the linearization I/O points are correctly placed, and the documented open-loop settings are active.

## Reference

- MathWorks, **Single-Phase Grid-Connected Solar Photovoltaic System**: https://in.mathworks.com/help/simscape-electrical/ug/single-phase-grid-connected-in-pv-system.html

The reference page contains the model description, subsystem details, MPPT options, inverter variants, and the open-loop linearization workflow. Review its licensing and redistribution terms before committing MathWorks-provided model files or example code to a public repository.

## License

No project-specific license has been declared in this README. Add a `LICENSE` file if you intend to distribute your own code. If this repository includes or modifies MathWorks example files, make sure your use and redistribution comply with the applicable MathWorks terms.
