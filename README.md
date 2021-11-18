[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Build Status](https://travis-ci.com/arminbiere/satch.svg?branch=master)](https://travis-ci.com/arminbiere/satch)

Sat Solver SATCH
================

This is the source code of SATCH a SAT solver written from scratch in C.

The actual version number can be found in [`VERSION`](VERSION) and
changes in the latest release are documented in [`NEWS.md`](NEWS.md).

The main purpose of this solver is to provide a simple and clean code base
for explaining and experimenting with SAT solvers. It is simpler than the
source code of [CaDiCaL](https://github.com/arminbiere/cadical) and
particularly [Kissat](https://github.com/arminbiere/kissat), while still
featuring most important implementation techniques needed to obtain a
state-of-the-art SAT solver. However, even though current version has
bounded variable elimination implemented, which is arguably the most
important preprocessing and inprocessing procedure, but still lacks other
preprocessing techniques and only supports incremental solving partially.

The code and its documentation is also meant to serve as a gentle
introduction into the code base of
[CaDiCaL](https://github.com/arminbiere/cadical) and
[Kissat](https://github.com/arminbiere/kissat).

It is possible to switch off general and more basic features at compile
time by using different options to [`configure`](configure). For instance
completely disabling clause learning can be achieved with `./configure
--no-learn`.  This not only gives a clean separation of features in the code
but also makes it easier to disable (through the C pre-processor) redundant
not needed code anymore if a certain feature is disabled.

For a more complete SAT solver you might want to use
[CaDiCaL](https://github.com/arminbiere/cadical), particularly for
incremental usage, and for fastest solving fall back to
[Kissat](https://github.com/arminbiere/kissat).

Building
========

Run

   ./configure && make test

to build and test the solver binary `satch` as well as the
library `libsatch.a`:

- [`satch`](satchs)          is the stand-alone solver binary
- [`satch.h`](satch.h)       is the solver API similar to IPASIR
- [`libsatch.a`](libsatch.a) is the library with API in [`satch.h`](satch.h)

The source of the application and library consists of the following:

- [`satch.c`](satch.c)       provides the library code of the SAT solver
- [`features.h`](features.h) checks consistency of feature selection
- [`colors.h`](colors.h)     defines shared code for using terminal colors
- [`rsort.h`](rsort.h)       is a generic radix sort implementation (header file only)
- [`stack.h`](stack.h)       is a generic stack implementation (header file only)
- [`queue.h`](queue.h)       simplistic queue implementation (header file only)
- [`config.c`](config.c)     provides build-information generated by `mkconfig.sh`
- [`main.c`](main.c)         contains application code with parser and witness printer

In the [`features`](features) sub-directory reside the followings files read
by [`features.h`](features.h):

- [`features/check.h`](features/check.h)       checks implied and clashing options
- [`features/diagnose.h`](features/diagnose.h) diagnoses and print final set of options
- [`features/init.h`](init.h)                  initializes additional (implied) options

You might want to consult [`features/README.md`](features/README.md) for
more information on their meaning and how they are generated.

The files used by the build process are the following:
               
- [`VERSION`](VERSION)         contains  current version
- [`configure`](configure)     is the configuration utility
- [`.config`](.config)         saved last configuration
- [`makefile.in`](makefile.in) is a makefile template used by [`configure`](configure)
- [`makefile`](makefile)       is generated from
                               [`makefile.in`](makefile.in) by [`configure`](configure)
- [`mkconfig.sh`](mkconfig.sh) generates [`config.c`](config.c) to provide build information

The [`configure`](configure) script will generate `makefile` from the template
[`makefile.in`](makefile.in).  The default make goal `all` first calls
[`mkconfig.sh`](mkconfig.sh) to generate `config.c` to record build and version
information.  Then the object files `config.o`, `satch.o` and `main.o` are
compiled.  The first two are combined to form the library `libsatch.a` which
is linked against `main.o` to produce the solver binary `satch`.  The `test`
target will call the shell script [`tatch.sh`](tatch.sh), which performs
tests on CNFs in the [`cnfs`](cnfs) and [`xnfs`](xnfs) directories.
See below for information on testing and debugging.

Refer to `./configure -h` for build options and after building the solver to
`./satch -h` for run-time options of the solver (solver usage is also shown at
the top of [`main.c`](`main.c`).  For debugging you can use `./configure -g` and
optionally then at run-time also enable logging with `./satch -l`.

Testing
=======

For testing and debugging the following are used:

- [`cnfs`](cnfs)            directory containing CNF files for testing
- [`xnfs`](xnfs)            directory containing XNF files for testing
- [`catch.h`](catch.h)      header file of internal proof checker for testing and debugging
- [`catch.c`](catch.c)      implementation of internal proof checker for testing and debugging
- [`testapi.c`](testapi.c)  simple separate (preliminary) API test suite
- [`tatch.sh`](tatch.sh)    test suite for CNFs in [`cnfs`](cnfs) and
                            [`xnfs`](xnfs) and for building and running [`testapi`](testapi)
- [`testapi`](testapi)      binary built from [`testapi.c`](testapi.c) by
                            [`tatch.sh`](tatch.sh)

Furthermore, as we are having many different combinations of configurations,
testing them is highly non-trivial and is achieved with the following:

- [`gencombi.c`](gencombi.c)         generates configurations ("eat your own dogfood")
- [`gencombi`](gencombi)             binary built from `gencombi.c` (by `make test`)
- [`checkconfig.sh`](checkconfig.sh) checks configurations (e.g., produced
                                     by [`gencombi`](gencombi))

We have a flexible combinatorial testing flow which uses
[`gencombi`](gencombi) to produce sets of configurations that can be tested
with [`checkconfig.sh`](checkconfig.sh):

    ./gencombi         | ./checkconfig.sh    # covers all valid pairs
    ./gencombi -a 2    | ./checkconfig.sh    # 2-fold valid combinations
    ./gencombi -a 3    | ./checkconfig.sh    # 3-fold valid combinations
    ./gencombi -a -i 2 | ./checkconfig.sh -i # check invalid option pairs

The first of these uses the SAT solver to generate a set of configurations
which covers all valid pairs of options and at the same time makes sure that
there is a configuration which does not contain it.  There are also
corresponding make goals `test-two-ways`, `test-all-pairs`, and
`test-all-triples`.
 
Armin Biere  
May 2021
