# Dutch-North-Sea-region-electricity-hydrogen-model

A Calliope-based, hourly electricity-hydrogen capacity expansion model for the Netherlands and North Sea region in 2050. It co-optimizes electricity generation, electrolysis, hydrogen and electricity storage, and transmission investments and operations, incorporating DC power flow, hydrogen linepacking, and alternative energy transition scenarios.

## Citation

**If you use or cite this model, please cite the accompanying manuscript:**

> Frowijn, L. S. F., Travaglini, R., Lukszo, Z., & Bruninx, K. *The role of electricity and hydrogen network representation in modeling green hydrogen supply chains: a Dutch case study*. Manuscript — **publication details TBD**.

The citation will be updated once the manuscript is published.

The underlying modeling framework should also be acknowledged as [Calliope](https://calliope.readthedocs.io/en/v0.7.0.dev7/).

## License

Please refer to the `LICENSE` file for the terms under which the model is distributed.

## Model overview

The model uses [Calliope](https://calliope.readthedocs.io/en/v0.7.0.dev7/), an open-source energy system modeling framework, to minimise annual system costs by jointly optimising investment and hourly operation of electricity and hydrogen infrastructure. The geographical scope includes the Netherlands, Belgium, Germany, Denmark, the United Kingdom, and their relevant offshore North Sea regions. The Netherlands is represented at provincial resolution, while neighbouring countries are represented at a more aggregated level.

The model covers one year (2050) at hourly resolution (8,760 timesteps). It investigates the allocation of onshore and offshore electrolysis, electricity and hydrogen transmission, storage, and cross-border energy exchange. Three energy transition scenarios are considered: **Base Ambition (BA)**, **Hydrogen-Oriented (HO)**, and **Electrification (EL)**.

## Requirements and installation

The model was developed using **Calliope 0.7.0.dev7** and **Gurobi** as its optimisation solver. Install the specified Calliope version rather than assuming compatibility with other Calliope releases. A working Gurobi installation and valid licence are required to solve the model.

1. Clone or download this repository.
2. Create and activate a Python environment.
3. Install the required Calliope version:

   ```bash
   python -m pip install "calliope==0.7.0.dev7"
   ```

4. Ensure that Gurobi and its Python interface are available in the environment, and that your Gurobi licence is configured. See the [Gurobi documentation](https://www.gurobi.com/documentation/).
5. Check the Calliope installation:

   ```bash
   calliope --version
   ```

For framework-specific installation guidance, consult the [Calliope 0.7.0.dev7 documentation](https://calliope.readthedocs.io/en/v0.7.0.dev7/).

## Model structure

Each model configuration is defined through a `model.yaml` file and associated YAML and CSV files. The principal components are:

| File or directory | Description |
|---|---|
| `model.yaml` | Main Calliope configuration, including imports, model settings, scenario definitions, input data tables, and solver settings. |
| `model_definition/` | Technology, node, link, and other model data definitions. |
| `overrides/` | Scenario-specific assumptions and policy constraints. |
| `custom_math/` | Additional mathematical formulations used to extend or override Calliope's standard model mathematics. |
| `data_tables/` | Hourly demand and renewable availability data and electricity network parameters. |

Paths are resolved relative to the relevant model configuration. Different runs may use different custom mathematical formulations.

## Scenario-specific overrides

The `overrides/` directory contains scenario- and run-specific changes to the shared model definition. These include policy assumptions, technology investment options, and capacity constraints. In particular, `policy.yaml` specifies restrictions relevant to the selected energy transition scenario or network-representation run.

The `model.yaml` file determines how these changes are applied. Files listed under `import` are loaded as part of the model definition, while the `scenarios` section defines named combinations of overrides that can be selected at runtime. For example, in configurations defining `scenario: [policy]`, `--scenario scenario` selects that named scenario and its associated `policy` override.

**Users should consult the `model.yaml` and `overrides/policy.yaml` files of each run** to identify the applicable assumptions. These files determine the model configuration in addition to any run-specific custom mathematical formulations.

## Running the model

Run each configuration from the directory containing its `model.yaml` file. The configurations define a scenario named `scenario`, selected through `--scenario scenario`.
Each model configuration is self-contained, including its own model definitions, mathematical formulations, scenario-specific overrides, and input data. This allows individual configurations to be executed and reproduced independently without modifying file paths.

### Base Ambition (BA)

The `Base_Ambition/` directory contains four model configurations that isolate the effects of Dutch onshore electricity network representation (DC power flow) and hydrogen pipeline flexibility (linepacking):

| Directory | Configuration | Network representation |
|---|---|---|
| `0.reference` | Reference | Includes DC power flow and hydrogen linepacking. |
| `1.no_both` | No DC power flow or linepacking | Excludes both formulations. |
| `2.no_linepack` | No linepacking | Includes DC power flow; excludes linepacking. |
| `3.no_DCPF` | No DC power flow | Includes linepacking; excludes DC power flow. |

For example, to run the **reference** configuration from the repository root:

```bash
cd "Base_Ambition/0.reference"
mkdir -p results
calliope run model.yaml --scenario scenario --save_netcdf results/results.nc
```

For the other BA configurations, navigate to the corresponding directory (`1.no_both`, `2.no_linepack`, or `3.no_DCPF`) and run the same Calliope command. If using Windows PowerShell, use `mkdir results` instead of `mkdir -p results` when creating the results directory.

### Hydrogen-Oriented (HO) and Electrification (EL)

The Hydrogen-Oriented and Electrification scenarios examine alternative electricity and hydrogen demand pathways and infrastructure investment assumptions. Both include DC power flow and hydrogen linepacking.

From the directory containing the relevant scenario's `model.yaml` file, run:

```bash
mkdir -p results
calliope run model.yaml --scenario scenario --save_netcdf results/results.nc
```

The `results/` directory must exist before saving the output. The NetCDF filename may be changed to distinguish runs.

**Computational requirements:** Each run optimises a full year with 8,760 hourly timesteps and can require substantial memory and computation time. Users can adapt solver options to their computing environment. Solver settings may be adjusted according to the user's computational environment. Results may differ slightly depending on solver versions, numerical tolerances, and computational settings.

## Input data

The `data_tables/` directory contains the following input files:

| File | Description |
|---|---|
| `electricity_demand.csv` | Hourly electricity demand by model node. |
| `hydrogen_demand.csv` | Hourly hydrogen demand by model node. |
| `solar_pv.csv` | Hourly solar photovoltaic availability profiles. |
| `wind_onshore.csv` | Hourly onshore wind availability profiles. |
| `wind_offshore.csv` | Hourly offshore wind availability profiles. |
| `ptdf.csv` | Power Transfer Distribution Factor (PTDF) coefficients for the Dutch onshore HVAC network. |
| `ptdf_link_cap.csv` | Transmission capacity limits for the links represented in the PTDF formulation. |
| `ptdf_link_lookup.csv` | Supporting lookup information connecting PTDF link identifiers to the underlying electricity network connections and parameters. |

### Data sources and processing

Dutch scenario assumptions for 2050, including provincial electricity and hydrogen demand and renewable generation capacity assumptions, are based on **Netbeheer Nederland, *Netbeheer Nederland Scenario’s Editie 2025***. For Belgium, Germany, Denmark, and the United Kingdom, demand and renewable capacity projections are based on the **ENTSO-E TYNDP 2026 National Trends+ scenario**, as described in the accompanying manuscript.



### Dutch onshore electricity network (PTDF)

The existing Dutch onshore high-voltage alternating-current (HVAC) network is represented using a linear DC power flow approximation based on PTDFs. In the accompanying study, the network is derived from the PyPSA-Eur transmission dataset and reduced from 51 nodes to 12 provincial nodes using the ElectricityNetworkReduction tool from TulipaEnergyModel.

The PTDF inputs have distinct functions:

- **`ptdf.csv`** maps nodal net electricity injections to transmission line flows through PTDF coefficients.
- **`ptdf_link_cap.csv`** provides line capacity limits used to constrain the calculated flows.
- **`ptdf_link_lookup.csv`** documents the correspondence between PTDF link identifiers and the underlying network. It is supporting reference information and is not directly loaded in the `data_tables` section of the provided `model.yaml`.

PTDF coefficients and line limits are imported into applicable Calliope configurations through the data table definitions in `model.yaml`. The PTDF formulation applies to the existing Dutch onshore HVAC network; additional HVDC transmission investments are represented separately as controllable links. The PTDF formulation is not used in configurations that exclude DC power flow.

## Custom mathematical formulations

The model is built on the standard mathematical formulations of [Calliope](https://calliope.readthedocs.io/en/v0.7.0.dev7/). Additional formulations in the `custom_math/` directory extend or override Calliope's standard equations and constraints, depending on the configuration being run.

| File | Function |
|---|---|
| `linepacking.yaml` | Represents hydrogen linepacking as storage flexibility, with usable storage capacity linked to hydrogen pipeline transmission capacity. |
| `nl_electrolyzer.yaml` | Specifies scenario-dependent constraints on total installed electrolyser capacity in the Netherlands. |
| `nl_vre_capacity.yaml` | Specifies Dutch variable renewable electricity capacity constraints for solar PV, onshore wind, and offshore wind. |
| `ptdf.yaml` | Implements the PTDF-based DC power flow formulation and transmission constraints for the Dutch onshore HVAC network. |

These files are referenced through `math_paths` and activated through `extra_math` in the relevant `model.yaml`. **The mathematical formulations included depend on the model run:** for example, Base Ambition configurations selectively omit `linepacking.yaml`, `ptdf.yaml`, or both. The custom formulations for Dutch electrolyser and renewable generation capacity constraints are applied where required by the scenario definitions.

For more information on Calliope's model mathematics and custom formulations, consult the [Calliope documentation](https://calliope.readthedocs.io/en/v0.7.0.dev7/).

## Model outputs

Model runs save results as NetCDF (`.nc`) files, which can be opened and analysed using Python and `xarray`. Model outputs include installed technology capacities, hourly electricity generation and hydrogen production, storage charging and discharging, electricity and hydrogen transmission flows, network investments, and system costs. Available outputs may differ depending on the model configuration and solver settings.


