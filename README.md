[travis-ci-badge]: https://travis-ci.org/jasonwhite/ducible.svg?branch=master
[appveyor-badge]: https://ci.appveyor.com/api/projects/status/96cmnhjw4159goep/branch/master?svg=true

# Ducible

[![Build Status][travis-ci-badge]](https://travis-ci.org/jasonwhite/ducible)
[![Build status][appveyor-badge]](https://ci.appveyor.com/project/jasonwhite/ducible/branch/master)

This is a tool to make builds of Portable Executables (PEs) and PDBs
repro*ducible*.

Timestamps and other non-deterministic data are embedded in DLLs, EXEs, and
PDBs. If some source is compiled and linked twice without changing any source,
the binary and PDB will not be bit-by-bit identical both times. This tool fixes
that by modifying DLLs/EXEs in-place and rewriting PDBs.

Don't worry, Ducible won't mess with the functionality of your executable. All
changes have no functional effect. It merely transforms one perfectly good
executable into another perfectly good, yet reproducible(!), executable.

## Why?

In general, reproducible builds give a verifiable path from *source code* to
*binary code*. There are a number of security reasons and practical reasons for
why this is good. More specifically, it enables

 * confidence that two parties built a binary with the same environment,
 * recreating a release bit-for-bit from source code,
 * recreating debug symbols for a particular version of source code,
 * verifiable and correct distributed builds,
 * no spurious changes in binaries under version control.

See also https://reproducible-builds.org/ for more information on why you should
want this.

## Using It

Usage is as follows:

    $ ducible IMAGE [PDB]

The EXE/DLL is specified as the first parameter and the PDB is optionally
specified as the second. The PDB must be modified because changing the image
invalidates the signature for the PDB.

As a post-build step, simply run:

    $ ducible MyModule.dll MyModule.pdb

The files are overwritten in-place.

## Downloading It

See the [releases][] for downloads.

[releases]: https://github.com/jasonwhite/ducible/releases

## Known Limitations

 1. This tool cannot prevent you from shooting yourself in the foot. Please
    don't ever have anything like this in your code:

    ```cpp
    std::cout << "Build date: " << __DATE__ << " " << __TIME__ << std::endl;
    ```

    There is nothing that Ducible can do about this. Embedding dates or times
    might seem useful, but all they do is prevent reproducible builds. Once you
    have reproducible builds and a proper versioning scheme, embedding this
    information is pointless.

 2. Digital signing with trusted timestamping cannot be made reproducible (e.g.,
    using Microsoft's [`signtool`][signtool]). Even while doing digital signing,
    you can still gain some of the benefits that using Ducible provides (e.g.,
    recreating a PDB for debugging purposes). Digital signatures can also be
    stripped off after being applied to make comparing binaries possible.

 3. Incremental linking using [`/INCREMENTAL`][incremental-flag] changes the
    executable quite extensivly upon subsequent builds. Ducible will invalidate
    the `.ilk` file and force the linker to do a full relink every time. However,
    this isn't enough to make the build reproducible. You can work around this
    issue by disabling `/INCREMENTAL` in the linker settings. (Unfortunately
    this is usually enabled by default for `Debug` builds in Visual Studio.)

[signtool]: https://msdn.microsoft.com/en-us/library/windows/desktop/aa387764.aspx
[incremental-flag]: https://msdn.microsoft.com/en-us/library/4khtbfyf.aspx

## Building It

This is written in C++14. There are no third party dependencies and it should be
buildable and runnable on any platform (even non-Windows!).

Required build tools:

 1. **Python 3**

    Python is used to generate `src/version.h`.

 2. **Git**

    Git is used to get the current commit hash. This is embedded in the
    `src/version.h` that is generated by Python.

Both of these tools must be in your `PATH`.

### Windows

Just open `vs/vs2015/ducible.sln` and build it. Of course, this requires Visual
Studio 2015. If another version of Visual Studio is needed, please submit an
issue or, better yet, a pull request.

### Linux

Although this is primarily a Windows utility, it was developed in a Linux
environment simply because it was faster and easier. One might also want to use
it when compiling Windows binaries on Linux. Thus, it builds and runs on Linux as
well.

To build it, just run `make`.

### OS X

This hasn't been tested on OS X, but it probably works as there is nothing
Linux-specific in the code.

To build it, just run `make`.

## Related Work

I am only aware of the [zap_timestamp][] tool in [Syzygy][]. Unfortunately, it
has a few problems:

 1. It does not work with 64-bit PE files (i.e., the PE32+ format).
 2. It does not create a reproducible PDB file.
 3. It is a pain to build. It is part of a larger suite of tools that operate on
    PE files. That suite then requires Google's depot_tools. The end result is
    that you're required to download hundreds of megabytes of tooling around
    something that should be very simple.

[zap_timestamp]: https://github.com/google/syzygy/tree/master/syzygy/zap_timestamp
[Syzygy]: https://github.com/google/syzygy

## License

As always, this tool uses the very liberal [MIT License](/LICENSE). Use it for
whatever nefarious purposes you like.
