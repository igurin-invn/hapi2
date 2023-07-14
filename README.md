# HAPI2: Second Generation of the HITRAN Application Programming Interface

The first generation of the HITRAN Application Programming Interface (HAPI) [[Kochanov et al. JQSRT 2016]](https://doi.org/10.1016/j.jqsrt.2016.03.005) has proven to be a convenient tool for acquiring and working with HITRAN data. 
The HAPI library provided a means of downloading and filtering the spectroscopic transitions
for molecules provided by the HITRANonline [[Hill et al. JQSRT 2016]](http://dx.doi.org/10.1016/j.jqsrt.2015.12.012) web server, using
a range of partition sums and spectral line parameters. 
A significant feature of HAPI was the ability to calculate absorption coefficients based on the line-by-line spectroscopic parameters. 
For a more detailed description of this software library, we refer readers
to the dedicated paper and corresponding user manual available online (https://hitran.org/hapi).

Although the first generation of HAPI allows users to build
new functions, it does not have the functionality to make use the
whole range of spectroscopic data currently available in the HITRAN database. 
For instance, the first version of the REST-API used by HAPI only allowed line-by-line data to be downloaded.<br>
For this reason, an extended version of HAPI (with greater functionality) is provided as part of HITRAN2020 database 
([Gordon et al. JQSRT 2022](https://doi.org/10.1016/j.jqsrt.2021.107949), https://hitran.org). 
This extended version, named “HAPI2”, includes all the functionality of HAPI but with a new
Python library and has been designed to be backward-compatible.
To take advantage of the more advanced features in the “second-generation” extension library, users will be required to upgrade to HAPI2.

One main feature of HAPI2 will be the ability to consider more objects available for downloading. 
This essentially means users will now be able to access the vast library of absorption cross sections, CIA, and more. 
This was achieved by revisiting the HITRAN server’s REST API. 
A new version is able to access the information for a number of entities available in HITRAN. 
Among these entities are molecule information, reference sources, line-by-line transitions, monomer and collision-induced absorption crosssections, and metadata on line parameters.

Secondly, for applications that require numerous transitions to be considered in absorption coefficient calculations (such as at high temperatures), the speed of calculation is of paramount importance. 
Although the first generation of HAPI contained some Numpy-based optimizations, it lacked the means for fast cross-section computation. 
In HAPI2, efficient coding for HT and SDV profiles that makes use of the “Just-in-time”
compilation approach, has provided a significant speed increase for the spectral simulation.

HAPI2 local database structure relies on the [SQLAlchemy](https://www.sqlalchemy.org/) interface to a number of the relational database management systems (RDBMS) which permit high-throughput analytics on a large number of spectral transitions. 
SQLALchemy provides object-relational mapping for main parts of HITRAN.

Last, but not least, HAPI2 has a modular structure, making it relatively easy to add custom plugins with new functionality (e.g. adding new LBL calculation plugins, new SQLAlchemy backends etc...)

# Installation

## Pulling from Github

1. `git clone https://github.com/hitranonline/hapi2.git`
2. `cd hapi2`
3. `pip install .`

## From Python Package index

*To be done*

## From Docker Hub

*To be done*

# Quick start tutorial

*This is a short tutorial on how to use basic functions of the HAPI2 library. More advanced tutorials will be added in the future.*

First, let's do some imports.

```python
>>> from getpass import getpass
>>> from hapi2 import *
HAPI2 version:  0.1
Updated SETTINGS_DEFAULT by local config file (C:\work\Activities\HAPI\HAPI2\git-repo\hapi2-all\hapi2\showcase\config.json).
Database name: local
Database engine: sqlite
Database path: ./
Web API TODO: include fetch_info() into init
jeanny, Ver.3.0
```

## Getting the API key

Most of HAPI2's main features are controllable through the config JSON file named `config.json`. 
To work properly, this file should always be in the current working directory. The settings include information about the local database, debugging flags, REST API settings, and the API key.

```python
>>> with open('config.json') as f:
>>>     print(f.read())
{
   "engine": "sqlite",
   "database": "local",
   "user": "root",
   "pass": null,
   "database_dir": "./",
   "echo": false,
   "debug": false,
   "display_fetch_url": false,
   "proxy": null,
   "host": "https://hitran.org",
   "api_version": "v2",
   "tmpdir": "~tmp",
   "api_key": ""
}
```

Unlike the previous version of the HITRAN API, the second generation of the API demands registration at the HITRANonline web site in order to get the API key.<br>
The API key is essential for HAPI2 to connect to the HITRANonline web site and retrieve the data. 
This key permits accessing the new functions of the HITRAN API v2 such as fetching the molecules, sources, cross-sections etc.<br>
It can be obtained from the HITRANonline user profile web page: https://hitran.org/profile/.<br>
To be able to go through the rest of this tutorial, go to the link above and paste your API key to the form below.

```python
>>> SETTINGS['api_key'] = getpass('Enter valid API key:')
>>> fetch_info()
Enter valid API key:········

Header for info is fetched from https://hitran.org

BEGIN DOWNLOAD: info
  1048576 bytes written to ~tmp\af306ba0-adcd-4ea1-8430-1e7b1aa2d84f.json
END DOWNLOAD
PROCESSED

{'status': 'OK',
 'message': '',
 'content': {'class': 'Info',
  'format': 'json',
  'data': {'static_root': 'data',
   'xsec_dir': 'data/xsec',
   'results_dir': 'results',
   'hapi_latest_version': None}},
 'timestamp': '2022-06-08 15:45:58.400858',
 'source': 'HITRANonline'}
```

To simplify the authentication process, one can store the API key in the config.json configuration file.

***Please note that users should keep the API key private. 
To regenerate the compromised API key, go to the HITRANonline [user profile](https://hitran.org/profile/) page and press the `Generate API key` button.***

Now, let's download some molecular data necessary for the tutorial.

## Molecules

The new version of the HITRAN API lets users download infomation not only on spectral lines, but also on molecules, isotopic species, publications, experimental cross-sections, partition sums, and collision induced absorption. 
Each such entity corresponds to the particular section of the API schema.<br>
Fetching such objects is done using the "fetch" functions of the API. 
Let's fetch the information on the molecules in the current version of the HITRAN database. 
HAPI2 will automatically create the Python objects for each of the molecules and save them to the local database.

```python
>>> mols = fetch_molecules() # fetch molecules
Header for molecules is fetched from https://hitran.org

BEGIN DOWNLOAD: molecules
  1048576 bytes written to ~tmp\c10d4283-b2ce-4c71-bdaa-619a042aada8.json
END DOWNLOAD
PROCESSED
```

By default, all data is passed in the JSON format. To discover the fields of the JSON records, use the `.dump()` method:

```python
>>> Molecule('Water').dump()
{'id': 1,
 'common_name': 'Water',
 'ordinary_formula': 'H2O',
 'ordinary_formula_html': 'H<sub>2</sub>O',
 'stoichiometric_formula': 'H2O',
 'inchi': 'InChI=1S/H2O/h1H2',
 'inchikey': 'XLYOFNOQVPJJNP-UHFFFAOYSA-N',
 '__class__': 'Molecule',
 '__identity__': 'common_name'}
```

Each of the fields can be accessed directly as a Python object attribute:

```python
>>> mol = Molecule('Water')
>>> print(mol.common_name, mol.ordinary_formula)
Water H2O
```

To search for the molecule in the local database, use one of its names (or "aliases"). 
To see which aliases are attached to the current molecule, use the `.aliases` attribute:

```python
>>> Molecule('Water').aliases
[7732-18-5,
 water vapor,
 Distilled water,
 XLYOFNOQVPJJNP-UHFFFAOYSA-N,
 NSC 147337,
 R-718,
 InChI=1S/H2O/h1H2,
 Dihydrogen monoxide,
 Dihydrogen oxide,
 HITRAN-mol-1,
 H2O,
 Water,
 R718,
 R 718,
 Hydrogen oxide (H2O)]
```

Using any of the aliases given above, you can find the very same molecule. 
For example, we will use the InChIKey this time to find the Water molecule:

```python
>>> Molecule('XLYOFNOQVPJJNP-UHFFFAOYSA-N') is Molecule('Water')
True
```

This greatly simplifies the molecule search in the local database. 
Also note that case doesn't matter:

```python
>>> Molecule('water') is Molecule('Water')
True
```

## Isotopologues

Each molecule has a number of isotopic species, which can also be fetched from the HITRAN website. 
In order to do that, one should already have the molecules fetched. 
Let's download all isotopologues for the Water molecule.

```python
>>> # Fetch isotopologues for Water molecule. 
>>> isos = fetch_isotopologues([
>>>     Molecule('Water'),
>>> ])
Header for isotopologues is fetched from https://hitran.org

BEGIN DOWNLOAD: isotopologues
  1048576 bytes written to ~tmp\e2683a16-c207-493a-8c1b-d142d175905d.json
END DOWNLOAD
PROCESSED
```

Just as for the molecules and any other API objects, isotopologues are constructed from JSON records:

```python
>>> iso = isos[0] # for example, take the first isotopologue in the list (not necessary this will be the principal one!)
>>> iso.dump()
{'id': 1,
 'molecule_alias_id': 4864,
 'isoid': 1,
 'inchi': 'InChI=1S/H2O/h1H2',
 'inchikey': 'XLYOFNOQVPJJNP-UHFFFAOYSA-N',
 'iso_name': 'H2(16O)',
 'iso_name_html': 'H<sub>2</sub><sup>16</sup>O',
 'abundance': 0.997317,
 'mass': 18.010565,
 'afgl_code': '161',
 '__class__': 'Isotopologue',
 '__identity__': 'iso_name'}
```

Like molecules, isotopologues are fetched with their aliases. 
To search for the isotopologue in the local database, use one of these aliases:

```python
>>> iso.aliases
[InChI=1S/H2O/h1H2,
 XLYOFNOQVPJJNP-UHFFFAOYSA-N,
 HITRAN-iso-1-1,
 H2(16O),
 HITRAN-iso-1]
>>> Isotopologue('H2(16O)') is iso
True
```

## Sources

Now, let's fetch the sources (notes, publications, etc). 
They can be fetched in the very same way as the molecules:

```python
>>> srcs = fetch_sources()
Header for sources is fetched from https://hitran.org

BEGIN DOWNLOAD: sources
  1048576 bytes written to ~tmp\cebb6728-de14-4009-a958-15d4b42c0b15.json
  1048576 bytes written to ~tmp\cebb6728-de14-4009-a958-15d4b42c0b15.json
END DOWNLOAD
PROCESSED
>>> src = srcs[0]
>>> src.dump()
{'id': 1,
 'type': 'article',
 'authors': 'L.S. Rothman, R.R. Gamache, A. Goldman, L.R. Brown, R.A. Toth, H.M. Pickett, R.L. Poynter, J.-M. Flaud, C. Camy-Peyret, A. Barbe, N. Husson, C.P. Rinsland, M.A.H. Smith',
 'title': 'The HITRAN database: 1986 edition',
 'journal': 'Applied Optics',
 'volume': '26',
 'page_start': '4058',
 'page_end': '4097',
 'year': 1987,
 'institution': '',
 'note': '',
 'doi': '10.1364/AO.26.004058',
 'bibcode': '1987ApOpt..26.4058R',
 'url': 'https://doi.org/10.1364/AO.26.004058',
 'short_alias': 'HITRAN-src-1',
 '__class__': 'Source',
 '__identity__': 'id'}
```

The Source objects have the special `citation` appearance mode. This can be helpful if one wants to cite the references from HITRAN:

```python
>>> src.citation
'L.S. Rothman, R.R. Gamache, A. Goldman, L.R. Brown, R.A. Toth, H.M. Pickett, R.L. Poynter, J.-M. Flaud, C. Camy-Peyret, A. Barbe, N. Husson, C.P. Rinsland, M.A.H. Smith. The HITRAN database: 1986 edition. Applied Optics 1987;26:4058-4097. doi:10.1364/AO.26.004058. '
```

## Transitions

One the isotopologues have been fetched, we can download the transitions for them. 
For this purpose, we will use the `fetch_transitions` function. 
Note that the first parameter can be either a single isotopologue or a list of isotopologues.

```python
>>> # Fetch transitions for water isotopologues.
>>> fetch_transitions(Molecule('water').isotopologues, 2000, 2100, 'h2o')
>>> transs = Molecule('water').transitions
Header for transitions is fetched from https://hitran.org

BEGIN DOWNLOAD: transitions
  1048576 bytes written to ~tmp\f7b18d96-7205-4674-abe7-605803747062.json
END DOWNLOAD
PROCESSED

File 62a0c463.out is fetched from results

BEGIN DOWNLOAD: 62a0c463.out
  67108864 bytes written to ~tmp\h2o.data
END DOWNLOAD
PROCESSED
saving HAPI header h2o.header to ~tmp
==================================
Total lines processed: 1420
==================================
```
    
By default, each transition has a standard HITRAN 160-character representation (even if it has additional non-standard parameters, for instance, foreign broadening coefficients):

```python
>>> transs[:10]
[ 11 2000.395383 9.745E-29 7.551E-01.03250.281 4265.97640.37-.005240          0 2 0          0 1 0 11  9  2       11  8  3      434233807294713152    69.0   69.0,
  11 2000.397458 3.248E-29 7.550E-01.03250.281 4265.97420.37-.005240          0 2 0          0 1 0 11  9  3       11  8  4      434233807294713152    23.0   23.0,
  11 2000.411407 1.980E-30 6.537E-09.06860.333  982.91170.620.000000          0 1 0          0 0 0  9  4  6        8  2  6     q5322308079917122 0    19.0   17.0,
  11 2000.416249 2.606E-30 7.489E-06.07460.371 2552.85730.66-.009310          0 0 1          0 1 0  7  3  5        6  5  2      534446807294713152    45.0   39.0,
  12 2000.783338 4.967E-28 4.632E+00.06700.354 2758.95990.91-.005980          0 2 0          0 1 0  9  5  5        8  4  4      546546807233162752    19.0   17.0,
  13 2000.795416 2.630E-28 4.043E-01.08260.430 2156.47120.96-.009750          0 0 1          0 1 0  5  2  4        6  2  5      563334803633162754   198.0  234.0,
  11 2000.903221 3.567E-24 5.799E+00.04090.229 2406.14310.52-.009525          0 2 0          0 1 0  6  6  1        5  5  0      587777305910162545    39.0   33.0,
  12 2000.934664 3.282E-29 9.668E-03.08440.391 2120.45450.71-.005560          1 0 0          0 1 0  4  4  1        5  3  2      534456807294713152    27.0   33.0,
  11 2001.003293 6.940E-30 2.695E+00.05150.352 4782.92000.50-.004950          0 1 1          0 0 1  8  6  3        7  5  2      532230807294713152    17.0   15.0,
  11 2001.016859 1.330E-24 6.488E+00.03610.219 2406.14090.74-.010037          0 2 0          0 1 0  6  6  0        5  5  1      587777805910162545    13.0   11.0]
```

For more details on the Transition object parameters, use the `dump()` method. 
All transition parameter names correspond to those used in the first version of HAPI (https://hitran.org/hapi):

```python
>>> trans = transs[0]
>>> trans.dump()
{'id': 11001228,
 'isotopologue_alias_id': 162,
 'molec_id': 1,
 'local_iso_id': 1,
 'nu': 2000.395383,
 'sw': 9.745e-29,
 'a': 0.7551,
 'gamma_air': 0.0325,
 'gamma_self': 0.281,
 'elower': 4265.9764,
 'n_air': 0.37,
 'delta_air': -0.00524,
 'global_upper_quanta': '          0 2 0',
 'global_lower_quanta': '          0 1 0',
 'local_upper_quanta': ' 11  9  2      ',
 'local_lower_quanta': ' 11  8  3      ',
 'ierr': '434233',
 'iref': '807294713152',
 'line_mixing_flag': ' ',
 'gp': 69,
 'gpp': 69,
 'extra': None,
 '__class__': 'Transition',
 '__identity__': 'id'}
```

## Parameter metas

All metadata on the transition parameters are stored in the ParameterMeta objects, which can also be fetched:

```python
>>> pmetas = fetch_parameter_metas() # fetch transition parameter descriptions
Header for parameter-metas is fetched from https://hitran.org

BEGIN DOWNLOAD: parameter-metas
  1048576 bytes written to ~tmp\1958be3a-5969-49db-9328-002d6075018c.json
END DOWNLOAD
PROCESSED
>>> pmetas[:10]
[global_iso_id,
 molec_id,
 local_iso_id,
 nu,
 sw,
 a,
 gamma_air,
 gamma_self,
 n_air,
 delta_air]
>>> pmetas[0].dump()
{'id': 1,
 'name': 'global_iso_id',
 'type': 'int',
 'description': 'Unique integer ID of a particular isotopologue: every global isotopologue ID is unique to a particular species, even between different molecules. The number itself is, however arbitrary.',
 'format': '%5d',
 'units': '',
 '__class__': 'ParameterMeta',
 '__identity__': 'name'}
```

## Relations between objects

Since HAPI2 uses object-relational mapping, it enables the access to the relational structure of the local database by means of the SQLAlchemy ORM language. 
Let's demonstrate it by starting with a transition.

Let's say we have a transition `trans`:

```python
>>> trans
 11 2000.395383 9.745E-29 7.551E-01.03250.281 4265.97640.37-.005240          0 2 0          0 1 0 11  9  2       11  8  3      434233807294713152    69.0   69.0
```

To get access to the isotopologue object attached to this transition, we will use the SQLAlchem relationship called by `trans.isotopologue`:

```python
>>> trans.isotopologue
H2(16O)
```

Now, for going further and getting the molecule for this isotopologue, we'll use the `molecule` relationship:

```python
>>> trans.isotopologue.molecule
Water
```

HAPI2 also provides a shortcut for transitions, allowing to get the corresponding molecule right away:

```python
>>> trans.molecule
Water
```

## Caching objects in the local HAPI2 database

Once downloaded ("fetched"), the objects live in the local database specified in `config.json`. 
Usually, if the default SQLite database backend is used, is is stored in the "local" file in the working directory.

*Note that there is no need to re-fetch the object each time your HAPI2-powered script is run.*

To search for the object in the local database, use the object constructor/initializer as follows:

```python
>>> Molecule('water') # this will search the molecule with the "water" alias attached
Water
>>> Molecule('h2o') # this will return the same molecule, but now the different alias was used
Water
>>> Isotopologue('H2(16O)') # HITRAN isotopologue notation
H2(16O)
>>> Isotopologue('XLYOFNOQVPJJNP-UHFFFAOYSA-N') # InChIKey
H2(16O)
```

## The "HITRAN-specific" object aliases

Besides the "common" aliases for objects like chemical formulas, HAPI2 assigns each object a HITRAN-secific alias.<br>
These aliases have similar structures that read *HITRAN-obj-ID*, where `obj` is the acronym of the object type (`mol`, `src`, `iso`, etc...), and `ID` is the HITRANonline "global" ID number. 

For instance, for the molecule with number 6 in the HITRAN rank (i.e. methane), this alias will read as "HITRAN-mol-6":

```python
>>> Molecule('HITRAN-mol-6')
Methane
```

For isotopologues, there are two ways of specifying the HITRAN aliases. 
The first way is using the "global" isotopologue ID number, used in HITRANonline. 
In the following example, we will search for the isotopologue with the global ID = 1:

```python
>>> Isotopologue('HITRAN-iso-1')
H2(16O)
```

The second way is using more traditional HITRAN isotopologue identification, where the number of molecule along with the "local" isotopologue ID are used 
(these numbers are the first two in the 160-character HITRAN text format):

```python
>>> Isotopologue('HITRAN-iso-1-1')
H2(16O)
```

For sources, the HITRANonline id can be used in the same way, as for the isotopologue and molecule:

```python
>>> Source('HITRAN-src-1').citation
'L.S. Rothman, R.R. Gamache, A. Goldman, L.R. Brown, R.A. Toth, H.M. Pickett, R.L. Poynter, J.-M. Flaud, C. Camy-Peyret, A. Barbe, N. Husson, C.P. Rinsland, M.A.H. Smith. The HITRAN database: 1986 edition. Applied Optics 1987;26:4058-4097. doi:10.1364/AO.26.004058. '
```

## Conclusion

This tutorial covers very basic features of HAPI2, such as fetching of objects, searching the local database, and using relationships. 
For more advanced topics (using custom database backends, efficient working with large amount of transitions, spectra simulation presets, tracking data provenance with HAPI2 etc...), separate dedicated tutorials will be created.
