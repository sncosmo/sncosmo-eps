# Improving simulations in sncosmo


author: Kyle Boone <!-- your name -->

date-created: 2020 June 9 <!-- replace with the date you submit the APE -->

date-last-revised: 2020 August 11 <!-- keep this up to date anytime something changes -->

date-accepted: <!-- replace with accepted date -->

status: Discussion  <!-- one of: Discussion, Accepted, Rejected -->


## Abstract

This proposal adds tools to `sncosmo` that can be used to generate simulations of
transient surveys. This simulation can include a wide range of different transient
models, each having its own redshift, spatial and parameter distributions. Ultimately,
the simulation produces a catalog of all of the transients in the survey field, and
each entry in the catalog is an `sncosmo.Model` object with appropriately simulated
parameters. We also provide tools that can be used to generate light curves from these
catalogs given the specific observing cadence of a survey.

## Detailed description

`sncosmo` contains models of many different transients, but it does not provide an easy
method of using those transient models to simulate surveys. This is one of the main
features of `SNANA` that would be very useful to have in python. The goal of this
proposal is to create a modular framework that can be used to simulate transient
surveys. The framework should not be tied to specific surveys, but should provide tools
to make it easy to interface with different surveys' tools. This framework consists of
several different parts.

First, a field is defined that establishes the region of the sky and time range that
should be simulated. A field does not take into account observing efficiency or
cadence, it simply establishes the region of the sky that should be simulated. The
field should be large enough to include all transients that could possibly be
observed by the survey, but small enough to avoid simulating many transients that won't
be observed. A simulation can be broken into many small non-overlapping fields to
parallelize computations.

The next piece of the simulation is to define the different transients that will be
simulated. `sncosmo.Model` objects can be simulated with arbitrary distributions of
redshifts, positions on the sky, or other parameters. Parameters can have distributions
that are redshift-dependent (e.g. the distribution of SALT2 x1 varying with redshift).

Given a set of transient distributions and a field, we can simulate a catalog of
transients. This catalog is effectively a list of `~sncosmo.Model` objects whose
parameters have been simulated according to the specified distributions. This catalog
can either be directly representative (i.e. a list of all of the objects that can will
be observed) or weighted (e.g. simulating equal numbers of each object type, equal
numbers in each redshift bin). Individual objects can also be simulated. A catalog is
sufficient on its own for many use cases. For example, it can be used directly to
calculate the number of transients above a specific brightness threshold that will be
in the field of a survey.

The catalog can also be turned into a set of light curves given an observing cadence
for the survey. The observing cadence will be survey specific, and generating it is
outside of the scope of `sncosmo`. There are several ways to couple an observing cadence
to the simulations:
- The simulations can all be evaluated using the same observing cadence.
- The observing cadence is the same for small regions of the sky. This is the case for
  many large astonomical surveys that tile the sky and repeat the same telescope
  pointings.
- Each pointing has its own observing cadence, and we have some external tool that can
  provide the cadence for a given RA and Dec. This is the case for LSST which has an
  observing database. 
In any case, the user will query some external system using the list of RA, Dec and time
values from the catalog and retrieve either a list of observing cadences or a single
observing cadence. The `~sncosmo.realize_lcs` function can already handle generating
light curves when there is a single observing cadence and a single transient type. We
propose to modify that function to handle situations where there are multiple cadences
or multiple transient types.

A single catalog can be used to simulate surveys with multiple different observing
cadences. This can be useful for comparing different proposed cadences for a survey.

## Branches and pull requests

TODO

This work draws from Rahul Biswas's work on LSST simulations that can be found at
https://github.com/rbiswas4/SNsims/blob/master/snsims/homogeneousSNUniverse.py

## Implementation

TODO

## Backward compatibility

`~sncosmo.realize_lcs` will be updated. It should be possible to make these changes
backwards compatible.

The `zdist` function in `simulation.py` will likely be irrelevant after these changes
are made and should probably be deprecated.

## Alternatives

TODO
- SNANA (not python)
- Rahul's simulation package (tied to LSST, and not as flexible as I would like)

## Decision rationale

<!-- To be filled in when the proposal is accepted or rejected -->
