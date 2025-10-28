# AlloCraft Analysis Repository

> **Note:** The current code is a refactored version of the scripts built by Mahdi Hijazi and is mainly refactored from [https://github.com/barth-lab/AlloDy](https://github.com/barth-lab/AlloDy) and [https://github.com/mahdiofhijaz/MDprot](https://github.com/mahdiofhijaz/MDprot).

This repository contains a suite of Matlab scripts for performing allosteric dynamics (AlloDy) analysis on molecular dynamics (MD) simulation trajectories. The workflow is designed to process raw trajectory data and perform a comprehensive set of calculations, including RMSD, RMSF, contact maps, Principal Component Analysis (PCA), GPCR order parameters, dihedral analysis, mutual information (MI), and Kullback- Leibler (KL) divergence.

## Repository Structure

The repository is organized into two main directories:

*   `scripts2/`: Contains all the Matlab `.m` scripts required to run the analysis. This includes the main workflow scripts, configuration files, and various utility functions organized into subdirectories (`src`, `MDprot`, `util`, `alloPathCalc`).
*   `data/`: Serves as the input and output hub. It should contain your MD simulation data (PDBs, trajectories) and is also where the analysis results will be saved.

## Generated Data

The analysis pipeline generates a variety of data, which is typically saved in a `md2pathdev` subdirectory within your specific data folder (e.g., `data/DA_F6.44M_S5.46G/md2pathdev/`). Key outputs include:

*   **Plots and Figures (.fig, .pdf):** Visualizations of RMSD, RMSF, contact maps, PCA results, and KL divergence.
*   **PDB Files (.pdb):** Structures with RMSF or KL divergence values saved in the B-factor column for easy visualization.
*   **MAT-files (.mat):** Contain processed data such as dihedral time series, MI matrices, and PCA results.
*   **Text and Excel Files (.txt, .xlsx):** Lists of interacting residues and other tabular data.

## Setup and Configuration

Before running, you must configure two key files in the `scripts2/` directory: `input_md2path.m` and `input_kldiv.m`. These scripts use a `global settings` variable to manage all parameters.

### Important Parameters to Modify

#### 1. `input_md2path.m` (Main Analysis)

*   `settings.mydir`: **Crucial.** Set this to the directory containing your simulation data. The script expects this directory to contain subfolders like `run1`, `run2`, etc.
*   `settings.xtcName`: The name of your trajectory file within each `run*` folder. The default is `'traj.dcd'`. **Note:** The scripts are optimized for `.dcd` files. If you use `.xtc` files, a conversion will be attempted using a VMD Tcl script (`load_save.tcl`), but direct `.dcd` usage is recommended.
*   `settings.chains`: Defines the chain identifiers for the `[receptor, G protein, ligand]`. Use a hyphen `-` for any missing chains (e.g., `'A-B'` for a receptor and ligand). The default is `'ACB'`.
*   `settings.numRuns`: The total number of simulation runs (e.g., `run1`, `run2`, ...) to analyze from your data directory.
*   `settings.stride`: The step size for reading trajectory frames. A value of `1` reads every frame, while a value of `10` would read every 10th frame. This is useful for reducing computational load on very large trajectories.

#### 2. `input_kldiv.m` (Comparative Analysis)

*   `settings.mydir`: The data directory for your primary or "perturbed" system.
*   `settings.refdir`: **Reference System.** The data directory for the second simulation that you want to compare against the primary one. KL divergence measures the difference between the dihedral distributions of the `mydir` and `refdir` systems (e.g., comparing a mutant against a wild-type).
*   `settings.mainName` & `settings.refName`: Short, descriptive names for your main and reference systems used for labeling outputs.

#### 3. PDB Filenames (A Common Source of Errors)

The scripts expect specific PDB filenames within the main analysis functions. If your files are named differently, you **must** update them.

*   In `md2pathMain.m`:
    ```matlab
    % Look for this line and change "prot_pymol.pdb" if needed
    database.read(fullfile(settings.mydir, "prot_pymol.pdb"), settings.chains, "Protein");
    ```
*   In `kldivMain.m`:
    ```matlab
    % Change "prot_pymol.pdb" and "prot.pdb" if your files are named differently
    database.read(fullfile(settings.mydir, "prot_pymol.pdb"), settings.chains, settings.mainName);
    database.read(fullfile(settings.refdir, "prot.pdb"), settings.chains, settings.refName );
    ```

## Running the Analysis

The analysis can be executed from the command line or the interactive MATLAB GUI.

### 1. MATLAB GUI (Interactive Environment)

1.  **Open MATLAB.**
2.  **Navigate to the scripts directory.** In the "Current Folder" toolbar, paste the absolute path to the `scripts2` directory and press Enter.
    ```matlab
    /path/to/AlloDy_Analysis_Repo/scripts2
    ```
3.  **Run the scripts.** In the "Command Window" (at the `>>` prompt), type the following commands. The semicolon at the end of each command suppresses verbose output and allows them to run sequentially.
    ```matlab
    input_md2path;
    md2pathMain;
    input_kldiv;
    kldivMain;
    ```

### 2. Command-Line Interface (CLI)

For batch processing or running on a remote server, the CLI is ideal.

1.  **Open your terminal.**
2.  **Execute the analysis.** Use the `matlab -batch` command to run the entire workflow non-interactively. This single command changes to the correct directory and runs the scripts in order.
    ```bash
    matlab -batch "cd('/path/to/AlloDy_Analysis_Repo/scripts2'); input_md2path; md2pathMain; input_kldiv; kldivMain;"
    ```
    **Note:** The `-batch` flag is essential for non-interactive execution. Ensure your scripts do not contain any `input()` calls, as this will cause an error in batch mode.




# Version & Compatibility

| MATLAB | Status | Notes |
|--------|--------|-------|
| R2025b | ✅ OK | Needs 1-line fix (below). |
| R2024b | ✅ OK | No changes needed in our tests. |
| R2024a | ✅ OK | No changes needed in our tests. |


**Please include your MATLAB version when reporting issues.**

---

## Quick fix for R2025b

`calcPlotOrderParameters.m` may use `size(plots)` where `length` is safer.

**Path:** `scripts2/src/calcPlotOrderParameters.m`

**Change:**

```matlab
% Old
nplots = size(plots);

% New
nplots = length(plots);
```

**Then run:**

```matlab
input_md2path; md2pathMain; input_kldiv; kldivMain;
```

---

## R2024a first-run workaround

If `input_md2path.m` errors on the first run, try again.

If it persists:

```matlab
restoredefaultpath; rehash toolboxcache;
input_md2path; md2pathMain; input_kldiv; kldivMain;
```

---

## Support window

* **Recommended:** R2024a or newer
* **Minimum likely to work:** R2021b+ (not actively tested)
* **Not supported:** R2019b

---

## When opening an issue

Include:

* MATLAB version (e.g., R2025b)
* OS (Linux/Windows/macOS)
* First ~10 lines of the error
* Whether you applied the `length(plots)` fix above

---

## Optional guard (to avoid size/length pitfalls)

```matlab
if ~isscalar(plots)
    nplots = length(plots);
else
    nplots = plots; % already numeric
end
```
