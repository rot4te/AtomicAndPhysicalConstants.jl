# [CODATA Releases](@id man-codata)

The [CODATA](https://codata.org/) task group on fundamental constants publishes
updated recommended values every four years.  AtomicAndPhysicalConstants.jl
ships constants from every release since 2002 and lets you choose which one to
use at runtime.

## Supported releases

| Release | Notes |
|---------|-------|
| 2002 | earliest supported; `M_TRITON` is not tabulated in this release (`CODATA2002.M_TRITON == NaN`) |
| 2006 | |
| 2010 | first release with `G_HELION`, `ANOMALY_ELECTRON`, `ANOMALY_MUON` |
| 2014 | |
| 2018 | |
| 2022 | **default** |

## Checking the active release

```julia
using AtomicAndPhysicalConstants
RELEASE_YEAR   # Int – the currently active release year
```

## Changing the release

```julia
using AtomicAndPhysicalConstants

set_release(year = "2014")
# [ Info: The default CODATA release is now 2014.
#         Restart your Julia session for this change to take effect.
```

The setting is stored via [Preferences.jl](https://github.com/JuliaPackaging/Preferences.jl)
and persists across Julia sessions until changed again.

!!! note "Restart required"
    Constants are resolved at package load time, so a Julia session restart
    is required for the new release to take effect.

!!! warning "Unsupported releases"
    Only the years listed under **Supported releases** above are valid. If the
    stored preference points at an unsupported release (for
    example after hand-editing `LocalPreferences.toml`), the package raises an
    error at load time listing the valid options rather than silently falling
    back to the default.

To revert to the default (2022):

```julia
set_release()          # no argument → defaults to "2022"
```


## Direct access to release structs

Each release is also exported as a named `CODATA_release` struct.
You can inspect or use individual values without changing the global default:

```julia
CODATA2018.M_ELECTRON    # electron mass from the 2018 release
CODATA2014.C_LIGHT       # speed of light from the 2014 release
```


## Constants not in CODATA

The pion masses (`M_PION_0`, `M_PION_CHARGED`) are not published in any CODATA
release.  They are taken from Particle Data Group (PDG) tables and remain
constant regardless of the selected release year.

`CLASSICAL_RADIUS_FACTOR` is derived rather than tabulated: it is computed as
`R_ELECTRON * M_ELECTRON` from the active release.  It is therefore not a field
of the `CODATA_release` structs — `CODATA2018.CLASSICAL_RADIUS_FACTOR` does not
exist, only the exported top-level constant.

## Constants that differ from the tabulated release value

`G_DEUTERON`, `G_HELION`, and `G_TRITON` are pulled from the NIST/CODATA tables
but are **rescaled before export**.  NIST tabulates these g-factors against the
nuclear magneton ``\mu_N = e\hbar/2m_p``; the package converts them to the
particle's own magneton so that the gyromagnetic anomaly ``a = (g-2)/2`` comes
out correct:

```julia
G_DEUTERON == CODATA2022.G_DEUTERON * M_DEUTERON / M_PROTON   # active release
```

So the exported constant and the release-struct field deliberately disagree:

```julia
CODATA2022.G_DEUTERON   # 0.8574382335  — as published by NIST
G_DEUTERON              # 1.7140254606… — renormalized for use in a = (g-2)/2
```

Both the mass and the g-factor come from the active release, so the conversion
stays self-consistent when you call `set_release`.  The unscaled values are also
kept as the unexported `_G_DEUTERON`, `_G_HELION`, and `_G_TRITON`.  See
[Physical Constants](@ref man-constants) for the full explanation.

The other g-factors (`G_ELECTRON`, `G_MUON`, `G_PROTON`, `G_NEUTRON`) are
exported exactly as tabulated.


## Release coverage caveats

The gyromagnetic anomalies (`ANOMALY_ELECTRON`, `ANOMALY_MUON`) and the helion
g-factor (`G_HELION`) were not individually tabulated by CODATA until the 2010
release.  Using a pre-2010 release will still define these symbols; worth
verifying their values against the source data if you are doing precision work
prior to that release.
