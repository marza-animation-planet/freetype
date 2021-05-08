import os
import sys
import pprint
import excons
import SCons.Script # pylint: disable=import-error


env = excons.MakeBaseEnv()

out_incdir = excons.OutputBaseDirectory() + "/include"
out_libdir = excons.OutputBaseDirectory() + "/lib"

staticlib = (excons.GetArgument("freetype-static", 1, int) != 0)

cmake_opts = {"BUILD_SHARED_LIBS" : (0 if staticlib else 1), # not supported on windows with msvc
              "FT_WITH_ZLIB": 1,
              "FT_WITH_BZIP2": 1,
              "FT_WITH_PNG": 1,
              "FT_WITH_HARFBUZZ": 0,
              "CMAKE_INSTALL_LIBDIR": "lib"}

cfg_deps = []
libpng_overrides = {}
libpng_deps = []

# ZLIB Setup ===================================================================

def ZlibLibname(static):
    return ("z" if sys.platform != "win32" else ("zlib" if static else "zdll"))

def ZlibDefines(static):
    return ([] if (static or sys.platform != "win32") else ["ZLIB_DLL"])

rv = excons.cmake.ExternalLibRequire(cmake_opts, name="zlib", libnameFunc=ZlibLibname, definesFunc=ZlibDefines)
if rv["require"] is None:
    excons.PrintOnce("freetype: Build zlib from sources ...")
    excons.Call("zlib", targets=["zlib"], imp=["RequireZlib", "ZlibName", "ZlibPath"])

    zlibstatic = excons.GetArgument("zlib-static", 1, int)
    zlibpath = ZlibPath(static=zlibstatic) # pylint: disable=undefined-variable

    cfg_deps.append(zlibpath)
    libpng_deps.append(zlibpath)

    cmake_opts["ZLIB_LIBRARY"] = zlibpath
    cmake_opts["ZLIB_INCLUDE_DIR"] = out_incdir

    def ZlibRequire(env):
        RequireZlib(env, static=zlibstatic) # pylint: disable=undefined-variable

    # Setup overrides for libpng subprojects
    libpng_overrides["with-zlib"] = os.path.dirname(os.path.dirname(zlibpath))
    libpng_overrides["zlib-static"] = zlibstatic
    libpng_overrides["zlib-name"] = ZlibName(static=zlibstatic) # pylint: disable=undefined-variable
else:
    ZlibRequire = rv["require"]

# BZIP2 Setup ===================================================================

def Bzip2Libname(static):
    return ("bz2" if sys.platform != "win32" else "libbz2")

def Bzip2Defines(static):
    return ([] if (static or sys.platform != "win32") else ["BZ_DLL"])

rv = excons.cmake.ExternalLibRequire(cmake_opts, name="bz2", libnameFunc=Bzip2Libname, definesFunc=Bzip2Defines, varPrefix="BZIP2_")
if rv["require"] is None:
    excons.PrintOnce("freetype: Build bzip2 from sources ...")
    excons.Call("bzip2", targets=["bz2"], imp=["RequireBZ2", "BZ2Name", "BZ2Path"])

    bz2static = (excons.GetArgument("bz2-static", 1, int) != 0)
    bz2path = BZ2Path() # pylint: disable=undefined-variable

    cfg_deps.append(bz2path)

    cmake_opts["BZIP2_LIBRARY_RELEASE"] = bz2path
    cmake_opts["BZIP2_LIBRARY_DEBUG"] = bz2path
    cmake_opts["BZIP2_INCLUDE_DIR"] = out_incdir

    def Bzip2Require(env):
        RequireBZ2(env) # pylint: disable=undefined-variable

else:
    Bzip2Require = rv["require"]

# LIBPNG Setup ===================================================================

def PngLibname(static):
    return ("png" if sys.platform != "win32" else "libpng")

def PngDefines(static):
    return (["PNG_USE_DLL"] if (not static and sys.platform == "win32") else [])

rv = excons.cmake.ExternalLibRequire(cmake_opts, name="libpng", libnameFunc=PngLibname, definesFunc=PngDefines, varPrefix="PNG_")
if rv["require"] is None:
    excons.PrintOnce("freetype: Build libpng from sources ...")
    excons.cmake.AddConfigureDependencies("libpng", libpng_deps)
    excons.Call("libpng", targets=["libpng"], overrides=libpng_overrides, imp=["RequireLibpng", "LibpngName", "LibpngPath"])

    pngstatic = (excons.GetArgument("libpng-static", 1, int) != 0)
    pngpath = LibpngPath(static=pngstatic) # pylint: disable=undefined-variable

    cfg_deps.append(pngpath)

    cmake_opts["PNG_LIBRARY"] = pngpath
    cmake_opts["PNG_INCLUDE_DIR"] = out_incdir

    def PngRequire(env):
        RequireLibpng(env, static=pngstatic) # pylint: disable=undefined-variable

else:
    PngRequire = rv["require"]


# Freetype library ===================================================================

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
    env.Append(CPPPATH=[out_incdir + "/freetype2"])
    env.Append(LIBPATH=[out_libdir])
    excons.Link(env, FreetypePath(), static=staticlib, force=True, silent=True)
    if staticlib:
        PngRequire(env)
        ZlibRequire(env)
        Bzip2Require(env)

prjs = [
    {   "name": FreetypeName(),
        "type": "cmake",
        "cmake-opts": cmake_opts,
        "cmake-cfgs": ["./CMakeLists.txt"] + cfg_deps,
        "cmake-srcs": excons.CollectFiles(".", patterns=["*.c"], recursive=True),
        "cmake-outputs": map(lambda x: "include/freetype2/freetype/%s" % os.path.basename(x), excons.glob("include/freetype/*.h")) +
                         ["include/freetype2/ft2build.h", FreetypePath()]
    }
]

excons.AddHelpOptions(freetype="""FREETYPE OPTIONS
  freetype-static=0|1 : Toggle between static and shared library build. [1]
  zlib-static=0|1     : When building zlib from sources, link static version of the library to freetype. [1]
  bz2-static=0|1      : When building bzip2 from sources, link static version of the library to freetype. [1]
  libpng-static=0|1   : When building libpng from sources, link static version of the library to freetype. [1]""")
excons.DeclareTargets(env, prjs)

SCons.Script.Export("FreetypeName FreetypePath RequireFreetype")
