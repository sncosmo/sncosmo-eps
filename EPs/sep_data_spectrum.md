# Add functionality for spectrum data


author: Kyle Boone <!-- your name -->

date-created: 2020 May 18 <!-- replace with the date you submit the APE -->

date-last-revised: 2020 May 18 <!-- keep this up to date anytime something changes -->

date-accepted: <!-- replace with accepted date -->

status: Discussion  <!-- one of: Discussion, Accepted, Rejected -->


## Abstract

The goal of this proposal is to add functionality to sncosmo to handle spectrum data. We
propose to add a SpectrumData class that represents a spectra taken of a supernova, and
implement functionality to work with this data.


## Detailed description

Sncosmo is currently limited to working with photometric data. However, spectra are
often obtained of supernovae, and it would be very useful if sncosmo could use this
additional data. We propose to add functionality to sncosmo to work with spectra.

There are several ways in which spectra can be used for supernova cosmology. First,
spectra can provide additional constraints when fitting models to observations of a
supernova. Models such as the SNEMO model that was recently added to sncosmo are
designed to be able to fit individual spectra of each supernova rather than being
limited to photometry. Surveys such as the SNfactory are obtaining spectrophotometric
timeseries of supernovae that can then be fit directly with these models, potentially
providing additional information beyond what is captured by photometry alone. 

Given a spectrophotometric timeseries, it is possible to generate synthetic photometry
in arbitrary bands. This photometry has many uses, such as comparing between experiments
or using light curve fitters that take photometry as input. Sncosmo already provides
functionality to generate photometry from spectral time series models, and this same
functionality could be applied to spectrophotometric spectra.

Less-well-calibrated spectra can also be useful for typing supernovae or constraining
models. However, they often require some warping of the spectrum or removal of host
galaxy contamination. This functionality is beyond the scope of this EP, but we would
like to set up a framework to be able to include it in the future.


## Branches and pull requests

An initial discussion of this work can be found in sncosmo/sncosmo#262.

sncosmo/sncosmo#265 has an implementation of fitting models to spectral time series.

sncosmo/sncosmo#259 implements basic synthesized photometry.


## Implementation

The building block for all of this work is adding a SpectrumData class to sncosmo to
handle spectrum data. There is currently a Spectrum class that is designed to
represent a model of a spectrum. The SpectrumData class should be different as it
represents binned observations of a spectrum rather than a continuous model. This is
similar to the distinction between the SourceModel and PhotData classes that currently
exist.

The SpectrumData class needs to encapsulate several aspects of an observed spectrum. An
observed spectrum can be thought of as a set of many small filters that are all adjacent
to each other. The class needs to keep track of the edges of these filters and the bin
widths. Spectra also often come with uncertainties or covariance matrices, and these
should be properly encapsulated. It is also important to track the zeropoint of the
spectrum if it is photometrically calibrated, but this isn't always the case. This
implementation will be inspired by the PhotData class that deals with similar issues,
and an initial implementation can be found in sncosmo/sncosmo#265.

With this architecture in place, we can build out the desired functionality in parallel.
For fitting purposes, we propose to modify the chisq function that is currently used to
handle both photometric data and spectrum data. Such a universal chisq function can then
be used universally by all of the different fitters implemented in sncosmo. An
implementation of this for spectrophotometric timeseries can be found in
sncosmo/sncosmo#265.

A separate step which doesn't depend on the fitting functionality is adding methods to
transform spectra. This includes procedures such as generating synthetic photometry,
rebinning, warping, or redshifting the spectrum. All of these can be implemented in
parallel and don't explicitly depend on each other. An initial effort at implementing
synthetic photometry can be found in sncosmo/sncosmo#259.


## Backward compatibility

No issues (yet).


## Alternatives

The main alternative would be to use the PhotData class to handle spectra. A spectrum
can be interpreted as observations in a large number of small filters, and each flux
observation could then be implemented as an individual photometric measurement. This
would work with the current implementation of sncosmo as is, but a lot of the properties
of spectra are lost since the measurements are mixed in with all of the other
observations of that supernova. This would make it very difficult to implement things
like synthetic photometry that require knowledge of the structure of the spectrum. It
would also be very computationally inefficient.


## Decision rationale

<!-- To be filled in when the proposal is accepted or rejected -->
