# `SPARC-X-API`: A Python API for the SPARC-X DFT Code
[![Package](https://raw.githubusercontent.com/SPARC-X/SPARC-X-API/badges/badges/package.svg)](https://raw.githubusercontent.com/SPARC-X/SPARC-X-API/badges/badges/package.svg)
[![Coverage](https://raw.githubusercontent.com/SPARC-X/SPARC-X-API/badges/badges/coverage.svg)](https://raw.githubusercontent.com/SPARC-X/SPARC-X-API/badges/badges/coverage.svg)
[![Unit tests](https://github.com/SPARC-X/SPARC-X-API/actions/workflows/installation_test.yml/badge.svg)](https://github.com/SPARC-X/SPARC-X-API/actions/workflows/installation_test.yml)
[![JSON-API](https://raw.githubusercontent.com/SPARC-X/SPARC-X-API/badges/badges/api_version.svg)](https://raw.githubusercontent.com/SPARC-X/SPARC-X-API/badges/badges/api_version.svg)

# Table of Contents
- [Installation](#installation)
- [Setting Up Environment](#setting-up-the-environment)
- [Basic Usage](#basic-usage-of-the-python-api)
- [Advanced Usage: Socket Interface](#advanced-usage-sparc-x-api-as-a-socket-interface)
- [Troubleshooting](#troubleshooting)
- [Advanced Topics](#advanced-topics)
- [Support and Contribution](#how-to-contribute)

`SPARC-X-API` is an [ASE](https://wiki.fysik.dtu.dk/ase/)-compatible Python API for the density functional theory (DFT) code [SPARC](https://github.com/SPARC-X/SPARC). It offers:

1. ASE-compatible I/O format for SPARC files
2. A JSON Schema interfacing with SPARC's C-code for parameter validation and conversion
3. A comprehensive calculator interface for SPARC with socket-communication support.


[Fig. 1](#fig-1-schematic-drawing-for-the-architecture-of-the-sparc-x-api-package) provides an overlook of the components of `SPARC-X-API` and its relation with the SPARC C-code.

#### Fig. 1 Schematic drawing for the architecture of the `SPARC-X-API` package
![scheme-sparc-api-outlook](doc/img/scheme_api_architecture.png)



## Installation:

The Python API may be installed via either of the following approaches:

### 1. Via `anaconda` or `miniconda` (recommended)

Set up a [conda environment](https://conda.io/projects/conda/en/latest/user-guide/tasks/manage-environments.html#creating-an-environment-with-commands) and install the Python API,
which includes the pseudopotential files:

```bash
# Change 'sparc-env' to your desired name if needed
conda create -n sparc-env
conda activate sparc-env
conda install -c conda-forge sparc-x-api
```

On Linux platforms (x86_64, aarch64), you can also install the
precompiled `sparc` DFT binaries alongside the API:

```bash
conda install -c conda-forge sparc-x
conda activate sparc-env   # Re-activate to have the env variables effective
```

### 2. Manual installation from source with `pip`


```bash
python -m pip install git+https://github.com/SPARC-X/SPARC-X-API
```

Optionally, you can download the latest SPMS pseudopotentials and unpacks the pseudopotential files into `<python-lib-root>/site-packages/sparc/psp`:

```bash
python -m sparc.download_data
```


To utilize the API for drive SPARC calculations, please
following the [SPARC manual](https://github.com/SPARC-X/SPARC) for
compilation and installation of the SPARC DFT code itself.

### Post-installation check

We recommend the users to run a simple test after installation and setup:

```bash
python -m sparc.quicktest
```

A proper setup will display the following sections at the output's conclusion:

<img width="500" alt="image" src="https://github.com/alchem0x2A/SPARC-X-API/assets/6829706/95cb712e-4c77-4b14-8130-4961e3c50278">

For using the API to parse SPARC input and output files, it's
essential that the "Import" and "JSON API" tests are successful. For
run SPARC calculations, all tests must pass.

Please refer to the [Setting Up the
Environment](#setting-up-the-environment) or guidance on correctly
configuring the environment variables. If you run into further problems, consult our
[Trouble Shooting](doc/troubleshooting.md).

## Setting up the environment
`SPARC-X-API` is designed to automate the discovery of
pseudopotential files, the JSON API, and the SPARC binary. However,
you can exert fine-grained control over their setup:

### A) Pseudopotential files
Pseudopotential files (in `Abinit` psp8 format) are loaded in the following
order:

1) Via the `psp_dir` argument passed to the `sparc.SPARC` calculator.
2) Through the environment variables `$SPARC_PSP_PATH` or `$SPARC_PP_PATH` (this is the
 method employed by [`conda` installation](#1-via-anaconda-or-miniconda-recommended)).
3) By using `psp8` files bundled with the SPARC-X-API installation (see the
[manual installation](#2-manual-installation-from-source-with-pip)).

To specify a custom path for your psp8 files, set the `$SPARC_PSP_PATH` or `$SPARC_PP_PATH` variable as follows:
```bash
export SPARC_PSP_PATH="/path/to/your/psp8/directory"
```

To determine the default location of psp8 files (as per option 3), run the following code:
```bash
python -c "from sparc.common import psp_dir; print(psp_dir)"
```

### B) JSON schema
`SPARC-X-API` is engineered for compatibility with the SPARC
C-code. It achieves this by loading a JSON schema for
parameter validation and unit conversion. You can review the default
schema used by the API at sparc.sparc_json_api.default_json_api

```json
"FD_GRID": {
   "symbol": "FD_GRID",
   "label": "FD_GRID",
   "type": "integer array",
   "default": null,
   "unit": "No unit",
   "example": "FD_GRID: 26 26 30",
   "description": "#<Some description...>",
   "allow_bool_input": false,
   "category": "system"
  },
```

The schema file is generated from SPARC's LaTeX documentation.  In
upcoming releases of `SPARC-X-API`, we're aiming to provide users the
flexibility to use their own custom schema files. This would be
particularly useful for those who might be testing a development
branch of SPARC. By default, the JSON schema is packaged under
`sparc/sparc_json_api` directory. If you have another version of SPARC
source code, you can set the environment variable `$SPARC_DOC_PATH` to
the directory containing the LaTeX codes for the documentation, such
as `<SPARC-source-code-root>/doc/.LaTeX`. If you obtain `sparc-x` from
the conda method as mentioned above, By default, the JSON schema is
packaged under `sparc/sparc_json_api` directory. If you have another
version of SPARC source code, you can set the environment variable
`$SPARC_DOC_PATH` is automatically set to
`<conda-env-root>/share/doc/sparc/.LaTeX`. Setting up the environment
variable `$SPARC_DOC_PATH` helps loading the correct JSON schame that
is compatible with your SPARC binary code.

### C) SPARC Command Configuration

The command to execute SPARC calculations is determined based on the following priority:

1) The command argument provided directly to the `sparc.SPARC` calculator.
2) The environment variable `$ASE_SPARC_COMMAND`
3) If neither of the above is defined, `SPARC-X-API` looks for the SPARC binary under current `$PATH` and combine with the suitable `mpi` command prefix.

Example:

1. Using `mpirun` (e.g. on a single test machine)
```bash
export ASE_SPARC_COMMAND="mpirun -n 8 -mca orte_abort_on_non_zero_status 1 /path/to/sparc -name PREFIX"
```

2. Using `srun` (e.g. in HPC slurm job system)
```bash
export ASE_SPARC_COMMAND="srun -n 8 --kill-on-bad-exit /path/to/sparc -name PREFIX"
```

*Notes*:
1. The `-name PREFIX` part can be omitted the `label` property of the `sparc.SPARC` calculator is set (which is the default behavior). Any extra features of the SPARC code (e.g. GPU acceleration) should be specified in the command.

2. We recommend adding kill switches for your MPI commands like the examples above when running `sparc` to avoid unexpected behaviors with exit signals.


## Basic usage of the Python API
### 1. Read / write SPARC files

In contrast to many other DFT codes, where the ASE I/O formats refer
to a single file, `SPARC-X-API` operates on the whole calculation
directory, also known as a "SPARC bundle". This API integrates
seamlessly with ASE, allowing for the automatic detection of the SPARC
file format:

- Reading from a SPARC bundle

```python
import sparc
from ase.io import read, write
atoms = read("test.sparc", index=-1)
```
*Note*: To read multiple output files from the same directory, e.g., SPARC.aimd, SPARC.aimd\_01, pass the keyword argument `include_all_files=True` to `read()`

- Writing a minimal SPARC bundle from atoms

```python
import sparc
from ase.io import read, write
from ase.build import Bulk
atoms = Bulk("Al") * [4, 4, 4]
atoms.write("test.sparc")
```

For a deeper dive into the bundle I/O format, see [Advanced Topics](doc/advanced_topics.md).

### 2. JSON Schema for SPARC calculator

A recurring challenge of Python interfaces to DFT codes it the
inconsistencies between low-level codes (Fortran/C/C++) and outdated
upper-level APIs regarding parameter sets and default values. To
address this issue, `SPARC-X-API` handles DFT parameters through a
JSON schema translated from SPARC's LaTeX documentation.  Each release
of `SPARC-X-API` is linked with a specific version of the SPARC
source code, ensuring compatibility and consistency with the default
parameter set. The main driver of this feature is the
`sparc.api.SparcAPI` class.

If you've obtained the full SPARC [source
code](https://github.com/SPARC-X/SPARC), you can generate a copy of
the schema by the following code:
```bash
python -m sparc.docparser <sparc-source-code-root>/doc/.LaTeX
```
which produces a `parameters.json` file.

To learn more about the JSON schema design, please refer to [Advanced
Topics](doc/advanced_topics.md).


### 3. Calculator interface (File-IO mode)

`SPARC-X-API` offers a calculator interface based on file I/O that aligns with many
other ASE calculators.  If you've worked with ASE modules like `Vasp`,
`QuantumEspresso`, or `GPAW`, you'll find this package intuitive,
as shown in the following examples:

1. Single point calculation
```python
from sparc.calculator import SPARC
from ase.build import molecule
atoms = molecule("H2", cell=(10, 10, 10), pbc=True, directory="run_sp")
atoms.calc = SPARC(h=0.25)
atoms.get_potential_energy()
atoms.get_forces()
```

This example sets up a calculation for H2 atoms in a 10 Å x 10 Å x 10
Å PBC cell with default parameters (PBE exchange correlation
functional, and a grid spacing (`h`) of 0.25 Å). Note by calling
`atoms.get_forces`, the calculator will automatically sets the flags
for printing the forces.

2. Geometric optimization (using SPARC's internal routines)
```python
from sparc.calculator import SPARC
from ase.build import bulk
atoms = bulk("Al", cubic=True)
atoms.rattle(0.05)
atoms.calc = SPARC(h=0.25, kpts=(3, 3, 3), relax_flag=True, directory="run_opt")
atoms.get_potential_energy()
atoms.get_forces()
```

This example sets up a calculation for a rattled Aluminum primitive
unit cell, calculate with PBE functional, grid spacing of 0.25 Å, and
3 x 3 x 3 k-point grid.  Optimization of ionic positions is handled
with SPARC's default LBFGS routine.

3. AIMD in SPARC

```python
from sparc.calculator import SPARC
from ase.build import bulk
md_params = dict(md_flag=True, ion_temp=800, md_method="NVE", md_timestep=0.6, md_nstep=5)
atoms = bulk("Al") * (3, 3, 3)
atoms.rattle()
atoms.calc = SPARC(h=0.25, kpts=(1, 1, 1), directory="run_aimd", **md_params)
atoms.get_potential_energy()
```

This example runs a short NVE MD simulation (5 steps) at 800 K for 27 Al atoms.
If you want to extract more information about the MD simulation steps, take a look at `SPARC.raw_results`.

4. Geometric optimization using ASE's optimizers

The power of `SPARC-X-API` is to combine single point `SPARC` calculations with advanced ASE optimizers, such as BFGS, FIRE or GPMin. Example 2 can be re-written as:

```python
from sparc.calculator import SPARC
from ase.build import bulk
from ase.optimize import LBFGS
atoms = bulk("Al", cubic=True)
atoms.rattle(0.05)
atoms.calc = SPARC(h=0.25, kpts=(3, 3, 3), directory="run_opt_ase")
opt = LBFGS(atoms, alpha=90)
opt.run(fmax=0.02)
```

### 4. Command-line tools

A simple command wrapper `sparc-ase` is provided to add
support of SPARC file formats to the `ase` cli tools. Simple
replace `ase [subcommand] [args]` with `sparc-ase [subcommand] [args]`
to access your SPARC bundle files as you would use for other file
formats.  As an example, use `sparc-ase gui path/to/your/bundle.sparc`
for the visualization of atomistic structures. Depending on the
bundle's contents, this could display individual atoms or multiple
images.

[Fig. 2](#fig-2-a-screenshot-of-the-sparc-ase-program) is a screenshot showing the usage of `sparc-ase gui` to visualize a
short [MD trajectory](tests/outputs/NH3_sort_lbfgs_opt.sparc).

#### Fig 2. A screenshot of the `sparc-ase` program
<img width="1200" alt="image" src="https://github.com/alchem0x2A/SPARC-X-API/assets/6829706/e72329ff-7194-4819-94f8-486ef2218844">

### 5. Parameters and units used in `SPARC-X-API`

In the SPARC DFT code, all input parameters conventionally employ atomic units, such as Hartree and Bohr. Conversely, ASE objects (like `Atoms.positions`, `Atoms.cell`, `Atoms.get_potential_energy()`) utilize eV/Angstrom units.

When you set up a calculator as below:
```python
atoms.calc = SPARC(h=0.25, REFERENCE_CUTOFF=0.5, EXX_RANGE_PBE=0.16, **params)
```
inputs following ASE's convention (e.g., `h`) adopt eV/Angstrom units (thus the same setting can be applied to other DFT calculators),
On the other hand, all SPARC-specific parameters, which can often be recognized by their capitalized format (like `REFERENCE_CUTOFF`, `EXX_RANGE_PBE`), retain their original values consistent with their representation in the `.inpt` files.
The reasoning and details about unit conversion can be found in the [Rules for Input Parameters](https://github.com/alchem0x2A/SPARC-X-API/blob/master/doc/advanced_topics.md#rules-for-input-parameters-in-sparcsparc-calculator)  in Advanced Topics.


In order for `SPARC-X-API` to be compatible with other ASE-based DFT calculators,
there is a list of special parameters consistent with the ASE convention and uses Å / eV / GPa / fs
unit system:

| parameter name | meaning                         | example        | equivalent `SPARC` input |
|----------------|---------------------------------|----------------|--------------------------|
| `xc`           | Exchange-correlation functional | `xc=pbe` | `EXCHANGE_CORRELATION: GGA_PBE` |
|                |                                 | `xc=lda` | `EXCHANGE_CORRELATION: LDA_PZ` |
|                |                                 | `xc=rpbe` | `EXCHANGE_CORRELATION: GGA_RPBE` |
|                |                                 | `xc=pbesol` | `EXCHANGE_CORRELATION: GGA_PBEsol` |
|                |                                 | `xc=pbe0` | `EXCHANGE_CORRELATION: PBE0` |
|                |                                 | `xc=hf` | `EXCHANGE_CORRELATION: HF` |
|                |                                 | `xc=hse` or `xc=hse03` | `EXCHANGE_CORRELATION: HSE` |
|                |                                 | `xc=vdwdf1` or `xc=vdw-df` | `EXCHANGE_CORRELATION: vdWDF1` |
|                |                                 | `xc=vdwdf2` or `xc=vdw-df2` | `EXCHANGE_CORRELATION: vdWDF2` |
|                |                                 | `xc=scan` | `EXCHANGE_CORRELATION: SCAN` |
| `h`            | Real grid spacing    (Å)           | `h=0.2`        | `MESH_GRID: 0.38`  (in Bohr)         |
| `gpts`         | Explicit grid points |   `gpts=[10, 10, 10]` | `FD_GRID: 10 10 10` |
| `kpts`         | Kpoint mesh          |   `kpts=[3, 3, 3]`    | `KPOINT_GRID: 3 3 3` |
| `convergence`  | Dict of convergence criteria (see below) |  | |
|                | `energy`  eV/atom         | `convergence={"energy": 1e-4}` | `TOL_SCF: 3e-6` |
|                | `relax` (forces)  eV/Å            | `convergence={"relax": 1e-2}` | `TOL_RELAX: 2e-4` |
|                | `density` e/atom          | `convergence={`density`: 1e-6}`| `TOL_PSEUDOCHARGE: 1e-6` |

Users from other DFT codes can easily port their ASE codes to `SPARC-X-API` using the special parameters with minimal modification:

Example 1: VASP vs SPARC

```python
# Using VASP
from ase.calculators.vasp import Vasp
calc = Vasp(xc="rpbe", kpts=(9, 9, 9), directory="vasp-calc")
```
vs
```python
# Using SPARC
from sparc.calculator import SPARC
calc = SPARC(xc="rpbe", kpts=(9, 9, 9), directory="sparc-calc.sparc")
```

Example 2: GPAW (another real-space DFT code) vs SPARC
```python
# Using GPAW
from gpaw import GPAW
calc = GPAW(xc="PBE", kpts=(9, 9, 9), h=0.25, directory="vasp-calc", convergence={"energy": 1.e-4})
```
vs
```python
# Using SPARC
from sparc.calculator import SPARC
calc = SPARC(xc="PBE", kpts=(9, 9, 9), h=0.25, directory="sparc-calc.sparc", convergence={"energy": 1.e-4})
```

## Advanced Usage: SPARC-X-API as a Socket Interface

*Disclaimer: The socket communication feature in SPARC and SPARC-X-API are experimental and subject to changes until the release of v2.0 SPARC-X-API.*

### Overview
Experienced users can harness the power of SPARC and SPARC-X-API's
socket communication layer to build efficient and flexible
computational workflows. By integrating a socket communication
interface directly into SPARC, users can significantly reduce the
overhead typically associated with file I/O during calculation
restarts. This feature is particularly beneficial for tasks involving
repetitive operations like structural optimization and saddle point
searches, where traditional file-based communication can become a
bottleneck. The underlying software architecture is shown in [Fig. 3](#fig-3-sparc-electronic-calculations-with-socket-communication-across-hybrid-computing-platforms):

#### Fig. 3. SPARC electronic calculations with socket communication across hybrid computing platforms

![scheme-sparc-socket](doc/img/scheme_socket_hetero.png)




**Requirements**: the SPARC binary must be manually compiled from the source
code with [socket
support](https://github.com/alchem0x2A/SPARC/tree/socket) and with the
`USE_SOCKET=1` flag enabled (see the [installation
instructions](https://github.com/alchem0x2A/SPARC/tree/socket).

### Usage
The socket communication layer in SPARC and SPARC-X-API are designed for:
- **Efficiency:** Eliminates the need for intermediate file I/O, directly streaming data between processes.
- **Speed:** Enhances the performance of iterative calculations, crucial for large-scale simulations.
- **Flexibility:** Allows dynamic modification of calculation parameters without the need to restart the process.

The communication protocol implemented in SPARC and SPARC-X-API
adheres to the [i-PI protocol](https://github.com/i-pi/i-pi)
standard. Specifically, we implement the original i-PI protocol within
the SPARC C-source code, while the python SPARC-X-API uses a
backward-compatible protocol based on i-PI. The dual-mode design is aimed for both low-level and
high-level interfacing of the DFT codes, providing the following features as shown in [Fig. 4](#fig-4-overview-of-the-sparc-protocol-as-an-extension-to-the-standard-i-pi-protocol):

#### Fig. 4. Overview of the SPARC protocol as an extension to the standard i-PI protocol.
![scheme-sparc-protocol](doc/img/scheme_sparc_protocol.png)

Based on the scenarios, the socket communication layer can be accessed via the following approaches as shown in [Fig. 5](#fig-5-different-ways-of-using-sparcs-socket-mode):

#### Fig. 5. Different ways of using SPARC's socket mode.
![scheme-sparc-modes](doc/img/scheme-SPARC-socket-modes.png)


1. **SPARC binary only** ([Fig. 5](#fig-5-different-ways-of-using-sparcs-socket-mode) **a**)

   SPARC binary with socket support can be readily coupled with any i-PI compatible socker server, such as
   `ase.calculators.socketio.SocketIOCalculator`, for example

   ```python
   from ase.calculators.socketio import SocketIOCalculator
   from subprocess import Popen
   calc = SocketIOCalculator(port=31415)
   with calc:
       # Start sparc at background
	   process = Popen("mpirun -n 8 sparc -name SPARC -socket localhost:31415", shell=True)
	   # Single point calculations
	   process.kill()
   ```

   The end user is responsible for generating the input files and
   making sure the same atoms structures are used by
   `SocketIOCalculator` and the SPARC binary. The mode is also limited
   to be run on a single computer system.


2. **Local-only Mode** ([Fig. 5](#fig-5-different-ways-of-using-sparcs-socket-mode) **b**)

   Ideal for standalone calculations, this mode simulates a conventional calculator while benefiting from socket-based efficiency.

   ```python
   with SPARC(use_socket=True, **normal_parameters) as calc:
       # Execute single-point calculations
   ```
   For most users we recommend using this mode when performing a calculation on a single HPC node.

3. **Client (Relay) Mode** ([Fig. 5](#fig-5-different-ways-of-using-sparcs-socket-mode) **c**)

   In this mode, the `sparc.SPARC` calculator servers as a passive
   client which listens to a remote i-PI-compatible server. When
   messages are received on the client side, it relays the relevant
   message to a local SPARC binary and send results back through the
   socket pipe. The server side can either be a normal i-PI compatible
   server (such as `SocketIOCalculator`) or server-mode `sparc.SPARC` (see 4).

   Start the client by:
   ```python
   client = SPARC(use_socket=True,
	              socket_params=dict(host="host.address.com", port=31415))
   with client:
       client.run()
   ```

   Or via Command-Line:
   ```bash
   python -m sparc.client -s host:port
   ```

   Note: when running SPARC-X-API as a socket client, the atoms object
   can be ommitted (is the server also runs the SPARC protocol). When
   new atoms positions and parameters arrive, the client will
   automatically determine if it is necessary to restart the SPARC
   subprocess.

4. **Server Mode** ([Fig. 5](#fig-5-different-ways-of-using-sparcs-socket-mode) **d**)

   Paired with the client mode in (3), SPARC-X-API can be run as a
   socket server, isolated from the node that performs the
   computation. This can be useful for highly-distributed
   computational workflows.

   On the server node, run:
   ```python
   server_calc = SPARC(use_socket=True, socket_params=dict(port=31415, server_only=True), **normal_parameters)
   with server_calc:
       # Execute single point calculations for atoms_1
	   # Execute single point calculations for atoms_2
   ```

   In this case, the server will opens `0.0.0.0:31415` for
   connection. Make sure your server is directly accessible from the
   clients and the port is not occupied. The socker server is capable
   of receiving `raw_results` directly from the clients, making it
   possible to access `server_calc.raw_results` without access to the
   file systems on the client side.


### (In-progress) Controlling SPARC routines from socket interface

As shown in [Fig. 4](#fig-4-overview-of-the-sparc-protocol-as-an-extension-to-the-standard-i-pi-protocol),
the SPARC socket protocol designs allows bidirectional control of
internal SPARC routines. Local- or server-mode `sparc.SPARC`
calculators can communicate with the SPARC binary via functions like
`set_params`. This can be useful for applications like on-the-fly
force field learning, electron density fitting, setting up boundary
conditions etc. Applications will be updated in both SPARC and
SPARC-X-API repositories.



## Troubleshooting
Please refer to the [troubleshooting](doc/troubleshooting.md) guidelines

## Advanced topics
A detailed description about how the API works can be found [here](doc/advanced_topics.md)

## API changes
The API changes compared to the older release ([v0.1](https://github.com/SPARC-X/SPARC-X-API/tree/eac557f214b402122a506f88f38c7a8767283503)) are summarized [here](doc/changes_v0.1.md)

## How to contribute
Please refer to our [guidelines for contributors](doc/contribution_guideline.md)
