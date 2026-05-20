# allesfitter — Installation Guide

## Requirements

- **Python 3.11** (recommended) — Python 3.12 may work, but Python 3.13 is **not supported** due to a known incompatibility with `llvmlite`/`numba`.
- [Anaconda](https://www.anaconda.com/download) or [Miniconda](https://docs.conda.io/en/latest/miniconda.html)
- A Fortran compiler (`gfortran`) — required to build the `ellc` shared library

---

## Step 1 — Create a dedicated conda environment

It is strongly recommended to install *allesfitter* in a clean environment to avoid dependency conflicts.

```bash
conda create -n allesfitter python=3.11 -y
conda activate allesfitter
```

---

## Step 2 — Install `llvmlite`, `numba`, and `gfortran` via conda-forge

`llvmlite` (required by `numba`, which is required by `wotan`) and `gfortran` (required to compile the `ellc` Fortran library) must be installed via `conda-forge` as **pre-built binaries**.

```bash
conda install -c conda-forge llvmlite numba gfortran -y
```

> **Why conda-forge?**  
> - `llvmlite` fails to build from source on Python 3.11+ due to a `setuptools` compatibility issue (`spawn() got an unexpected keyword argument 'dry_run'`).  
> - `gfortran` is required by `ellc` to compile its Fortran shared library (`libellc.so`). Without it, `ellc` installs as a pure Python wheel and fails at runtime with `OSError: libellc.so not found`.

---

## Step 3 — Install the required dependencies via pip

```bash
pip install tqdm \
            celerite \
            dynesty \
            emcee \
            corner \
            rebound \
            transitleastsquares \
            seaborn \
            statsmodels
```

Then install `wotan` **without dependencies** (to prevent pip from trying to rebuild `llvmlite` from source):

```bash
pip install wotan --no-deps
```

---

## Step 4 — Build and install `ellc` from source

`ellc` must be built from source so its Fortran library (`libellc.so`) gets compiled. The pip-installed wheel does **not** include this compiled binary.

```bash
git clone https://github.com/pmaxted/ellc.git /tmp/ellc
cd /tmp/ellc

# Build the Fortran library explicitly (more reliable than pip install)
python setup.py build_ext --inplace
python setup.py install
```

> **Verify the Fortran library was compiled:**
> ```bash
> find $HOME/miniconda3/envs/allesfitter -name "libellc.so"
> # Must return a path — if empty, the build failed silently
> ```
> If empty, check that `which gfortran` points to the conda environment's gfortran, then retry.

---

## Step 5 — Pin NumPy to a compatible version

`numba 0.61` requires `numpy < 2.2`. If a newer numpy was pulled in, pin it:

```bash
pip install "numpy<2.2"
```

---

## Step 6 — Install allesfitter

Clone the repository (if you haven't already):

```bash
git clone https://github.com/MNGuenther/allesfitter.git
cd allesfitter
```

Then install the package:

```bash
pip install .
```

---

## Step 7 — Apply source code patches for NumPy 2.x compatibility

The allesfitter source uses `np.VisibleDeprecationWarning` and `np.RankWarning`, which were removed in NumPy 2.x. The following files need to be patched:

- `allesfitter/basement.py`
- `allesfitter/mcmc.py`
- `allesfitter/computer.py`
- `allesfitter/general_output.py`
- `allesfitter/nested_sampling.py`

In each file, replace:

```python
warnings.filterwarnings('ignore', category=np.VisibleDeprecationWarning)
warnings.filterwarnings('ignore', category=np.RankWarning)
```

with:

```python
warnings.filterwarnings('ignore', category=np.exceptions.VisibleDeprecationWarning)
warnings.filterwarnings('ignore', category=RuntimeWarning)
```

> These patches are already applied if you installed from this fork of the repository.

---

## Verifying the installation

```bash
conda activate allesfitter
conda run -n allesfitter python -c "import allesfitter; print('allesfitter imported successfully')"
```

---

## Troubleshooting

### `TypeError: spawn() got an unexpected keyword argument 'dry_run'`

Occurs when building `llvmlite` from source on Python 3.11+. **Fix:** install `llvmlite` via `conda-forge` (Step 2).

### `OSError: libellc.so not found`

Occurs when `ellc` was installed as a pure Python wheel without compiling the Fortran library. **Fix:** install `gfortran` via `conda-forge` (Step 2) and build `ellc` from source (Step 4).

### `AttributeError: module 'numpy' has no attribute 'VisibleDeprecationWarning'`

Occurs with NumPy 2.x. **Fix:** apply the source code patches described in Step 7.

### `ModuleNotFoundError: No module named 'seaborn'` / `No module named 'statsmodels'`

These packages are not listed as explicit dependencies in `setup.py` but are required at import time. **Fix:** install them via pip (Step 3).

---

## Installed packages summary

| Package | Source | Notes |
|---|---|---|
| `llvmlite` | conda-forge | Must be installed via conda, not pip |
| `numba` | conda-forge | Must be installed via conda, not pip |
| `gfortran` | conda-forge | Required to compile `ellc`'s Fortran library |
| `ellc` | GitHub source build | Must be built from source; pip wheel lacks `libellc.so` |
| `emcee` | pip | MCMC sampler (required at import time) |
| `corner` | pip | Corner plots (required at import time) |
| `tqdm` | pip | Progress bars |
| `celerite` | pip | Gaussian Process models |
| `dynesty` | pip | Nested sampling |
| `rebound` | pip | N-body integrator |
| `transitleastsquares` | pip | Transit search |
| `seaborn` | pip | Plotting (required at import time) |
| `statsmodels` | pip | Statistics (required at import time) |
| `wotan` | pip (`--no-deps`) | Detrending; uses numba/llvmlite from conda |
| `numpy` | pip (`<2.2`) | Pinned for numba 0.61 compatibility |
| `allesfitter` | local (`pip install .`) | This package |
