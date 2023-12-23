The Versioneer
==============

**Latest Version**:  [![Latest Version][pypi-image]][pypi-url]
[![Github Release][github-release-image]][github-release-url]

**Latest Version @ 3rd Party Packagers**: [![Latest Version Anaconda main][anaconda-org-main-image]][anaconda-org-main-url]
[![Latest Version Conda Forge][conda-forge-image]][conda-forge-url]

**Support Python Versions**: ![Python Versions][python-versions-img]

**Downloads**: ![Pypi Downloads][pypi-downloads-img]
![Conda Downloads Anaconda Main][downloads-conda-main-img]
![Conda Downloads Anaconda Main][downloads-conda-forge-img]

**Community**: [![Matrix][matrix-image]][matrix-url]
[![Gitter.im][gitter-image]][gitter-url]
![Github Stars][github-stars-img]

**Build Status**: [![Github Workflow][github-workflow-tox-img]][github-workflow-tox-url]

**License**: (Public Domain) ![Unlicense][license-img]

* like a rocketeer, but for versions!
* https://github.com/python-versioneer/python-versioneer
* Brian Warner
* Compatible with: Python 3.8, 3.9, 3.10, 3.11 and pypy3
* Experimental support for Python 3.12.

This is a tool for managing a recorded version number in setuptools-based
python projects. The goal is to remove the tedious and error-prone "update
the embedded version string" step from your release process. Making a new
release should be as easy as recording a new tag in your version-control
system, and maybe making new tarballs.

## Quick Install

Versioneer provides two installation modes. The "classic" vendored mode installs
a copy of versioneer into your repository. The experimental build-time dependency mode
is intended to allow you to skip this step and simplify the process of upgrading.

### Vendored mode

