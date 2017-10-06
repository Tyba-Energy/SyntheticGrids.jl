# SyntheticGrids v0.1
August - 2017

```
IMPORTANT: Currently, this package requires Julia to be ran with '--inline=no' due to a known issue:
https://github.com/JuliaLang/julia/issues/22582
```

Power grid research requires testing in realistic, large-scale,  electric  networks.   However,  in  light  of  security threats,  most  information  on  the  actual  power  grids  is considered  sensitive  and  therefore  not  available  to  the general  public.   So  far,  most  power  transmission  studies have been carried using a few publicly available test grids.  Still,  these test grids are too small to capture the  complexity  of  real  grids.   With  this  in  mind,  there has recently been a strong concentrated effort in developing methodologies for building realistic synthetic grids, based only on publicly available information.  These synthetic grids are supposed to be based on some real example  and  to  present  analogous  properties  —  such  as geographic  load/generation  distribution,  total  load  and generator types — while not actually presenting potentially sensitive information about the real grid.

This module provides an open source suite for generating synthetic grids based on real data openly available to the public. Power grids constructed via the SyntheticGrids module can be easily exported to [pandapower](https://pandapower.readthedocs.io/en/v1.2.2/index.html) for running optimum power flow calculations. Currently, information is limited to the USA region, but the framework can be readily applied to any other region, provided there are data sources available. We leverage the works published by Overbye's group and Soltan and Zussman, providing a direct implementation of their methods. For more details on the approaches adopted, please see [Model](Model.md).

REFERENCES:
- Birchfield, Adam B., et al. "Grid Structural Characteristics as Validation Criteria for Synthetic Networks." IEEE Transactions on Power Systems (2016).
- Gegner, Kathleen M., et al. "A methodology for the creation of geographically realistic synthetic power flow models." Power and Energy Conference at Illinois (PECI), 2016 IEEE. IEEE, 2016.
- Birchfield, Adam B., et al. "Statistical considerations in the creation of realistic synthetic power grids for geomagnetic disturbance studies." IEEE Transactions on Power Systems 32.2 (2017): 1502-1510.
- Soltan, Saleh, and Gil Zussman. "Generation of synthetic spatially embedded power grid networks." Power and Energy Society General Meeting (PESGM), 2016. IEEE, 2016.

## Detailed documentation
* [Types](Types.md)
* [Public Functions](Functions.md)
* [Private Functions](Private.md)
* [Model](Model.md)

## Current functionalities

- Implements basic types.
- Builds networks from real-world data.
- Builds connection topology from nodes.
- Builds transmission lines.
- Clusters nodes into substations.
- Provides basic checks for the graph structure.
- Interfaces with [pandapower](https://pandapower.readthedocs.io/en/v1.2.2/index.html) for exporting networks.

## Basic Usage

```julia
grid = Grid()
```

Create new (empty) grid

```julia
place_loads_from_zips!(grid, latitude limits, longitude limits)
```

Build load buses from zipcodes

```julia
place_gens_from_data!(grid, latitude limits, longitude limits)
```

Build generation buses from data
(buses may also be placed manually)

```julia
connect!(grid)
```

Generate node connections

```julia
cluster!(grid, nloads, nboth, ngens)
```

Build substations by clustering nodes (optional)

```julia
create_lines!(grid)
```

Build transmission lines from connection topology

## Exporting network to pandapower

This module makes use of `PyCall` in order to interface with `pandapower`. Since the reference charts in `pandapower` do not contain transmission lines and transformer parameters for several voltages, currently, this module places all loads at 110kV and all generation at 380kV in order to have the proper parameters. Transformers are automatically placed every time two connected buses operate at different voltages. In order to obtain a `pandapower` object from a SynGrid instance, simply run the command:

```julia
to_pandapower(grid)
```

This returns a `PyObject`. A path can be optionally passed as well in order to obtain a file saved in the native `pandapower` format. These files can later be imported via the `loadppgrid` command.

- Note: the exported grid is zero-indexed, as it is being passed to Python routines.

## Current issues

### Limitations:
- No type has been implemented for transformers.
- Connections between buses ignore differences in voltages.
- Transmission line properties are still oversimplified.
- There are missing quantities for AC OPF runs (DC should be fine).
- These limitations only pertain the SynGrids objects, not the grids exported to pandapower.

### Data:
- Generator data has inconsistencies.
  - 2015 survey data has 1337 power plants without any reported generator.
  - 2014 survey data has 1403 power plants without any reported generator.
  - There are unsited plants and plants with clearly wrong coordinates (the obvious ones
    were manually corrected).