# Installing Superconducting-Toolkit (`sctk.x`) with Quantum ESPRESSO

## Theory
Superconducting-Toolkit (`sctk.x`) is a first-principles Fortran program package based on Density Functional Theory for Superconductors (SCDFT) that evaluates superconducting transition temperatures ($T_c$), superconducting gaps ($\Delta_{n\mathbf{k}}$), coherence lengths ($\xi_0$), and magnetic penetration depths ($\lambda_L$) without empirical parameters. Because `sctk.x` relies on normal-state electronic band structures, electron-phonon matrix elements, and dynamically screened Coulomb interactions computed by Quantum ESPRESSO (QE), it is distributed as an add-on module that integrates directly into the QE source tree. Applying `patch.diff` modifies QE's build system and core routines (`pw.x`, `ph.x`, `pp.x`) so they can export the required wavefunctions and interaction kernels. Running `configure` detects the local Fortran compiler, MPI parallelization, BLAS/LAPACK linear algebra libraries, and FFTW packages to generate `make.inc`, while `make pw ph pp sctk` compiles both the patched QE binaries and `sctk.x` into the `bin/` directory. Adding `bin/` to `$PATH` (or creating a shell alias) allows `sctk.x` and its paired QE executables to be invoked from any working directory.

## Procedure

### 1. Install Required System Dependencies
```bash
sudo apt update
sudo apt install -y build-essential gfortran git patch libopenmpi-dev openmpi-bin libblas-dev liblapack-dev libfftw3-dev
```

### 2. Clone Quantum ESPRESSO and SCTK
Clone the Quantum ESPRESSO repository, enter the directory, clone SCTK inside it, and apply the compatibility patch:
```bash
git clone https://gitlab.com -b master
cd q-e
git clone https://github.com -b main SCTK
patch -p1 < SCTK/patch.diff
```
> **Optional:** To use the latest experimental features (such as finite-momentum Cooper pairs for coherence length and penetration depth), replace `-b master` with `-b develop` for `q-e` and `-b main` with `-b develop` for `SCTK`.

### 3. Configure and Compile
```bash
./configure --enable-openmp
make pw ph pp sctk
```
> **Note:** Compilation may take several minutes depending on the number of CPU cores.

### 4. Verify the Compiled Binaries and Get the Full Path
```bash
cd bin
ls -lh sctk.x pw.x ph.x pp.x
pwd
```

### 5. Add `bin/` to `$PATH`
Choose **Option A** to make `sctk.x` and this patched QE build available permanently in all terminals, or **Option B** if you already have another version of Quantum ESPRESSO installed and only want to activate this build on demand.

**Option A: Permanent `PATH` (Recommended for fresh setups)**
```bash
echo 'export PATH="your directory:\$PATH"' >> ~/.bashrc
source ~/.bashrc
```

**Option B: On-Demand Alias (Recommended if another QE version is already installed)**
```bash
echo 'alias use_sctk="export PATH=\"your directory:\$PATH\""' >> ~/.bashrc
source ~/.bashrc
```
*(When using Option B, type `use_sctk` in your terminal before running SCTK calculations.)*

### 6. Confirm Installation
```bash
which sctk.x pw.x ph.x
sctk.x
```

* **Note:** Replace `"your directory"` in Step 5 with the exact output path printed by `pwd` in Step 4 (for example, `/home/username/q-e/bin`) — leaving the literal placeholder text will break your `$PATH`.
* **Note:** Running `sctk.x` with no input file is only to verify that the system finds and executes the binary. It will wait for standard input (`stdin`) until you press `Ctrl+C` or `Ctrl+D` to exit.
