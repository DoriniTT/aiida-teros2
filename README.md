# AiiDA-TEROS
## Ab Initio Atomistic Thermodynamics for Surface Science using AiiDA WorkGraph

**TEROS (Thermodynamics of Extended Reactive Oxide Surfaces)** is a plugin for AiiDA that automates *ab initio* atomistic thermodynamics calculations for surface science using Quantum Espresso.

## Overview

AiiDA-TEROS provides a comprehensive framework for studying the thermodynamic stability of surfaces under different environmental conditions. By leveraging AiiDA's WorkGraph capability and Quantum Espresso's precision, it offers a fully automated workflow that handles:

* Generation of surface terminations from bulk structures
* Automated DFT calculations with error handling and recovery
* Thermodynamic analysis to determine surface stability
* Visualization of phase diagrams and stability regions

For researchers in catalysis, corrosion science, and materials engineering, AiiDA-TEROS simplifies the complex process of determining the most stable surface configurations under realistic temperature and pressure conditions.

## Features

* **Automated Surface Generation**: Creates symmetric surface slabs from any bulk structure
* **Quantum Espresso Integration**: Seamless interface with QE for accurate DFT calculations
* **Thermodynamic Analysis**: Calculates surface free energies and phase diagrams
* **Chemical Potential Mapping**: Explores stability across chemical potential space
* **Environment Effects**: Includes temperature and pressure effects
* **Error Handling**: Robust error recovery for calculation failures
* **Visualization**: Generates publication-quality phase diagrams
* **Provenance Tracking**: Full data lineage through AiiDA's provenance system
* **Extensibility**: Modular design for easy addition of new functionalities

## Installation

### Prerequisites

* Python 3.7+
* AiiDA 2.0+
* AiiDA-Quantum-Espresso plugin
* Quantum Espresso (installed on computational resources)

### Installation from PyPI

