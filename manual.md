- [Installation](#orgd1d76f6)
  - [Toolkit installation](#org6629a59)
    - [Source code](#org0b4d554)
    - [Configuring the source](#orgedeb777)
    - [Building the source](#org98679a1)
    - [Install the results](#org4fb6fa2)
    - [Other build commands](#org3c4be57)
  - [Runtime environment](#orga5c76c5)
  - [Guide for installation of dependencies](#org94bfbe2)
    - [Manual](#orgcd90a28)
    - [Spack](#orgf6c4161)
    - [UPS](#orgc0aed1a)
  - [Release management](#orgadae3c9)
    - [Release versions](#org2be9ed4)
    - [Branch policy](#orgaec8b56)
    - [Branch mechanics](#orgaaf14b2)
- [Configuration](#orgea1340e)
  - [Introduction](#org26a02e2)
  - [Configuration from a user point of view](#orgb2eb1ab)
    - [File formats](#org2e00366)
    - [Basic command line](#orgadb5d8e)
    - [Dump default configuration](#orgf7ebe29)
    - [Diving into JSON](#orgd6441b8)
    - [Limitations of JSON](#orga071e07)
    - [Learning Jsonnet](#orge35ee84)
    - [Specific detector support](#org5e31ad5)
    - [Using Jsonnet](#orgf77c51b)
  - [Configuration from a developer point of view](#org0e08810)
- [Howtos](#org12ef43e)
  - [Run `wire-cell` command line program](#org3abda9e)
  - [Add a new component class](#org328b514)
- [Internals](#org03fc200)
  - [Toolkit packages](#org7d0a17a)
    - [Names](#org5905d4c)
    - [Dependencies](#orgd9e2a37)
    - [Package structure](#orgb928e97)
    - [Build package](#org7de875f)
    - [Adding a new code package](#org8fcc319)
  - [Coding conventions](#orgcde6a11)
    - [C++ code formatting](#org77592ef)
    - [C++ namespaces](#orgf54fc01)
  - [Interfaces](#orgf01828d)
  - [Components](#orgd5214c0)
  - [Configuration](#org0ab032e)
  - [Execution Models](#org3ed4406)
    - [Ad-hoc](#orgea0b306)
    - [Component](#org5d04ca6)
    - [Interface](#orgc700f34)
    - [Data flow programming](#org0b8b85e)
- [Packages](#org69c0692)
  - [Utilities](#org35ab38e)
    - [Units](#org533998c)
    - [Persistence](#org437c079)
    - [Etc](#orgbe131f2)
  - [Interfaces](#org2a97b94)
    - [Data](#orgd086459)
    - [Nodes](#org4f96cb7)
    - [Misc](#orgd66fead)
  - [Simulation](#orgc51c2cf)
    - [Depositions](#orga1f5ad9)
    - [Drifting](#org3e0b166)
    - [Response](#org7cfb0dd)
    - [Digitizing](#org06e8157)
  - [Waf tools](#org5a709a2)
    - [Recreating `wcb`](#org3181422)
    - [Included Waf tools](#org3e4b2b8)



<a id="orgd1d76f6"></a>

# Installation

The Wire Cell Toolkit (WCT) should be easy to build on any POSIX&rsquo;y system with a recent C++ compiler. This section describes how to build releases and development branches, it gives guidance for supplying the few software dependencies, and documents how releases are made.


<a id="org6629a59"></a>

## Toolkit installation     :user:installer:developer:

<div class="warning">
This assumes you already have available the required dependencies. See section [1.3](#org94bfbe2).

</div>

Installation requires four steps:

1.  get the source
2.  configure the source
3.  build the code
4.  install the results


<a id="org0b4d554"></a>

### Source code

WCT source is composed of several packages (see section [5](#org69c0692)) and all source is available from the [Wire Cell GitHub organization](https://github.com/WireCell/). Releases of each package are made and documented on GitHub (*eg* [here](https://github.com/WireCell/wire-cell-build/releases)) and can be downloaded as archives. However, using git to assemble a working source area is recommended and easier. Releases and development branches are handled slightly differently.

To obtain a release requires no GitHub authentication:

    $ git clone --recursive --branch 0.5.x https://github.com/WireCell/wire-cell-build.git

This gets the tip of the release branch for the `0.5.x` series. If a specific release is desired a few more commands are needed. For example, if the `0.5.0` release that started the series is wanted:

    $ git checkout -b 0.5.0 0.5.0
    $ git submodule init
    $ git submodule update
    $ git submodule foreach git checkout -b 0.5.0 0.5.0

To obtain the development branch requires SSH authentication with GitHub:

    $ git clone --recursive git@github.com:WireCell/wire-cell-build.git wct

Which ever way the source is obtained, enter the resulting directory

    $ cd wire-cell-build/

<div class="tip">
At some time later if there is a need to switch between HTTP or SSH a `switch-git-urls` script is available in this directory.

</div>


<a id="orgedeb777"></a>

### Configuring the source

At a minimum, the source must be configured with an installation location for the build results and to allow it to find its dependencies. This, and the remaining steps are done with the provided `wcb` script which is an instance of [Waf](https://waf.io/).

    $ ./wcb --prefix=/path/to/install configure

This will print the results of the attempts to detect required and optional dependencies. Missing but optional dependencies will not cause failure. See below for guidance on installing dependencies if this step fails or if desired optional dependencies are not found.

Dependencies are first located in standard system locations, barring that those that are traditionally installed with support for `pkg-config` may be found by suitably setting the `PKG_CONFIG_PATH` environment variable. To force a particular location for any given dependency a `--with-*` option may be used.

<div class="note">
As shown in the example below, the flags to locate Boost are slightly different.

</div>

    $ ./wcb configure --prefix=/path/to/install \
       --boost-includes=$WCT_EXTERNALS/include --boost-libs=$WCT_EXTERNALS/lib --boost-mt \
       --with-eigen=$WCT_EXTERNALS --with-jsoncpp=$WCT_EXTERNALS --with-tbb=$WCT_EXTERNALS \
       --with-root=$WCT_EXTERNALS --with-fftw=$WCT_EXTERNALS --with-root=$WCT_EXTERNALS

This example assumes all externals are available in a directory set by the `WCT_EXTERNALS` variable. This variable is **not** used by the build and is only used to make this example brief.


<a id="org98679a1"></a>

### Building the source

After a successful source configure step, the results are cached and any long command line need not be repeated. To build the source simply run:

    $ ./wcb

If there are build failures more information can be obtained by repeating the build with more verbosity:

    $ ./wcb -vv

The build will try to run tests which can be avoided to save time:

    $ ./wcb --notests


<a id="org4fb6fa2"></a>

### Install the results

To install the build results into the location given by `--prefix` simply issue:

    $ ./wcb install


<a id="org3c4be57"></a>

### Other build commands

These other commands may be useful:

    $ ./wcb clean          # clean build products
    $ ./wcb distclean      # also clean configuration
                           # build with debug symbols  
    $ ./wcb configure --build-debug=-ggdb3 [...]
                           # to save some time, just 
                           # rebuild the given test 
                           # and don't run any tests
    $ ./wcb --notests --target=test_xxx
    $ ./wcb --help         # see more options.


<a id="orga5c76c5"></a>

## Runtime environment     :user:installer:

Managing environment is usually a personal choice or computer facility policy and WCT does not place any significant requirements on this. The usual setting of `PATH` like variables will likely be needed.

FIXME: we should look into setting `RPATH`.

Internally, WCT does not require any environment however it will search a `WIRECELL_PATH` when locating configuration or other (non data) input files. More information is in the section [2](#orgea1340e).


<a id="org94bfbe2"></a>

## Guide for installation of dependencies     :installer:

The WCT depends on a number of third-party software packages. They are intentionally limited to reduce the requirements for installation. Some packages are optional. Here lists packages that users may not already have as part of a &ldquo;reasonably POSIX&rsquo;y system and some guidance is given on how to install them.

The required packages are:

-   **Boost:** various functions
-   **Eigen3:** matrix representation, interface to FFTW
-   **FFTW3:** for fast Fast Fourier Transforms
-   **JsonCPP:** basis for configuration and input data files

The optional package are:

-   **Doxygen:** for building reference documentation.
-   **Jsonnet:** structured configuration files.
-   **ROOT:** for many tests and I/O packages
-   **TBB:** for data flow programming paradigm support

<div class="note">
Some &ldquo;optional&rdquo; packages may actually be currently required for a successful build and test. This is considered a &ldquo;bug&rdquo; and needs fixing.

</div>

This list may not represent current reality. To get a full, up-to-date list of what packages WCT can use run `./wcb --help`.


<a id="orgcd90a28"></a>

### Manual

In the DIY mode, the installer is free to provide the third-party packages in any convenient way. Many of them are available on well supported operating systems such as Debian/Ubuntu. Homebrew for Mac OS X is not a core developer platform but may provide many. Redhat derived Linux distributions may find suitable package on EPEL. Most of the required packages are fairly easy to build from source.

However the installer decides to build in DIY-mode the WCT build system should be able to be given proper installation locations via the `--with-*` flags as described above. If it seems not to be the case, please contact the developers.


<a id="orgf6c4161"></a>

### Spack

[Spack](https://github.com/LLNL/spack) is a &ldquo;meta build system&rdquo; that runs the individual build systems that come with packages. It allows one to manage an ever growing installation area which can accommodate multiple versions of a package. It also comes with support for [Environment Modules](http://modules.sourceforge.net/) to handle your users&rsquo; setup of these packages or can make targeted release &ldquo;views&rdquo; of its package tree.

WCT provides a package [wire-cell-spack](https://github.com/WireCell/wire-cell-spack) which collects instructions and an Spack &ldquo;repo&rdquo; that builds WCT and its third-party dependencies. This leverages Spacks built-in &ldquo;repo&rdquo; to provide dependencies needed by WCT&rsquo;s direct dependencies. Using it will tend to build packages that one may already have installed through the OS (eg, Python). However, this duplication should not add much to the overall build time which is automatic nor lead to any problems.

An installer that wishes to use wire-cell-spack to provide the dependencies should begin by following its [README](https://github.com/WireCell/wire-cell-spack/blob/master/README.org) file.


<a id="orgc0aed1a"></a>

### UPS

Fermi National Accelerator Lab (FNAL) uses a user environment similar but incompatible with to [Environment Modules](http://modules.sourceforge.net/). It is typical to download binaries provided by FNAL and use UPS to &ldquo;setup&rdquo; a user environment with many environment variables. For each package so setup there is a variable that gives the installation location. These can be used to provide suitable values for the `--with-*` flags to `wcb` as described above.


<a id="orgadae3c9"></a>

## Release management     :core:developer:

Releases are made by developers as needed and as described in this section.


<a id="org2be9ed4"></a>

### Release versions

WCT label releases are made following a fixed procedure. Releases are labeled with the common three-number convention: `X.Y.Z`. These take the following semantic meanings:

-   **X:** a major release is made when developers believe some substantial milestone has been achieved or to being wholly new or a globally breaking development path.
-   **Y:** a minor or feature release is made when substantial new and in particular any breaking development is made.
-   **Z:** a bug release fixes problems without otherwise substantial changes.


<a id="orgaec8b56"></a>

### Branch policy

Any new major or minor releases produce a new Git branch in each package. Only bug fixes are made to this branch. Where applicable, release bug fixes should be applied to `master`. Nominally, all development is on the `master` branch however developers are free to make their own feature branches. They are encourage to do this if their development is expected to be disruptive to other developers.


<a id="orgaaf14b2"></a>

### Branch mechanics

To make releases, the above details are baked into two test scripts [make-release.sh](https://github.com/WireCell/waf-tools/blob/master/make-release.sh) and [test-release.sh](https://github.com/WireCell/waf-tools/blob/master/test-release.sh). See comments at the top of each for how to run them. These scripts can be used by others but are meant for developers to make official releases.


<a id="orgea1340e"></a>

# Configuration

As the Wire Cell Toolkit (WCT) is a toolkit, it is up to the parent application to provide some mechanism for the user to provide configuration information to WCT *components*. Users of an application not provided by the WCT itself should refer to its documentation. This section of the manual documents the configuration mechanism that is provided by WCT itself. If an application decides to use the WCT file format then its users may refer to this document. Developers of WCT components should read it as well.


<a id="org26a02e2"></a>

## Introduction

WCT itself provides a such a mechanism which is exposed to the user by the `wire-cell` command line application. Any application may easily adopt this same mechanism by making use of the `WireCell::ConfigManager` class.

This WCT configuration mechanism is described here from the point of view of **user** and **developer**. Details for each role are given in the following sections. However, both user and developer must understand one aspect of WCT internal design in order to understand configuration. One aspect of WCT is that an application is composed of a number of *component* classes. Components work together in some way to enact the job of the application. A component is specifically a C++ class which implements one or more *interface* base classes. One interface pertinent here is `IConfigurable`. A component that implements this interface is called a *configurable component* or just *configurable*. A configurable then is the atomic unit of WCT configuration and this unit is reflected in what the user provides for configuration and what developers should expect if they write configurable components.

The user then provides an ordered list of *configuration objects* or simply *configurations*. Each configuration is associated by WCT with exactly one *instance* of a configurable component class. This association is done via two string identifiers. The pair are also used to initially construct and later locate the instance of any type of WCT component (not just configurables):

-   **type:** specifies the &ldquo;configurable type&rdquo; which often matches the C++ class name with any C++ namespace removed. However, developers of configurable components are free to chose any unique type name.

-   **name:** specifies a &ldquo;configurable instance&rdquo;, that is an C++ object instance of the C++ class associated with the configurable type identifier. The name is free form and may be omitted in which case it defaults to the empty string. A specific name is needed if multiple instances are required or if multiple configurables require sharing a component.

Finally, configurations have a third attribute:

-   **data:** specifies a data structure following a schema specific to the configurable type. This is the &ldquo;payload&rdquo; that WCT gives to the instance of the configurable component.

In the next section, WCT user-configuration support is described. The following section gives guidance to developers who wish to write their own configurable components.


<a id="orgb2eb1ab"></a>

## Configuration from a user point of view     :user:

Users of the WCT command line interface `wire-cell` or any WCT application that uses `WireCell::ConfigManager` can provide configuration information in the form of one or more files. This files express the same ordered list of configuration objects as described above.


<a id="org2e00366"></a>

### File formats

WCT supports two related configuration file formats: [JSON](http://www.json.org/) and [Jsonnet](http://jsonnet.org/). Of the two, JSON is more fundamental while Jsonnet provides a way to better organize and construct complex configurations. Jsonnet support is a compile-time option. The stand-alone `jsonnet` program may also be used to evaluate Jsonnet into JSON.


<a id="orgadb5d8e"></a>

### Basic command line

A user gives one or more configuration files to the `wire-cell` application each with a `-c` flag:

    $ wire-cell -c myparameters.cfg [...]

If a relative path is given, the file will be searched for starting in the current working directory and then in each directory listed in a `WIRECELL_PATH` environment variable, if given. When multiple configuration are used, their top-level arrays are conceptually concatenated in the order on which they are given on the command line.


<a id="orgf7ebe29"></a>

### Dump default configuration

The user can also dump out the hard-coded default configuration for one or more components:

    $ wire-cell \
        -p <plugin> \
        -D <component1> \
        -D <component2> \
        --dump-file=mydefaults.cfg

Here the components to dump are specified by their &ldquo;type&rdquo; identifier as described above. The *plugins* which provide the components must be specified. A plugin typically takes the name of its shared library with the `lib` prefix and `.so` extension removed.

The user must know what components to dump. There is no way for the application to iterate over all possible components. In general, it is up to the provider of a plugin to catalog what component types it provides. WCT provides a [simple script](https://github.com/WireCell/wire-cell-cfg/blob/master/scripts/generate-defaults.sh) that will search the WCT source, determine the components and dump them out using `wire-cell`. Result of this dump, possibly out of date, is available in the [wire-cell-cfg](https://github.com/WireCell/wire-cell-cfg/tree/master/defaults) repository.


<a id="orgd6441b8"></a>

### Diving into JSON

An example configuration dump from this command

    $ wire-cell -D TrackDepos -p WireCellGen

produces:

```js
[
   {
      "data" : {
	 "clight" : 1,
	 "step_size" : 0.10000000000000001,
	 "tracks" : []
      },
      "name" : "",
      "type" : "TrackDepos"
   }
]
```

Here we see an array holding one element which is an object with the `type`, (instance) `name` and payload `data` structure as described above. If `wire-cell` were to load this configuration it would create a default instance of the component type `TrackDepos` which happens to correspond to the C++ class `WireCell::Gen::TrackDepos` (see the [simulation package manual](./gen.md) for more information). This component is responsible for produces deposition (`IDepo`) objects using a simple linear source model.

The `tracks` array in this example is empty and no depositions would be produced. The user most certainly should specify a nonempty set of tracks. In principle, the user may produces a huge `tracks` array. WCT support bzip2 compressed JSON files (see the section on [persistence in the util package manual](./util.md)).


<a id="orga071e07"></a>

### Limitations of JSON

As the complexity of a `wire-cell` job grows, hand crafting JSON becomes tedious and error prone. Splitting the files and/or using `WIRECELL_PATH` can provide some rudimentary means of organizing a large, complex configuration.

However, a user will quickly outgrow direct authoring of JSON files. An accomplished user will likely turn to some form of JSON generation using a more expressive language maybe by developing some scripts. Or, some part of a configuration may need to be extracted or converted from another source. For example, Geant4 steps might be extracted and formatted into a `TrackDepos` configuration as a long `tracks` array.

Another limitation is that any numerical quantities **must** be expressed in the base units used by the WCT *system of units* (see the section on [units in the Utilities manual](./util.md)). This places a burden on the configuration author and is a source of error.

The user is free to generate JSON in any manner they wish as long as the result conforms to the required schema. However, WCT provides a second, more powerful JSON-like configuration file format which described next.


<a id="orge35ee84"></a>

### Learning Jsonnet

WCT provides support for configuration files following the [Jsonnet data templating language](http://jsonnet.org/). This language is evaluated to produce JSON. If WCT is compiled with support it can evaluation Jsonnet files directly. Otherwise the user may install and run the `jsonnet` command line program to produce JSON for feeding to WCT.

To learn how to write Jsonnet in general, the user should refer to its documentation which is excellent. There is no one right way to write Jsonnet, however, the [wire-cell-cfg](https://github.com/wirecell/wire-cell-cfg) package provides a number of examples and support files that can help the user craft their configuration in Jsonnet. In particular the WCT system of units and some common data structures used by WCT are exported to Jsonnet in [wirecell.jsonnet](https://github.com/WireCell/wire-cell-cfg/blob/master/wirecell.jsonnet). Some of this exported functionality is illustrated below.

WCT locates Jsonnet files as it does JSON files and in particular using the environment variable `WIRECELL_PATH`. However, it does not (currently) support compressed Jsonnet files.


<a id="orgb7fc00e"></a>

#### System of units

Wire Cell provides an internal system of units as described in the section on [units in the Utilities manual](./util.md) and as stated above, users must take care to give numerical quantities in WCT units when providing JSON. However, when writing Jsonnet one can provide explicit units which is easy and far less error prone. For example:

```js
local wc = import "wirecell.jsonnet";
[
    {
    type:"TrackDepos",
    data: {
	step_size: 1.0 * wc.millimeter,
	// or could abreviate with wc.mm
	}
    }
]
```


<a id="org035c150"></a>

#### Functions

Some data sub-structures are needed in multiple laces and it can be laborious to write them by hand. Jsonnet provides functions to assist in this. A number of functions are defined to assist in representing common data types. For example `point()` and `ray()`:

```js
{
  // ...
  tracks : [ wc.ray(wc.point(10,0,0,wc.cm),
	     wc.point(100,10,10,wc.cm)) ]
},
```


<a id="org8e4f048"></a>

#### Default parameters

It is typical that different components must share common values, or separate values which derive from common values. Jsonnet allows for this to be expressed in the configuration in a simple manner. For example, in the `gen` package both `Drifter` and `Ductor` may apply statistical fluctuations. For debugging it can be useful to turn this feature off. This can be done in a consistent manner like in a global parameter file

```js
// in uboone/globals.json
{
    // ...
    // True if simulation should do fluctuations
    fluctuate: true,
    // ...
}
```

This file can then be imported so that this variable may be applied where ever it is needed.

```js
// in uboone/components.jsonnet
local params = import "uboone/globals.jsonnet";
{
    // ...
    drifter: {
	type : "Drifter",
	data : {
	    // ... other parameters ...
	    fluctuate : params.fluctuate,
	}
    },
    ductor: {
	type : 'Ductor',
	data : {
	    // ... other parameters ...
	    fluctuate : params.fluctuate,
	}
    },        
    // ...
}
```

See next how these definitions are used.


<a id="org7088e0f"></a>

#### Default structures

One useful way to factor a configuration is to have one Jsonnet file which holds default values and one or more that customize on top of those defaults. For example one the [MicroBooNE configuration](https://github.com/WireCell/wire-cell-cfg/tree/master/uboone) provided by `wire-cell-cfg` defines a default configuration for the `FourDee` WCT app.

<div class="note">
An &ldquo;app&rdquo; is a top level main class run by WCT while an &ldquo;application&rdquo; refers to a program built with WCT that a user runs.

</div>

This app is configured with a list of components to use for certain portions of the &ldquo;FourDee&rdquo; simulation. By default these can are configured with the default types provided directly in the `gen` package. Note, these configuration are generally in the form `"TypeName:InstanceName"` but the defaults to not specify an instance name.

```js
// in uboone/components.json
{
    // ...
    fourdee : {
	type : 'FourDee',
	data : {
	    DepoSource: "TrackDepos",
	    Drifter: "Drifter",
	    Ductor: "Ductor",
	    Dissonance: "SilentNoise",
	    Digitizer: "Digitizer",
	    FrameSink: "DumpFrames",            
	},
    },
    // ...
}
```

The default type for `FrameSink` is given as `DumpFrames`. This component just prints a little bit of info to the terminal. The user probably wants to be able to save the result of the simulation in some more useful way. The [simple I/O](https://github.com/WireCell/wire-cell-sio) package provides a `FrameSink` which will save the resulting simulated waveforms as 2D ROOT histograms. The user merely needs to override `FrameSink` like:

```js
// assumes user has this directory in their WIRECELL_PATH
local uboone = import "uboone/components.jsonnet";
[
    // ... skip other overrides ...

    uboone.fourdee {
	data : super.data {
	    FrameSink: "HistFrameSink",            
	}
    },
]
```

This says to override `uboone.fourdee` with what&rsquo;s given. The `type` is inherited. The `data` is replaced by the parent&rsquo;s via `super.data` plus the additional override of the `FrameSink` attribute.


<a id="org252ad64"></a>

#### Commas

One of the most irritating aspect of crafting JSON files by hand is that any array or object must **not** have an internal trailing comma. Jsonnet allows this otherwise extraneous comma, as shown in the example above. For this reason alone and if no other features are used, writing Jsonnet instead of raw JSON is worth the added dependency!


<a id="org5e31ad5"></a>

### Specific detector support

The `wire-cell-cfg` package also provides support for popular LArTPC detectors. You can find these files under a directory named for the experiment (such as [that for MicroBooNE](https://github.com/WireCell/wire-cell-cfg/tree/master/uboone)).


<a id="orgf77c51b"></a>

### Using Jsonnet

Jsonnet&rsquo;s command line program `jsonnet` is fast and gives good error messages. It&rsquo;s often easiest to develop a Jsonnet configuration with periodic checking. Assuming the current working directory is the top of the WCT source then running the following:

    $ jsonnet -J cfg cfg/uboone/fourdee.jsonnet

should reward you with a big screen full of JSON. You can then run `wire-cell` something like:

    $ wire-cell -c uboone/fourdee.jsonnet

This relies on the `WIRECELL_PATH` to include the `cfg/` directory as well as any directories holding any configuration data files referenced by the configuration.


<a id="org0e08810"></a>

## Configuration from a developer point of view     :devel:

For the C++ part of developing WCT components or applications the developer should refer to the [configuration section in the manual on WCT Internals](./internals.md).

In addition, a developer is encouraged to provide Jsonnet files that abstract away any less important details and give users a simplified way to configure the developers components.

In particular, if the developer writes multiple components, an application component or a component that refers to another component, working example configuration files should be provided.


<a id="org12ef43e"></a>

# Howtos

This section of the manual gives brief guidance on how to do various things with WCT.


<a id="org3abda9e"></a>

## Run `wire-cell` command line program


<a id="org328b514"></a>

## Add a new component class


<a id="org03fc200"></a>

# Internals

This doc describes the Wire Cell Toolkit (WCT) internal structure and support facilities. It is intended for developers to read carefully, understand and follow. It may be of interest to users as well. It does not cover the &ldquo;batteries included&rdquo; or &ldquo;reference implementations&rdquo; such as the simulation, signal processing, imaging, etc which are described in section [5](#org69c0692).


<a id="org7d0a17a"></a>

## Toolkit packages

The WCT is composed of a number of *packages*. Each package has an associated with a Git source repository. Most packages produce a shared library, which may also be a WCT plugin library, C++ header files, some number of main or test applications. Others include a single package holding all Python code in various modules, a package providing support for developing WCT configuration files and the documentation package holding this document. One special type of package is a *build* package described more in section on the build package.


<a id="org5905d4c"></a>

### Names

Package repositories are named like `wire-cell-<name>` where `<name>` is some short identifier giving indication of the main scope of the package. In the documentation the `wire-cell-` prefix is often dropped and only this short name is used.

If a package produces a shared library it should be named in `CamelCase` with a prefix `WireCell`. For example the `gen` package produces a library `libWireCellGen.so`. As a plugin name or an entry in the build system, the `lib` and `.so` are dropped. If the package has public header files to expose to other packages they should use this same name for a subdirectory in which to hold them. Package layout is described move below.


<a id="orgd9e2a37"></a>

### Dependencies

Some of the C++ packages are designated as *core* packages. These include the packages providing the toolkit C++ structure (described later in this document) as well as the reference implementations (eg, `gen`, `sigproc`). These packages have strict requirements on what dependencies may be introduced and in particular their shared libraries are not allowed to depend on ROOT (although their apps and tests are, see sections [4.1.3](#orgb928e97) and [4.1.4](#org7de875f)).

The base package is `util` and it *must* not depend on any other WCT package. The next most basic is `iface` and it *must* not depend on any other WCT except `util`. Core implementation packages such as `gen` or `sigproc` may depend on both but should not depend on each other.

Fixme: there is a need to factor some general utility routines and data structures that depend on `iface` and which the implementation packages should use that needs to be created.

WCT also provides a number of peripheral implementation packages, which are free to have more dependencies, including ROOT, than &ldquo;core&rdquo; packages. These are mostly for the purpose of providing WCT components which provide file I/O. The `sst` package in particular support the so called `celltree` ROOT `TTree` format used by the Wire Cell prototype code.

Finally, there may be third-party implementation packages. They are free to mimic WCT packages but WCT itself will not depend on them. They should not make use of the `WireCell::` C++ namespace.


<a id="orgb928e97"></a>

### Package structure

The WCT package layout and file extensions must follow some conventions in order to greatly simplify the build system. In the description below, `WireCellName` is as described above.

-   **`src/*.cxx`:** C++ source file for libraries with .cxx extensions or private headers
-   **`inc/WireCellName/*.h`:** public/API C++ header files with .h extensions
-   **`test/test_*.cxx`:** main C++ programs named like test<sub>\*</sub>.cxx, may also hold Python, shell scripts, private headers
-   **`apps/*.cxx`:** main application(s), one appname.cxx file for each app named `appname`, should be very limited in number

In the root of each C++ package directory must exist a file called `wscript_build`. It typically consist of a single line with a method call like:

```python
bld.smplpkg('WireCellName', use='...')
```

The `bld` object is automagically available. If the package has no dependencies then only the name is given. Most packages will need to specify some dependencies via `use` or may specify a different list of dependencies just for any applications (using `app_use`) or for the test programs (via `test_use`). Dependencies are transitive so one must only list those on which the package directly depends.

Fixme: make a script that generates a dot file and show the graph.


<a id="org7de875f"></a>

### Build package

To actually build WCT see the section on toolkit installation (section [1](#orgd1d76f6)). The build system is based on [Waf](https::waf.io) and uses the `wcb` command and a `wscript` file provided by the top level *build package*. More details on the build system are given in section [5.4](#org5a709a2))

Besides holding the main build instructions this package aggregates all the other packages via Git&rsquo;s &ldquo;submodule&rdquo; feature. In principle, there may be more than one build package maintained. This allows developers working on a subset to avoid having to build unwanted code. In practice there is a single build package which is at: <https://github.com/wirecell/wire-cell-build>.


<a id="org8fcc319"></a>

### Adding a new code package

To add a new code package to a build package from scratch, select a `<name>` following guidance above and do something like:

    $ mkdir <name>
    $ cd <name>/
    $ echo "bld.smplpkg('WireCell<Name>', use='WireCellUtil WireCellIface')" > wscript_build
    $ git init
    $ git add wscript_build
    $ git commit -a -m "Start code package <name>"

Replace `<name>` with your package name. You can create and commit actual code at this time as well following the layout in [4.1.3](#orgb928e97).

Now, make a new repository by going to the [WireCell GitHub](https://github.com/WireCell) and clicking &ldquo;New repository&rdquo; button. Give it a name like `wire-cell-<name>`. Copy-and-paste the two command it tells you to use:

    $ git remote add origin git@github.com:WireCell/wire-cell-<name>.git
    $ git push -u origin master

If you made your initial package directory inside the build package move it aside. Then, from the build package directory, add this new repository as a Git submodule:

    $ cd wire-cell-build/ # or whatever you named it
    $ git submodule add -- git@github.com:WireCell/wire-cell-<name>.git <name>
    $ git submodule update
    $ git commit -a -m "Added <name> to top-level build package."
    $ git push

In order to be picked up by the build the new package short name must be added to the `wscript` file.


<a id="orgcde6a11"></a>

## Coding conventions


<a id="org77592ef"></a>

### C++ code formatting

-   Base indentation *should* be four spaces.

-   Tabs *should* not be used.

-   Opening braces *should not* be on a line onto themselves, closing braces *should be*.

-   Class names *should* be `CamelCase`, method and function names *should be* `snake_case`, class data attributes *should be* prefixed with `m_` (signifying &ldquo;member&rdquo;).

-   Doxygen triple-slash `///` or double-star `/** */` comments *must* be used for in-source reference documentation.

-   Normal comments *may* be used for implementation documentation.

-   Interface classes and their types and methods *must* each have a documenting Doxygen comment.

-   Header files *must* have `#ifndef/#define/#endif` protection.

-   The C++ `using namespace` keyword *must not* be used at top file scope in a header.

-   Unused headers *should not* be retained.

-   Any =#include# need in an implementation file but not the corresponding header file *should not* be in the header file.


<a id="orgf54fc01"></a>

### C++ namespaces

-   All C++ code part of WCT proper and which may be accessed by other packages (eg, exported via &ldquo;public headers&rdquo;) *must* be under the `WireCell::` namespace.

-   WCT core code (`util` and `iface` packages) *may* exist directly under `WireCell::` but bare functions *must* be in a sub namespace.

-   Non-core, WCT implementation code (eg contents of `gen` package) *must* use secondary namespace (eg `WireCell::Gen::`).

-   Any third-party packages providing WCT-based components or otherwise depending on WCT *should not* use the `WireCell::` namespace.


<a id="orgf01828d"></a>

## Interfaces

A central design aspect of the WCT is that all &ldquo;important&rdquo; functionality which may have more than one implementation must be accessed via an *pure abstract interface class*. All such interface classes are held in the [iface](https://github.com/wirecell/wire-cell-iface) package. Interface classes should present a very limited number of purely abstract methods that express a single, cohesive concept. Implementations typically inherent from more than one interface. If two concepts are close but not cohesive they are best put into two interface classes. Besides defining the method interface, Interface classes may define types. They may also be templated.

After an implementation of an interface is instantiated and leaves local scope it should be referenced only through one of its interfaces. It should be held through an appropriately typed `std::shared_ptr<>` of which one should be defined as `ITheInterface::pointer`.

Interfaces are used not only to access functionality but the data model for major working data is defined in terms of interfaces inheriting from `WireCell::IData`. Once an instance is created it is immutable.

Another category of interfaces are those which express the &ldquo;node&rdquo; concept. They inherit from `WireCell::INode`. These require implementation of an `operator()` method. Nodes make up the main unit of code. They are somewhat equivalent to `Algorithm` concept from the Gaudi framework where the `operator()` method is equivalent to Gaudi&rsquo;s `execute()` method. They also require some additional instrumenting in order to participate in the data flow programming paradigm described below.


<a id="orgd5214c0"></a>

## Components

Components are implementations an interface which itself inherits from the `WireCell::IComponponent` interface class (this interface class is in `util` as a special case due to dependency issues. fixme: needs to be solved with a general package depending on both `iface and =util`). This inheritance follows [CRTP](https://en.wikipedia.org/wiki/Curiously_recurring_template_pattern).

Components also must have some tooling added in their implementation file. This is in the form of a single CPP macro which generates a function used to load a factory that can create and retain instances based on a *type* name and an *instance* name. For `WireCell::Gen::TrackDepos` the tooling looks like:

```c++
#include "WireCellUtil/NamedFactory.h"
WIRECELL_FACTORY(TrackDepos, WireCell::Gen::TrackDepos, WireCell::IDepoSource, WireCell::IConfigurable);
```

Note, this macro needs to appear before any `using namespace` directives. The arguments to the macro are:

1.  The &ldquo;type name&rdquo; which is typically the class name absent any namespace prefixes. It must be unique across the entire WCT application.
2.  The full class name.
3.  A list of all interfaces that it implements.

A component may be retrieved as an interface using the *named factory pattern* implemented in WCT. If the component has yet to be instantiated it will be through this lookup. This is performed with code like:

```c++
#include "WireCellUtil/NamedFactory.h"
auto a = Factory::lookup<IConfigurable>("TrackeDepos");
// or
auto b = Factory::lookup<IConfigurable>("TrackeDepos","some instance name");
// or
auto c = Factory::lookup_tn<IConfigurable>("TrackeDepos:");
// or
auto d = Factory::lookup_tn<IConfigurable>("TrackeDepos:some instance name");
```

The four example differ in if an instance name is known and if it is known separately from the type name or in the canonical join (eg as `type:name`). The returned value in this example is a `std::shared_ptr<const IConfigurable>`. This example accesses the `IConfigurable` interface of `TrackDepos`. Not typically required by most code but there exists also a function `lookup_factory()` to get the factory that constructs the component instance.


<a id="org0ab032e"></a>

## Configuration

One somewhat special component interface is `IConfigurable`. A class inheriting from this interface is considered a *configurable component* such as `TrackDepos` in the above example. It is *required* for any main application using the WCT toolkit to adhere to the Wire Cell Toolkit Configuration Protocol. This is a contract by which the main application promises to do the following:

1.  Load in user-provided configuration information (see the configuration section of hte manual)
2.  Instantiate all configurables referenced in that configuration.
3.  Request the default configuration object from each instance.
4.  Update that object with, potentially partial, information provided by the user.
5.  Give the instance the updated configuration object.
6.  Do this before entering any execution phase of the application.

If the main application uses `WireCell::Toolkit` then the protocol can be enacted with code similar to

```c++
using namespace WireCell;
ConfigManager cfgmgr();
// ... load up cfgmgr
for (auto c : cfgmgr.all()) {
    string type = get<string>(c, "type");
    string name = get<string>(c, "name");
    auto cfgobj = Factory::lookup<IConfigurable>(type, name); // throws 
    Configuration cfg = cfgobj->default_configuration();
    cfg = update(cfg, c["data"]);
    cfgobj->configure(cfg);
}
```

FIXME: shouldn&rsquo;t we put this all inside `ConfigManager`?

Developers of new configurables should keep this protocol in mind and should refer to existing configurables for various useful patterns to provide their end of the exchange.


<a id="org3ed4406"></a>

## Execution Models


<a id="orgea0b306"></a>

### Ad-hoc

Direct calling of utility functions and concrete objects.


<a id="org5d04ca6"></a>

### Component

Concrete components.


<a id="orgc700f34"></a>

### Interface

Using NamedFactory.


<a id="org0b8b85e"></a>

### Data flow programming

Using abstract DFP


<a id="org69c0692"></a>

# Packages


<a id="org35ab38e"></a>

## Utilities

Introduction.


<a id="org533998c"></a>

### Units

Describe units.


<a id="org437c079"></a>

### Persistence

Describe support for persistent files including compression and location.


<a id="orgbe131f2"></a>

### Etc

&#x2026;.


<a id="org2a97b94"></a>

## Interfaces

Brief overview but it&rsquo;s also in <./internals.md> so don&rsquo; t over do it.


<a id="orgd086459"></a>

### Data


<a id="org4f96cb7"></a>

### Nodes


<a id="orgd66fead"></a>

### Misc


<a id="orgc51c2cf"></a>

## Simulation

Scope and intro blah blah.


<a id="orga1f5ad9"></a>

### Depositions


<a id="org3e0b166"></a>

### Drifting


<a id="org7cfb0dd"></a>

### Response


<a id="org06e8157"></a>

### Digitizing


<a id="org5a709a2"></a>

## Waf tools

The WCT build system is based on [Waf](https://waf.io/). The parts of the build system include:

-   the `wcb` command provided by `wire-cell-build` which is a bundled version of Waf&rsquo;s `waf` command.
-   a number of Waf tools provide instructions for finding the required and optional software dependencies
-   the main `wscript` and per-package `wscript_build` files provide the high-level instructions for building WCT (ie, they are like old fashioned `Makefile` files).


<a id="org3181422"></a>

### Recreating `wcb`

The `wcb` command bundles some optional Waf tools which are not included in the default version of the `waf` command. In case new versions of Waf or new tools are needed it can be recreated like this:

    $ git clone https://github.com/waf-project/waf.git
    $ cd waf/
    $ ./waf-light --tools=compat15,doxygen,boost,bjam
    $ cp waf /path/to/wire-cell-build/wcb
    $ cd /path/to/wire-cell-build
    $ git commit [...]


<a id="org3e4b2b8"></a>

### Included Waf tools

A number of Waf tools are provided in the [waftools](https://github.com/wirecell/wire-cell-waftools) submodule. This provides a Python module for each software package which is a required or optional dependency and which is not already covered by Waf itself. New dependencies can be added by using existing modules as examples. It is the `smplpkgs.py` module which handles the building of the WCT packages themselves. The `wcb.py` module is used as a simple aggregate of all the other modules. It is this that is loaded by the main `wscript`.

<div class="note">
The scripts to make and test releases are also housed in this package.

</div>
