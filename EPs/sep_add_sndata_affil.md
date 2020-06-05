# Add SNData as Affiliate Package


author: Daniel Perrefort

date-created: 2020 May 19

date-last-revised:  2020 May 19

date-accepted: 2020 Jun 04

status: Accepted 



## Abstract

SNData provides access to data releases published by a variety of supernova (SN) surveys. It is designed to support the development of scalable analysis pipelines that translate with minimal effort between and across data sets. This SEP proposes to make SNData an SNCosmo affiliate package and is motivated by previous work undertaken by Kyle Barbary on the `sndatasets` package.




## Detailed description

A few years ago work was undertaken to build an `sncosmo` affiliated data access package called `sndatasets` ([source](https://github.com/sncosmo/sndatasets)). Work on this project has stalled and been abandoned. More recently, the `sndata` package was developed for internal use in ongoing projects at the University of Pittsburgh. This new package has been made public and recently made a v1.0 release. We here proposes to depricate `sndatasets`  and adopt  `sndata` as an official affiliate package. This would make it easier for users to get new projects up and running with `sncosmo` on real data. It would also provide a standardized but optional design for combining  `sncosmo` and data access in third pary analysis pipelines.



### Package description

A minimal package description is provided here. Detailed questions should refer to existing documentation on [Read The Docs](https://sn-data.readthedocs.io/en/latest/). Full examples are available in the [quick](https://sn-data.readthedocs.io/en/latest/getting_started/quick_start.html) and [slow](https://sn-data.readthedocs.io/en/latest/getting_started/slow_start.html) starts.



`sndata` provides access to photometric and spectroscopic data using the following base classes:

- `SpectroscopicRelease()` for parsing spectroscopic data
- `PhotometricRelease(SpectroscopicRelease)` for parsing photometric data

There are some differences between these classes, but they both support the same key functionality for downloading, parsing, and retrieving data. For example:

```python
# All data releases are available as ``sndata.<survey name>.<release name>``
from sndata.csp import DR3
dr3 = DR3()  # Locates data on the local machine

# Download data to your machine.
# Note: If data is already downloaded, this function call won't do anything.
dr3.download_module_data()

# Check what tables are available from the release's corresponding publication
# and read one of those tables by referencing the table name or number
published_tables = dr3.get_available_tables()
dr3_demo_table = dr3.load_table(published_tables[0])

# Check what objects are included in the data release
# and read in the data for an object using it's Id
object_ids = dr3.get_available_ids()  # A list
demo_id = object_ids[0]
dr3.get_data_for_id(demo_id)  # An astropy table

# Data is auto-formatted to be compatible with the SNCosmo package.
# To disable this and return data as presented on the data release,
# set ``format_table=False``.
dr3.get_data_for_id(demo_id, format_table=False)

```

Multiple data sets can also be combined into a single data access instance while maintaning the same data access UI demonstrated above (see more [here](https://sn-data.readthedocs.io/en/latest/getting_started/combining_datasets.html)):

```python
 from sndata import CombinedDataset, csp, des

 combined_data = CombinedDataset(csp.DR3(), des.SN3YR())
```






## Branches and pull requests

Existing source code for SNData can be found [on GitHub](https://github.com/mwvgroup/sndata)

Existing documentation can be found on [Read The Docs](https://sn-data.readthedocs.io/en/latest/)



## Implementation

SNData is already on a 1.n release. Changes to make it an affiliated package are minimal:

- Move SNData into the sncosmo GitHub organization
- Officially depricate the `sndatasets` project
- Add some additional compatibility tests (this is mostly done)
- Use SNData to handle the loading of `sncosmo` example data
- Various doc updates listing explicit support between the packages.
- **optional:** SNData downloads data from the servers / websites of individual surveys. If we want to make this an affiliate package, thought should be given to whether we want to use a centralized server (e.g. zenodo) to provide improved uptime.
- **optional:** Work is ongoing to add the LOSS, BSNIP, and Sweetspot surveys to SNData. It would be nice to resume / finish adding these data sets as part of this SEP.




## Backward compatibility

SNData is **NOT** compatibale with Python2.




## Alternatives

The Open Supernova Cataloge (OSC) provides public access to SN data from a variety of surveys. However, it provides limited utilities for accessing or parsing the data beyond a RESTFUL API. The OSC also does not include transmission curves or survey data (e.g. tables from the data release paper). This limits OSC's primary use to being a data aquisition tool and leaves users to build their tools for downloading and parsing data.




## Decision rationale

The SNData package fills the need that sndatasets was originally trying to address. Bringing it under the sncosmo organization will enable discovery and possibly make it easier for more people to contribute.  There were no objections raised.
