# CMake-Scripts-3rd-Party
A collection of CMake scripts to build common 3rd-Party libraries that do not already use the CMake build system generator.

There are a number of very common third-party libraries that tend to be used everywhere. My build system of choice is CMake, but many of these third-party libraries are built using tools such as autotools, visual studio projects, or pre-defined `Makefile`s. This makes building these libraries quite difficult, and sometimes tedious, especially on Windows when the library requires tools such as MSYS to build.

The goal of this repository is to host custom `CMakeLists.txt` files that can be used to build these libraries using CMake. Initially I will be happy with a "drop-in" `CMakeLists.txt` for each library, which can be used to build the library in question, but eventually I would like to be able to use portions of the repo within other projects to aid in dependency building.

## Acknowledgements 

While I have had the idea of creating CMake scripts for these third-party libraries for quite some time, the great work done by [Björn Blissing](https://github.com/bjornblissing) for his [`osg-3rdparty-cmake` repo](https://github.com/bjornblissing/osg-3rdparty-cmake) has let me hit the ground running. The first few releases of this repo are thus taken almost verbatim from that repo and modified to suit my particular needs/desires.

In fact, it was while working on fork of Björn's repo that I decided to create this separate repo since the `osg-3rdparty-cmake` repo is specific to [OpenSceneGraph](https://github.com/openscenegraph/OpenSceneGraph), and I ran into similar issues while attempting to build the dependencies for [FreeCAD](https://github.com/FreeCAD/FreeCAD).

## Disclaimers

1. Please note that this repo is primarily here as a way for me to keep record of this work and will most likely (read: almost certainly) be biased towards achieving my goals. Sometimes, in the interest of achieving my primary goal, the code committed here will be quite specific, though I will generally try to follow good practices where I can, within the limitations imposed by the various projects involved.
2. The scripts will (out of necessity) be tailored to specific versions of the third-party libraries. There may be occasions where the same script can be used for various libraries, but in general this will not be the case. I will try to keep the "old" versions of the scripts around in case someone needs to build one of these older versions, but see the next point:
3. There is a high maintenance burden to attempting to do this, so do not expect this repo to be up-to-date with the latest version of each library. Likewise, once I have moved onto a new version of a library, the older version's script is not necessarily going to be kept up to date.
4. Many libraries out there do not use different names for debug and release builds. This makes installing such libraries side-by-side challenging. Where feasible the build scripts will be modified to accomplish this, though I will add a flag that will let the default naming convention be available as well.
5. Likewise, version numbers do not usually form part of a particular library's name (unless you are on linux and the `.so` files are versioned) so I might decide to rename some of the libraries to indicate versioning better. Again I will have a flag to revert to default behaviour.
6. There are many compilers/operating systems out there. I use a very limited subset of these, so the focus of these scripts will be on those systems/toolchains. Please feel free to add your own.
7. This is a hobby for me, not my full-time job so progress will most likely be sporadic, and usually driven by whatever I am working on at that moment in time - do not expect too much.

## Licensing

Most of the scripts are derived from the official library build scripts and are thus derived works. The licensing of the applicable library for that script will this apply. The support/infrastructure scripts that form part of this repo (not including sub-repos or linked repos, which may have their own license) are licensed according to the Unlicense:

http://unlicense.org/

## Supported OS and Toolchains

<coming>

## Supported Libraries

### <Library Name>

Version(s) Supported: 
Original Website:
Notes:
