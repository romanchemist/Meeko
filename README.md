# Meeko: preparation of small molecules for AutoDock

[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)
[![API stability](https://img.shields.io/badge/stable%20API-no-orange)](https://shields.io/)
[![PyPI version fury.io](https://img.shields.io/badge/version-0.2-green.svg)](https://pypi.python.org/pypi/ansicolortags/)

Meeko reads a chemoinformatics molecule object (currently Open Babel) and writes a string (or file)
in PDBQT format for use with [AutoDock-Vina](https://github.com/ccsb-scripps/AutoDock-Vina)
and [AutoDock-GPU](https://github.com/ccsb-scripps/AutoDock-GPU). Additionally, it has tools for post-processing
of docking results which are not yet fully developed. Meeko supports the following features:
* Docking with explicit water molecules attached to the ligand [(paper)](https://pubs.acs.org/doi/abs/10.1021/jm2005145)
* Sampling of macrocyclic conformations during docking [(paper)](https://link.springer.com/article/10.1007/s10822-019-00241-9)
* Creation of RDKit molecules with docked coordinates from PDBQT or DLG files without loss of bond orders.

Meeko is developed by the [Forli lab](https://forlilab.org/) at the
[Center for Computational Structural Biology (CCSB)](https://ccsb.scripps.edu)
at [Scripps Research](https://www.scripps.edu/).

## Dependencies

* Python (>=3.5)
* Numpy
* Scipy
* Openbabel (>=3)
* RDKit

Conda or Miniconda can install the dependencies:
```bash
conda install -c conda-forge numpy scipy openbabel rdkit
```

## Installation (from PyPI)
```bash
$ pip install meeko
```
If using conda, `pip` installs the package in the active environment.

## Installation (from source)

```bash
$ git clone https://github.com/forlilab/Meeko
$ cd Meeko
$ pip install .
```

Optionally include `--editable`. Changes in the original package location
take effect immediately without the need to run `pip install .` again.
```bash
$ pip install --editable .
```

## Examples using the command line scripts
```console
mk_prepare_ligand.py -i molecule.sdf -o molecule.pdbqt
mk_prepare_ligand.py -i multi_mol.sdf --multimol_outdir folder_for_pdbqt_files
mk_copy_coords.py vina_results.pdbqt -o vina_results.sdf
mk_copy_coords.py adgpu_results.dlg -o adgpu_results.sdf
```

## Quick Python tutorial

#### 1. flexible macrocycle with attached waters

```python
from meeko import MoleculePreparation
from meeko import obutils

input_molecule_file = 'example/BACE_macrocycle/BACE_4.mol2'
mol = obutils.load_molecule_from_file(input_molecule_file)

preparator = MoleculePreparation(keep_nonpolar_hydrogens=False, macrocycle=True, hydrate=True)
preparator.prepare(mol)
preparator.show_setup()

output_pdbqt_file = "test_macrocycle_hydrate.pdbqt"
preparator.write_pdbqt_file(output_pdbqt_file)
```

#### 2. RDKit molecule from docking results

Assuming that 'docked.dlg' was written by AutoDock-GPU and that Meeko prepared the input ligands.

```python
from meeko import PDBQTMolecule

with open("docked.dlg") as f:
    dlg_string = f.read()
pdbqt_mol = PDBQTMolecule(dlg_string, is_dlg=True)

# alternatively, read the .dlg file directly
pdbqt_mol = PDBQTMolecule.from_file("docked.dlg", is_dlg=True)

for pose in pdbqt_mol:
    rdkit_mol = pose.export_rdkit_mol()
```

For Vina's output PDBQT files, omit `is_dlg=True`.
```python
pdbqt_mol = PDBQTMolecule.from_file("docking_results_from_vina.pdbqt")
```
