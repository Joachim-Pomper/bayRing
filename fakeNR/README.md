# FakeNR Injection Catalogs

`FakeNR` catalogs are small, synthetic ringdown injections generated directly by
bayRing from metadata. They are useful for testing Kerr-mode recovery without
shipping waveform data files. Additionally, they are great for testing implementations, as the mode content is exactly known. BayRing automatically plots injection lines in the results if a `FakeNR` catalog is used.

## Directory Layout

Create one directory per injection setup:

```text
fakeNR/
  My_injection/
    meta_data.ini
```

Point a bayRing config at that directory with:

```ini
[NR-data]
catalog = FakeNR
dir     = fakeNR/My_injection
ID      =
error   = constant-0.0001
```

If `ID` is empty, bayRing reads `meta_data.ini`. If `ID = example`, bayRing reads
`meta_data_example.ini` from the same directory.

## Minimal Metadata File

```ini
[signal-parameter]
t-start = 0.0
t-end   = 100.0
dt      = 0.2

[event-parameter]
final-mass   = 67.0
final-spin   = 0.67
final-charge = 0.0

[kerr-model]
kerr-amplitudes = {"2220": 1.0}
kerr-phases     = {"2220": 0.0}
```

The mode key format is `slmn`, with spin weight `s=2`. For example:

```ini
kerr-amplitudes = {"2220": 1.0, "2221": 0.5}
kerr-phases     = {"2220": 0.0, "2221": 1.5}
```

Negative `m` values are encoded with a minus sign after `l`:

```ini
kerr-amplitudes = {"22-20": 0.1}
kerr-phases     = {"22-20": 2.0}
```

This key is interpreted as `(s, l, m, n) = (2, 2, -2, 0)`.

## Optional Sections And Keys

The following defaults are used when keys are omitted:

```ini
[signal-parameter]
t-start = 0.0
t-end   = 100.0
dt      = 0.2

[mimick-sxs]
times-from-sxs = false
error-from-sxs = false
sxs-simulation = 

[event-parameter]
final-mass   = 67.0
final-spin   = 0.67
final-charge = 0.0

[kerr-model]
kerr-amplitudes      = {}
kerr-phases          = {}
```

Notes:

- `t-start` must be non-negative.
- Use hyphenated option names, for example `times-from-sxs`, not
  `times_from-sxs`.
- `times-from-sxs = true` asks bayRing to use the time grid and peak time from
  the configured SXS simulation.
- `error = constant-X` or `error = gaussian-X` in `[NR-data]` sets a constant
  complex error scale of `X`. `error = from-SXS-NR` uses the SXS-derived error
  and should be paired with SXS-derived times.

## Matching The Inference Config

The model modes in the bayRing config should match the injected modes you want
to recover:

```ini
[Model]
template  = Kerr
QNM-modes = 220,221
```

For linear inversion, all non-linear parameters must be fixed or absent from the
inference problem. For nested sampling or minimization, set the corresponding
priors in `[Priors]`.

## Existing Examples

This directory contains a few ready-made examples:

- `Kerr_2220_2221`: two positive-m modes, `220` and `221`.
- `Kerr_up_to_5th_overtone`: positive and negative-m modes through several
  overtones.