```bash
pip install aiida-teros
Installation from Sourcegit clone [https://github.com/yourusername/aiida-teros.git](https://github.com/yourusername/aiida-teros.git)
cd aiida-teros
pip install -e .
Verify Installationverdi plugin list aiida.workflows | grep teros
Quick StartBasic Examplefrom aiida import load_profile
from aiida.orm import Dict, StructureData, load_code
from aiida_teros.workflows.thermodynamics_workflow import create_thermodynamics_workgraph

# Load AiiDA profile
load_profile()

# Load structure from file
from ase.io import read
structure = StructureData(ase=read('MgO_bulk.cif'))

# Set up parameters
parameters = Dict(dict={
    'structure': {
        'miller_indices': [1, 0, 0],
        'min_layers': 4,
        'vacuum_size': 15.0
    },
    'espresso': {
        'ecutwfc': 60,
        'ecutrho': 480,
        'conv_thr': 1e-8,
        'occupations': 'smearing',
        'smearing': 'gaussian',
        'degauss': 0.01
    },
    'thermodynamics': {
        'temperature': 300,  # K
        'pressure': [1e-20, 1]  # atm
    }
})

# Load Quantum Espresso code
code = load_code('pw-7.1@localhost')

# Create and run workflow
wg = create_thermodynamics_workgraph()
inputs = {
    'surface_generation': {
        'prepare_bulk': {
            'structure': structure,
            'parameters': parameters['structure']
        }
    },
    'calculations': {
        'bulk_relaxation': {
            'parameters': parameters['espresso'],
            'code': code
        },
        'slab_calculations': {
            'parameters': parameters['espresso'],
            'code': code
        }
    },
    'phase_diagram': {
        'temperature': parameters['thermodynamics']['temperature'],
        'pressure_range': parameters['thermodynamics']['pressure']
    }
}

from aiida.engine import submit
process = submit(wg, **inputs)
print(f"Submitted workflow with PK: {process.pk}")
DocumentationWorkflow StructureAiiDA-TEROS implements a hierarchical WorkGraph structure:ThermoWorkGraph
├── Preparation Phase
│   ├── ValidateInputs Task
│   ├── OptimizeBulk Task
│   └── SurfaceGeneration WorkGraph
│       ├── AnalyzeBulkSymmetry Task
│       ├── GenerateMiller Task
│       ├── CreateTerminations Task
│       └── BuildSlabs Task
├── Calculation Phase
│   ├── BulkCalculation Task
│   └── SlabCalculations (Dynamic)
│       ├── Slab_1_Calculation Task
│       ├── Slab_2_Calculation Task
│       └── ... (created at runtime)
└── Analysis Phase
    ├── ExtractEnergies Task
    ├── CalculateSurfaceEnergies Task
    ├── BuildPhaseStability Task
    └── VisualizeResults Task
Available TasksStructure Tasks:prepare_bulk_structure: Prepare bulk structure for calculationsgenerate_miller_indices: Generate symmetrically distinct Miller indicescreate_surface_terminations: Create surface terminationsCalculation Tasks:run_espresso_scf: Run QE SCF calculationrun_espresso_relax: Run QE relaxation calculationAnalysis Tasks:calculate_formation_energy: Calculate formation energycalculate_surface_energies: Calculate surface energiesgenerate_phase_diagram: Generate phase diagramConfiguration OptionsStructure ParametersParameterTypeDescriptionmiller_indiceslistMiller indices [h,k,l]min_layersintMinimum number of atomic layersvacuum_sizefloatVacuum size in ÅsymmetricboolCreate symmetric slabsEspresso ParametersParameterTypeDescriptionecutwfcfloatPlane-wave cutoff energyecutrhofloatCharge density cutoffconv_thrfloatConvergence thresholdoccupationsstrOccupation typesmearingstrSmearing methoddegaussfloatSmearing widthThermodynamic ParametersParameterTypeDescriptiontemperaturefloatTemperature in KpressurelistPressure range [min, max] in atmchemical_potentialsdictChemical potential rangesExamplesSee the examples/ directory for complete working examples:examples/simple_oxide.py: Basic example for binary oxideexamples/complex_scenarios/ternary_oxide.py: Example for ternary oxideexamples/complex_scenarios/temperature_pressure.py: Temperature and pressure dependent exampleDevelopmentProject Structureaiida-teros/
├── aiida_teros/
│   ├── __init__.py
│   ├── data/
│   │   ├── __init__.py
│   │   └── surface.py
│   ├── utils/
│   │   ├── __init__.py
│   │   ├── structure.py
│   │   ├── constants.py
│   │   └── thermo.py
│   ├── tasks/
│   │   ├── __init__.py
│   │   ├── structure_tasks.py
│   │   ├── espresso_tasks.py
│   │   └── analysis_tasks.py
│   ├── workflows/
│   │   ├── __init__.py
│   │   ├── surface_workflow.py
│   │   ├── calculation_workflow.py
│   │   └── thermodynamics_workflow.py
│   ├── io/
│   │   ├── __init__.py
│   │   ├── inputs.py
│   │   ├── outputs.py
│   │   └── visualization.py
│   └── cli/
│       ├── __init__.py
│       └── commands.py
├── examples/
│   ├── simple_oxide.py
│   └── complex_scenarios/
├── tests/
│   ├── __init__.py
│   ├── test_utils.py
│   ├── test_tasks.py
│   └── test_workflows.py
├── setup.py
├── pyproject.toml
└── README.md
Running Testspip install -e .[testing]
pytest -v
ContributingContributions are welcome! Please feel free to submit a Pull Request.Fork the repositoryCreate your feature branch (git checkout -b feature/amazing-feature)Commit your changes (git commit -m 'Add some amazing feature')Push to the branch (git push origin feature/amazing-feature)Open a Pull RequestPlease make sure your code adheres to the project's style guide and passes all tests.LicenseThis project is licensed under the MIT License - see the LICENSE file for details.AcknowledgmentsThis project builds on the AiiDA infrastructure for workflow management and provenance trackingQuantum Espresso is used for the underlying quantum mechanical calculationsThis work was supported by [Your funding agency or institution]CitationIf you use AiiDA-TEROS in your research, please cite:@software{aiida_teros,
  author       = {Your Name},
  title        = {AiiDA-TEROS: Ab Initio Atomistic Thermodynamics for Surface Science},
  year         = {2025},
  publisher    = {GitHub},
  journal      = {GitHub repository},
  url          = {
