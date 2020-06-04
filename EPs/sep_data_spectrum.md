# Add functionality for spectrum data


author: Kyle Boone <!-- your name -->

date-created: 2020 May 18 <!-- replace with the date you submit the APE -->

date-last-revised: 2020 June 4 <!-- keep this up to date anytime something changes -->

date-accepted: <!-- replace with accepted date -->

status: Discussion  <!-- one of: Discussion, Accepted, Rejected -->


## Abstract

The goal of this proposal is to add functionality to `sncosmo` to handle spectrum data.
We propose to add a `Spectrum` class that represents a spectra taken of a supernova, and
implement functionality to work with this data.


## Detailed description

`sncosmo` is currently limited to working with photometric data. However, spectra are
often obtained of supernovae, and it would be very useful if `sncosmo` could use this
additional data. We propose to add functionality to `sncosmo` to work with spectra.

There are several ways in which spectra can be used for supernova cosmology. First,
spectra can provide additional constraints when fitting models to observations of a
supernova. Models such as the SNEMO model that was recently added to `sncosmo` are
designed to be able to fit individual spectra of each supernova rather than being
limited to photometry. Surveys such as the SNfactory are obtaining spectrophotometric
timeseries of supernovae that can then be fit directly with these models, potentially
providing additional information beyond what is captured by photometry alone. 

Given a spectrophotometric timeseries, it is possible to generate synthetic photometry
in arbitrary bands. This photometry has many uses, such as comparing between experiments
or using light curve fitters that take photometry as input. `sncosmo` already provides
functionality to generate photometry from spectral time series models, and this same
functionality could be applied to spectrophotometric spectra.

Less-well-calibrated spectra can also be useful for typing supernovae or constraining
models. However, they often require some warping of the spectrum or removal of host
galaxy contamination. This functionality is beyond the scope of this EP, but we would
like to set up a framework to be able to include it in the future.


## Branches and pull requests

An initial discussion of this work can be found in
https://github.com/sncosmo/sncosmo/issues/262.

The main PR for this EP can be found as https://github.com/sncosmo/sncosmo/pull/269.

This PR is based off of and supercedes the following PRs:
- https://github.com/sncosmo/sncosmo/pull/265 has an implementation of fitting models to
  spectral time series.
- https://github.com/sncosmo/sncosmo/pull/259 implements basic synthesized photometry.


## Implementation

The building block for all of this work is adding a `Spectrum` class to `sncosmo` to
handle spectrum data. There is currently a `Spectrum` class that is designed to
represent a model of a spectrum that will be renamed to `SpectrumModel`. The `Spectrum`
class should be different as it represents binned observations of a spectrum rather than
a continuous model. This is similar to the distinction between the `SourceModel` and
`PhotometricData` classes that currently exist.

The `Spectrum` class needs to encapsulate several aspects of an observed spectrum. An
observed spectrum can be thought of as a set of many small filters that are all adjacent
to each other. The class needs to keep track of the edges of these filters and the bin
widths. Spectra also often come with uncertainties or covariance matrices, and these
need to be properly encapsulated. Our implementation is inspired by the
`PhotometricData` class that deals with similar issues. The `Spectrum` API is designed
to be compatible with the `SpectrumModel` API in order to avoid breaking any code that
was previously using the old internal `Spectrum` class.

### Initializing a spectrum

A spectrum is initialized with a list of wavelengths, fluxes and optionally
uncertainties. The `Spectrum` initializer has the following signature:

```python
def __init__(self, wave=None, flux=None, fluxerr=None, fluxcov=None, bin_edges=None,
             wave_unit=u.AA, unit=(u.erg / u.s / u.cm**2 / u.AA), time=None):
```

For the most common use case (a list of wavelengths, fluxes and uncertainties), all of
these attributes can be passed directly into the function. This is compatible with the
old `Spectrum` class which only supported arguments of `wave` and `flux`.

Internally, the `Spectrum` class stores the edges of each bin rather than the centers of
each bin. We use the `_estimate_bin_edges` function to recover the bin edges perfectly
for linear binning, and with a fractional error of (dλ/λ)^4 for higher order (e.g. log)
binning. Optionally, `bin_edges` can be specified directly. Exactly one of `wave` and
`bin_edges` must be specified. `flux` must also be specified and have the same length as
`wave` (or a length of 1 less than `bin_edges`).

