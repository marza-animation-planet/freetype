import os
import sys
import pprint
import excons


env = excons.MakeBaseEnv()

out_incdir = excons.OutputBaseDirectory() + "/include"
out_libdir = excons.OutputBaseDirectory() + "/lib"

staticlib = (excons.GetArgument("freetype-static", 1, int) != 0)

cmake_opts = {"BUILD_SHARED_LIBS" : (0 if staticlib else 1), # not supported on windows with msvc
              "WITH_ZLIB": 1,
              "WITH_BZIP2": 1,
              "WITH_PNG": 1,
              "WITH_HARFBUZZ": 0}

cfg_deps = []
zlib_from_source = False
libpng_overrides = {}

# ZLIB Setup ===================================================================

def ZlibLibname(static):
    return ("z" if sys.platform != "win32" else ("zlib" if static else "zdll"))

def ZlibDefines(static):
    return ([] if static else ["ZLIB_DLL"])

rv = excons.cmake.ExternalLibRequire(cmake_opts, name="zlib", libnameFunc=ZlibLibname, definesFunc=ZlibDefines)
if rv is None:
    excons.PrintOnce("freetype: Build zlib from sources ...")
    excons.Call("zlib", imp=["RequireZlib", "ZlibName", "ZlibPath"])

    cfg_deps.append(excons.cmake.OutputsCachePath("zlib"))

    zlibstatic = excons.GetArgument("zlib-static", 1, int)
    zlibpath = ZlibPath(static=zlibstatic)

    cmake_opts["ZLIB_LIBRARY"] = zlibpath
    cmake_opts["ZLIB_INCLUDE_DIR"] = out_incdir

    def ZlibRequire(env):
        RequireZlib(env, static=zlibstatic)

    # Setup overrides for libpng subprojects
    libpng_overrides["with-zlib"] = os.path.dirname(os.path.dirname(zlibpath))
    libpng_overrides["zlib-static"] = zlibstatic
    libpng_overrides["zlib-name"] = ZlibName(static=zlibstatic)

    zlib_from_source = True

else:
    ZlibRequire = rv

# BZIP2 Setup ===================================================================

def Bzip2Libname(static):
    return ("bz2" if sys.platform != "win32" else "libbz2")

def Bzip2Defines(static):
    return ([] if static else ["BZ_DLL"])

rv = excons.cmake.ExternalLibRequire(cmake_opts, name="bzip2", libnameFunc=Bzip2Libname, definesFunc=Bzip2Defines)
if rv is None:
    excons.PrintOnce("freetype: Build bzip2 from sources ...")
    excons.Call("bzip2", imp=["RequireBzip2", "Bzip2Name", "Bzip2Path"])

    libpath = Bzip2Path()
    cfg_deps.append(libpath)

    bz2static = (excons.GetArgument("bzip2-static", 1, int) != 0)

    cmake_opts["BZIP2_LIBRARY_RELEASE"] = libpath
    cmake_opts["BZIP2_LIBRARY_DEBUG"] = libpath
    cmake_opts["BZIP2_INCLUDE_DIR"] = out_incdir

    def Bzip2Require(env):
        RequireBzip2(env)

else:
    Bzip2Require = rv

# LIBPNG Setup ===================================================================

def PngLibname(static):
    return ("bz2" if sys.platform != "win32" else "libbz2")

rv = excons.cmake.ExternalLibRequire(cmake_opts, name="libpng", libnameFunc=PngLibname, varPrefix="PNG_")
if rv is None:
    excons.PrintOnce("freetype: Build libpng from sources ...")
    if zlib_from_source:
        excons.cmake.AddConfigureDependencies("libpng", [excons.cmake.OutputsCachePath("zlib")])

    excons.Call("libpng", overrides=libpng_overrides, imp=["RequireLibpng", "LibpngName", "LibpngPath"])

    cfg_deps.append(excons.cmake.OutputsCachePath("libpng"))

    pngstatic = (excons.GetArgument("libpng-static", 1, int) != 0)
    libpath = LibpngPath(static=pngstatic)

    cmake_opts["PNG_LIBRARY"] = libpath
    cmake_opts["PNG_INCLUDE_DIR"] = out_incdir

    def PngRequire(env):
        RequireLibpng(env, static=pngstatic)

else:
    PngRequire = rv


# Freetype library ===================================================================

prjs = [
    {   "name": "freetype",
        "type": "cmake",
        "cmake-opts": cmake_opts,
        "cmake-cfgs": ["./CMakeLists.txt"] + cfg_deps,
        "cmake-srcs": excons.CollectFiles(".", patterns=["*.c"], recursive=True)
    }
]

excons.AddHelpOptions(libtiff="""FREETYPE OPTIONS
  freetype-static=0|1 : Toggle between static and shared library build. [1]
  zlib-static=0|1     : When building zlib from sources, link static version of the library to freetype. [1]
  bzip2-static=0|1    : When building bzip2 from sources, link static version of the library to freetype. [1]
  libpng-static=0|1   : When building libpng from sources, link static version of the library to freetype. [1]""")
excons.AddHelpOptions(ext_zlib=excons.ExternalLibHelp("zlib"))
excons.AddHelpOptions(ext_bzip2=excons.ExternalLibHelp("bzip2"))
excons.AddHelpOptions(ext_libpng=excons.ExternalLibHelp("libpng"))
excons.DeclareTargets(env, prjs)


def FreetypeName():
    return "freetype"

def FreetypePath():
    name = FreetypeName()
    if sys.platform == "win32":
        libname = name + ".lib"
    else:
        libname = "lib" + name + (".a" if staticlib else excons.SharedLibraryLinkExt())
    return out_libdir + "/" + libname

def RequireFreetype(env):
    env.Append(CPPPATH=[out_incdir])
    env.Append(LIBPATH=[out_libdir])
    if staticlib:
        PngRequire(env)
        ZlibRequire(env)
        Bzip2Require(env)
    excons.Link(env, FreetypeName(), static=staticlib, force=True, silent=True)

Export("FreetypeName FreetypePath RequireFreetype")

