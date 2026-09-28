# swisstrace

Correct and calibrate raw Swisstrace **twilite** automatic blood-sampler `.crv`
recordings into whole-blood activity curves, with PMOD, QC plots and BIDS PET blood
exports. Equivalent tools are available in R and Python.

| Folder | Contents | Installation from this checkout |
| --- | --- | --- |
| [R/](R/) | Complete R package: source, metadata, documentation and tests | `remotes::install_local("R")` |
| [Python/](Python/) | Installable Python package, documentation and tests | `python -m pip install ./Python` |

The repository root is no longer an R package. Existing R installations should
use the `R` subdirectory:

```r
remotes::install_github("CPernet/swisstrace", subdir = "R")
library(swisstrace)
swisstrace_process("recording.crv", calibration_factor = 0.425,
                   isotope = "F18", pet_start = "11:16:30")
```

```python
from swisstrace import swisstrace_process

paths = swisstrace_process("recording.crv", calibration_factor=0.425,
                           isotope="F18", pet_start="11:16:30")
```

See the [R guide](R/README.md) or [Python guide](Python/README.md) for batch
conversion, calibration lookup, frame schemes and BIDS output.

Both implementations use the coincidence channel (the first counter), subtract
background, correct decay relative to PET start, and apply the calibration factor.
Native sampling is retained unless a frame scheme is supplied. Supply the correct
isotope or an explicit half-life in seconds; it cannot be inferred from the file.
The first corrected sample/frame is zeroed by default. These are whole-blood
curves, without plasma, metabolite or dispersion correction.

## Development and validation

```sh
python -m pip install -e './Python[test,excel]'
python -m pytest Python/tests
Rscript -e 'testthat::test_local("R")'
```

R development commands such as `devtools::load_all()`, `devtools::document()` and
`devtools::install()` run inside `R/` (or receive `"R"` as their package path).
The R package's own source directory is consequently `R/R/`.

Tests use synthetic recordings, including known analytical correction values.
Direct R/Python comparisons run when R plus `tibble` and `jsonlite` are available;
otherwise those tests are skipped. GitHub CI installs both runtimes and checks
numerical agreement, calibration lookup and exported curves. Real recordings and
vendor reference outputs are not included, so the Python port has not been
independently validated against those reference outputs.