The unit on the wavelength elements can be specified with `wave_unit`, and is assumed to
be angstroms by default. The flux is assumed to be a spectral flux density fλ in units
of erg/s/cm^2/A unless otherwise specified by `unit`. Internally, the wavelength is
stored in angstroms and the flux as fλ.

Uncertainties can optionally be specified, either through `fluxerr` which is a list of
uncorrelated uncertainties for each spectral element or through `fluxcov` which provides
the full covariance matrix. Uncertainties are assumed to be in the same units as the
flux.

Optionally, the `time` of the spectrum can be specified. This must be specified for
fitting purposes, but isn't necessary for things like synthetic photometry.

Regardless of how the `Spectrum` was constructed, the wavelength attributes `wave`,
`bin_edges`, `bin_starts`, and `bin_ends` are all available. `fluxerr` and `fluxcov` are
available if either was originally specified with conversion happening automatically
internally.


### Synthetic photometry

Synthetic photometry can be calculated on the spectrum using the `bandflux` method:

```python
def bandflux(self, band, zp=None, zpsys=None):
```

where `band` is an `sncosmo.Bandpass` object, the name of a bandpass in the registry, or
a list of such objects. By default, this returns the total flux in photons/s/cm^2. If zp
and zpsys are given, the flux(es) are scaled to the requested zeropoints.

`bandfluxcov` has the same signature as `bandflux`, but it also returns the covariance
matrix between the observed fluxes (this copies how this is handled for the `Model`
class).

Magnitudes can be computed with `bandmag`:

```python
def bandmag(self, band, magsys):
```

This is similar to `bandflux` except that magsys is required and the final result is
converted to magnitudes.


### Fitting models

With this EP, models can be fit to any combination of photometry and spectra. We propose
modifying `fit_lc` and adding an optional `spectra` keyword that takes a `Spectrum`
object or a list of `Spectrum` objects. Any combination of spectra and photometry is
allowed, including fitting only spectra or fitting a mix of photometry and spectra
simultaneously. Here are some examples of valid calls to `fit_lc`:

```python
fit_lc(photometry, model)
fit_lc(photometry, model, spectra=[spec1, spec2])
fit_lc(model=model, spectra=[spec1, spec2])
fit_lc(model=model, spectra=spec)
```

Internally, we will modify the `generate_chisq` function used by
`fit_lc` to compute the chi-square individually for all of the different photometry and
spectra and then combine the results. This approach means that the API will remain
unchanged, but there is a new option to include spectra as part of the fit.

## Backward compatibility

The old `Spectrum` class will be renamed to `SpectrumModel`, and all internal references
to that class will be updated. The new `Spectrum` class that replaces it will implement
the same API, so all code that used this class should continue to work as is. Speaking
with users, the main use case for this class was synthetic photometry. There will be
some changes in the new synthetic photometry because of how we treat the spectrum
(before: interpolating with arbitrary sampling, now: treating each spectral element as a
tophat filter with proper sampling.) These changes will improve the quality of the
photometry, especially for low-resolution spectra.


## Alternatives

The main alternative would be to use the `specutils.Spectrum1D` class directly instead
of implementing a new one. Unfortunately, this class does not yet support several
necessary features such as covariance between different spectral elements (see
https://github.com/astropy/specutils/issues/684 for a discussion of this). If these
features are implemented in `specutils`, then we could transition this class to be based
off of `specutils.Spectrum1D` internally in the future. We do not plan on implementing a
large amount of functionality to manipulate spectra in `sncosmo`.

Another alternative would be to use the `PhotometricData` class to handle spectra. A
spectrum can be interpreted as observations in a large number of small filters, and each
flux observation could then be implemented as an individual photometric measurement.
This would work with the current implementation of `sncosmo` as is, but a lot of the
properties of spectra are lost since the measurements are mixed in with all of the other
observations of that supernova. This would make it very difficult to implement things
like synthetic photometry that require knowledge of the structure of the spectrum. It
would also be very computationally inefficient.


## Decision rationale

<!-- To be filled in when the proposal is accepted or rejected -->
