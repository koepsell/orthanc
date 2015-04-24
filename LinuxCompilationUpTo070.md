

# Introduction #

This page summarizes the Linux build instructions that were used **up to Orthanc 0.7.0 (inclusive)**.

Instructions for Orthanc above 0.7.0 can be found [directly inside the source package](https://code.google.com/p/orthanc/source/browse/LinuxCompilation.txt).

# Static linking #

In general, the static linking **should** work on any Linux distribution (in particular, this works on **Debian Squeeze**):

```
# cmake	-DSTATIC_BUILD:BOOL=ON -DCMAKE_BUILD_TYPE=Debug
```

Peter Somlo provides [detailed instructions to statically build Orthanc on a minimal Ubuntu installation](https://groups.google.com/d/msg/orthanc-users/hQYulBBvJvs/S1Pm125o59gJ).


# Dynamic linking against system-wide libraries #

If you want to dynamically link against the system libraries, the following CMake configurations have been reported to work.

## Dynamic Linking on Ubuntu 11.10 ##

```
# cmake "-DDCMTK_LIBRARIES=wrap;oflog" -DSTATIC_BUILD=OFF -DCMAKE_BUILD_TYPE=Debug
```

_Explanation:_ You have to manually link against the `wrap` and `oflog` shared libraries because of a packaging error in `libdcmtk`.

## Dynamic Linking on Ubuntu 12.04 LTS ##

```
# cmake "-DDCMTK_LIBRARIES=wrap;oflog" -DSTATIC_BUILD=OFF  -DUSE_DYNAMIC_GOOGLE_LOG:BOOL=OFF -DDEBIAN_USE_GTEST_SOURCE_PACKAGE:BOOL=ON -DCMAKE_BUILD_TYPE=Debug
```

## Dynamic Linking on Ubuntu 12.10 ##

```
# cmake "-DDCMTK_LIBRARIES=wrap;oflog" -DSTATIC_BUILD=OFF -DDEBIAN_USE_GTEST_SOURCE_PACKAGE:BOOL=ON -DCMAKE_BUILD_TYPE=Debug ..
```

## Dynamic Linking on Debian Sid ##

```
# cmake -DSTATIC_BUILD:BOOL=OFF -DSTANDALONE_BUILD:BOOL=ON -DUSE_DYNAMIC_JSONCPP:BOOL=ON -DDEBIAN_USE_GTEST_SOURCE_PACKAGE:BOOL=ON -DCMAKE_BUILD_TYPE=Debug -DDCMTK_LIBRARIES="wrap;oflog"
```

This is the configuration from the [official Debian package](http://anonscm.debian.org/viewvc/debian-med/trunk/packages/orthanc/trunk/debian/rules?view=markup).

## Dynamic Linking on Fedora 18 and 19 ##

```
# cmake -DSTATIC_BUILD:BOOL=OFF -DSTANDALONE_BUILD:BOOL=ON -DUSE_DYNAMIC_GOOGLE_LOG:BOOL=ON -DUSE_DYNAMIC_JSONCPP:BOOL=ON -DCMAKE_BUILD_TYPE=Debug
```

This is the configuration from the [official Fedora package](http://pkgs.fedoraproject.org/cgit/orthanc.git/tree/orthanc.spec?h=f18).

## Static Linking on CentOS 6.3 and 6.4 ##

You have to build and install [CMake 2.8 from source](http://www.cmake.org/cmake/resources/software.html), or you can use the `cmake28` package from [EPEL](https://admin.fedoraproject.org/pkgdb/acls/name/cmake28). The `STATIC_BUILD=ON` option will then work:

```
# /usr/local/bin/cmake -DSTATIC_BUILD:BOOL=ON -DCMAKE_BUILD_TYPE=Debug
```

_Thanks to Will Ryder._

# Please explain the CMake build infrastructure #

The build infrastructure of Orthanc is based upon [CMake](http://www.cmake.org/). The build scripts are designed to embed all the third-party dependencies directly inside the Orthanc executable. This is the meaning of the `-DSTATIC_BUILD=TRUE` option, as described in the [INSTALL](http://orthanc.googlecode.com/hg/INSTALL) file of Orthanc.

Such a static linking is very desirable under Windows, since the Orthanc binaries do not depend on any external DLL, which results in a straightforward installation procedure (just download the Windows binaries and execute them), which eases the setup of the development machines (no external library is to be manually installed, everything is downloaded during the build configuration), and which avoids the [DLL hell](http://en.wikipedia.org/wiki/Dll_hell). As a downside, this makes our build infrastructure rather complex.

Static linking is not as desirable under Linux than under Windows. Linux prefers software that dynamically links against third-party dependencies: This is explained by the fact that whenever a third-party dependency benefits from a bugfix, any software that is linked against it also immediately benefits from this fix. This also reduces the size of the binaries as well as the build time. Under Linux, it is thus recommended to use the `-DSTATIC_BUILD=FALSE` option whenever possible.

When the dynamic build is used, some third-party dependencies may be unavailable or incompatible with Orthanc, depending on your Linux distribution. Some CMake options have thus been introduced to force the static linking against some individual third-party dependencies. Here are the most useful:

  * `-DUSE_DYNAMIC_GOOGLE_LOG=FALSE` to statically link against [Google Log](http://code.google.com/p/google-glog/).
  * `-DUSE_DYNAMIC_JSONCPP=FALSE` to statically link against [JsonCpp](http://jsoncpp.sourceforge.net/).

Please also note that the option `-DSTANDALONE_BUILD=TRUE` must be used whenever your plan to move the binaries or to install them on another computer. This option will embed all the external resource files (notably Orthanc Explorer) into the resulting executable. If this option is set to `FALSE`, the resources will be read from the source directories.