* `pip install versioneer` to somewhere in your $PATH
   * A [conda-forge recipe](https://github.com/conda-forge/versioneer-feedstock) is
     available, so you can also use `conda install -c conda-forge versioneer`
* add a `[tool.versioneer]` section to your `pyproject.toml` or a
  `[versioneer]` section to your `setup.cfg` (see [Install](INSTALL.md))
   * Note that you will need to add `tomli; python_version < "3.11"` to your
     build-time dependencies if you use `pyproject.toml`
* run `versioneer install --vendor` in your source tree, commit the results
* verify version information with `python setup.py version`

### Build-time dependency mode

* `pip install versioneer` to somewhere in your $PATH
   * A [conda-forge recipe](https://github.com/conda-forge/versioneer-feedstock) is
     available, so you can also use `conda install -c conda-forge versioneer`
* add a `[tool.versioneer]` section to your `pyproject.toml` or a
  `[versioneer]` section to your `setup.cfg` (see [Install](INSTALL.md))
* add `versioneer` (with `[toml]` extra, if configuring in `pyproject.toml`)
  to the `requires` key of the `build-system` table in `pyproject.toml`:
  ```toml
  [build-system]
  requires = ["setuptools", "versioneer[toml]"]
  build-backend = "setuptools.build_meta"
  ```
* run `versioneer install --no-vendor` in your source tree, commit the results
* verify version information with `python setup.py version`

## Version Identifiers

Source trees come from a variety of places:

* a version-control system checkout (mostly used by developers)
* a nightly tarball, produced by build automation
* a snapshot tarball, produced by a web-based VCS browser, like github's
  "tarball from tag" feature
* a release tarball, produced by "setup.py sdist", distributed through PyPI

Within each source tree, the version identifier (either a string or a number,
this tool is format-agnostic) can come from a variety of places:

* ask the VCS tool itself, e.g. "git describe" (for checkouts), which knows
  about recent "tags" and an absolute revision-id
* the name of the directory into which the tarball was unpacked
* an expanded VCS keyword ($Id$, etc)
* a `_version.py` created by some earlier build step

For released software, the version identifier is closely related to a VCS
tag. Some projects use tag names that include more than just the version
string (e.g. "myproject-1.2" instead of just "1.2"), in which case the tool
needs to strip the tag prefix to extract the version identifier. For
unreleased software (between tags), the version identifier should provide
enough information to help developers recreate the same tree, while also
giving them an idea of roughly how old the tree is (after version 1.2, before
version 1.3). Many VCS systems can report a description that captures this,
for example `git describe --tags --dirty --always` reports things like
"0.7-1-g574ab98-dirty" to indicate that the checkout is one revision past the
0.7 tag, has a unique revision id of "574ab98", and is "dirty" (it has
uncommitted changes).

The version identifier is used for multiple purposes:

* to allow the module to self-identify its version: `myproject.__version__`
* to choose a name and prefix for a 'setup.py sdist' tarball

## Theory of Operation

Versioneer works by adding a special `_version.py` file into your source
tree, where your `__init__.py` can import it. This `_version.py` knows how to
dynamically ask the VCS tool for version information at import time.

`_version.py` also contains `$Revision$` markers, and the installation
process marks `_version.py` to have this marker rewritten with a tag name
during the `git archive` command. As a result, generated tarballs will
contain enough information to get the proper version.

To allow `setup.py` to compute a version too, a `versioneer.py` is added to
the top level of your source tree, next to `setup.py` and the `setup.cfg`
that configures it. This overrides several distutils/setuptools commands to
compute the version when invoked, and changes `setup.py build` and `setup.py
sdist` to replace `_version.py` with a small static file that contains just
the generated version data.

## Installation

See [INSTALL.md](./INSTALL.md) for detailed installation instructions.

## Version-String Flavors

Code which uses Versioneer can learn about its version string at runtime by
importing `_version` from your main `__init__.py` file and running the
`get_versions()` function. From the "outside" (e.g. in `setup.py`), you can
import the top-level `versioneer.py` and run `get_versions()`.

Both functions return a dictionary with different flavors of version
information:

* `['version']`: A condensed version string, rendered using the selected
  style. This is the most commonly used value for the project's version
  string. The default "pep440" style yields strings like `0.11`,
  `0.11+2.g1076c97`, or `0.11+2.g1076c97.dirty`. See the "Styles" section
  below for alternative styles.

* `['full-revisionid']`: detailed revision identifier. For Git, this is the
  full SHA1 commit id, e.g. "1076c978a8d3cfc70f408fe5974aa6c092c949ac".

* `['date']`: Date and time of the latest `HEAD` commit. For Git, it is the
  commit date in ISO 8601 format. This will be None if the date is not
  available.

* `['dirty']`: a boolean, True if the tree has uncommitted changes. Note that
  this is only accurate if run in a VCS checkout, otherwise it is likely to
  be False or None

* `['error']`: if the version string could not be computed, this will be set
  to a string describing the problem, otherwise it will be None. It may be
  useful to throw an exception in setup.py if this is set, to avoid e.g.
  creating tarballs with a version string of "unknown".

Some variants are more useful than others. Including `full-revisionid` in a
bug report should allow developers to reconstruct the exact code being tested
(or indicate the presence of local changes that should be shared with the
developers). `version` is suitable for display in an "about" box or a CLI
`--version` output: it can be easily compared against release notes and lists
of bugs fixed in various releases.

The installer adds the following text to your `__init__.py` to place a basic
version in `YOURPROJECT.__version__`:

    from ._version import get_versions
    __version__ = get_versions()['version']
    del get_versions

## Styles

The setup.cfg `style=` configuration controls how the VCS information is
rendered into a version string.

The default style, "pep440", produces a PEP440-compliant string, equal to the
un-prefixed tag name for actual releases, and containing an additional "local
version" section with more detail for in-between builds. For Git, this is
TAG[+DISTANCE.gHEX[.dirty]] , using information from `git describe --tags
--dirty --always`. For example "0.11+2.g1076c97.dirty" indicates that the
tree is like the "1076c97" commit but has uncommitted changes (".dirty"), and
that this commit is two revisions ("+2") beyond the "0.11" tag. For released
software (exactly equal to a known tag), the identifier will only contain the
stripped tag, e.g. "0.11".

Other styles are available. See [details.md](details.md) in the Versioneer
source tree for descriptions.

## Debugging

Versioneer tries to avoid fatal errors: if something goes wrong, it will tend
to return a version of "0+unknown". To investigate the problem, run `setup.py
version`, which will run the version-lookup code in a verbose mode, and will
display the full contents of `get_versions()` (including the `error` string,
which may help identify what went wrong).

## Known Limitations

Some situations are known to cause problems for Versioneer. This details the
most significant ones. More can be found on Github
[issues page](https://github.com/python-versioneer/python-versioneer/issues).

### Subprojects

Versioneer has limited support for source trees in which `setup.py` is not in
the root directory (e.g. `setup.py` and `.git/` are *not* siblings). The are
two common reasons why `setup.py` might not be in the root:

* Source trees which contain multiple subprojects, such as
  [Buildbot](https://github.com/buildbot/buildbot), which contains both
  "master" and "slave" subprojects, each with their own `setup.py`,
  `setup.cfg`, and `tox.ini`. Projects like these produce multiple PyPI
  distributions (and upload multiple independently-installable tarballs).
* Source trees whose main purpose is to contain a C library, but which also
  provide bindings to Python (and perhaps other languages) in subdirectories.

Versioneer will look for `.git` in parent directories, and most operations
should get the right version string. However `pip` and `setuptools` have bugs
and implementation details which frequently cause `pip install .` from a
subproject directory to fail to find a correct version string (so it usually
defaults to `0+unknown`).

`pip install --editable .` should work correctly. `setup.py install` might
work too.

Pip-8.1.1 is known to have this problem, but hopefully it will get fixed in
some later version.

[Bug #38](https://github.com/python-versioneer/python-versioneer/issues/38) is tracking
this issue. The discussion in
[PR #61](https://github.com/python-versioneer/python-versioneer/pull/61) describes the
issue from the Versioneer side in more detail.
[pip PR#3176](https://github.com/pypa/pip/pull/3176) and
[pip PR#3615](https://github.com/pypa/pip/pull/3615) contain work to improve
pip to let Versioneer work correctly.

Versioneer-0.16 and earlier only looked for a `.git` directory next to the
`setup.cfg`, so subprojects were completely unsupported with those releases.

### Editable installs with setuptools <= 18.5

`setup.py develop` and `pip install --editable .` allow you to install a
project into a virtualenv once, then continue editing the source code (and
test) without re-installing after every change.

"Entry-point scripts" (`setup(entry_points={"console_scripts": ..})`) are a
convenient way to specify executable scripts that should be installed along
with the python package.

These both work as expected when using modern setuptools. When using
setuptools-18.5 or earlier, however, certain operations will cause
`pkg_resources.DistributionNotFound` errors when running the entrypoint
script, which must be resolved by re-installing the package. This happens
when the install happens with one version, then the egg_info data is
regenerated while a different version is checked out. Many setup.py commands
cause egg_info to be rebuilt (including `sdist`, `wheel`, and installing into
a different virtualenv), so this can be surprising.

[Bug #83](https://github.com/python-versioneer/python-versioneer/issues/83) describes
this one, but upgrading to a newer version of setuptools should probably
resolve it.


## Updating Versioneer

To upgrade your project to a new release of Versioneer, do the following:

* install the new Versioneer (`pip install -U versioneer` or equivalent)
* edit `setup.cfg` and `pyproject.toml`, if necessary,
  to include any new configuration settings indicated by the release notes.
  See [UPGRADING](./UPGRADING.md) for details.
* re-run `versioneer install --[no-]vendor` in your source tree, to replace
  `SRC/_version.py`
* commit any changed files

## Future Directions

This tool is designed to make it easily extended to other version-control
systems: all VCS-specific components are in separate directories like
src/git/ . The top-level `versioneer.py` script is assembled from these
components by running make-versioneer.py . In the future, make-versioneer.py
will take a VCS name as an argument, and will construct a version of
`versioneer.py` that is specific to the given VCS. It might also take the
configuration arguments that are currently provided manually during
installation by editing setup.py . Alternatively, it might go the other
direction and include code from all supported VCS systems, reducing the
number of intermediate scripts.

## Similar projects

* [setuptools_scm](https://github.com/pypa/setuptools_scm/) - a non-vendored build-time
  dependency
* [minver](https://github.com/jbweston/miniver) - a lightweight reimplementation of
  versioneer
* [versioningit](https://github.com/jwodder/versioningit) - a PEP 518-based setuptools
  plugin

## License

To make Versioneer easier to embed, all its code is dedicated to the public
domain. The `_version.py` that it creates is also in the public domain.
Specifically, both are released under the "Unlicense", as described in
https://unlicense.org/.

<!--Latest Version-->
[pypi-image]: https://img.shields.io/pypi/v/versioneer.svg
[pypi-url]: https://pypi.python.org/pypi/versioneer/
[github-release-image]: https://img.shields.io/github/v/release/python-versioneer/python-versioneer?logo=github
[github-release-url]: https://github.com/python-versioneer/python-versioneer/releases
<!--Latest Version @ external-->
[anaconda-org-main-image]: https://img.shields.io/conda/v/main/versioneer
[anaconda-org-main-url]: https://anaconda.org/main/versioneer
[conda-forge-image]: https://img.shields.io/conda/v/conda-forge/versioneer
[conda-forge-url]: https://anaconda.org/conda-forge/versioneer
<!--Python Supports-->
[python-versions-img]: https://img.shields.io/pypi/pyversions/versioneer
<!--Downloads-->
[pypi-downloads-img]: https://img.shields.io/pypi/dm/versioneer?label=pypi
[downloads-conda-main-img]: https://img.shields.io/conda/dn/main/versioneer?label=conda%7Cmain
[downloads-conda-forge-img]: https://img.shields.io/conda/dn/conda-forge/versioneer?label=conda%7Cconda-forge
<!--Community-->
[matrix-image]: https://img.shields.io/badge/Matrix%20-chat-brightgreen?style=plastic&logo=matrix
[matrix-url]: https://matrix.to/#/#python-versioneer:mozilla.org
[gitter-image]: https://img.shields.io/badge/Gitter%20-chat-brightgreen?style=plastic&logo=gitter
[gitter-url]: https://matrix.to/#/#python-versioneer:gitter.im
[github-stars-img]: https://img.shields.io/github/stars/python-versioneer/python-versioneer?logo=github
<!--Build Status-->
[github-workflow-tox-img]:
https://img.shields.io/github/actions/workflow/status/python-versioneer/python-versioneer/tox.yml?logo=github
[github-workflow-tox-url]:
https://github.com/python-versioneer/python-versioneer/actions/workflows/tox.yml?query=branch%3Amaster
<!--License-->
[license-img]: https://img.shields.io/github/license/python-versioneer/python-versioneer
