import os
import sys
import pprint
import excons


env = excons.MakeBaseEnv()

out_incdir = excons.OutputBaseDirectory() + "/include"
out_libdir = excons.OutputBaseDirectory() + "/lib"

staticlib = (excons.GetArgument("freetype-static", 1, int) != 0)
with_zlib = excons.GetArgument("freetype-zlib", 1, int)
with_bzip2 = excons.GetArgument("freetype-bzip2", 1, int)
with_libpng = excons.GetArgument("freetype-libpng", 1, int)

cmake_opts = {"BUILD_SHARED_LIBS" : (0 if staticlib else 1), # not supported on windows with msvc
              "WITH_ZLIB": with_zlib,
              "WITH_BZIP2": with_bzip2,
              "WITH_PNG": with_libpng,
              "WITH_HARFBUZZ": 0}

cfg_deps = []
overrides = {}

# ZLIB Setup ===================================================================

if with_zlib:
  def ZlibLibname(static):
     return ("z" if sys.platform != "win32" else ("zlib" if static else "zdll"))

  def ZlibDefines(static):
     return ([] if static else ["ZLIB_DLL"])

  rv = excons.cmake.ExternalLibRequire(cmake_opts, name="zlib", libnameFunc=ZlibLibname, definesFunc=ZlibDefines)
  if rv is None:
     excons.PrintOnce("freetype: Build zlib from sources ...")
     excons.Call("zlib", overrides=overrides, imp=["RequireZlib", "ZlibName", "ZlibPath"])

     cfg_deps.append(excons.cmake.OutputsCachePath("zlib"))

     zlibstatic = excons.GetArgument("zlib-static", 1, int)
     zlibpath = ZlibPath(static=zlibstatic)

     cmake_opts["ZLIB_LIBRARY"] = zlibpath
     cmake_opts["ZLIB_INCLUDE_DIR"] = out_incdir

     def ZlibRequire(env):
        RequireZlib(env, static=zlibstatic)

     # Setup overrides for other subprojects
     overrides["with-zlib"] = os.path.dirname(os.path.dirname(zlibpath))
     overrides["zlib-static"] = zlibstatic
     overrides["zlib-libname"] = ZlibName(static=zlibstatic)

  else:
     ZlibRequire = rv

# BZIP2 Setup ===================================================================

if with_bzip2:
  def Bzip2Libname(static):
     return ("bz2" if sys.platform != "win32" else "libbz2")

  def Bzip2Defines(static):
     return ([] if static else ["BZ_DLL"])

  rv = excons.cmake.ExternalLibRequire(cmake_opts, name="bzip2", libnameFunc=Bzip2Libname, definesFunc=Bzip2Defines)
  if rv is None:
     excons.PrintOnce("freetype: Build bzip2 from sources ...")
     excons.Call("bzip2", overrides=overrides, imp=["RequireBzip2", "Bzip2Name", "Bzip2Path"])

     libpath = Bzip2Path()
     cfg_deps.append(libpath)

     bz2static = excons.GetArgument("bzip2-static", 1, int)

     cmake_opts["BZIP2_LIBRARY_RELEASE"] = libpath
     cmake_opts["BZIP2_LIBRARY_DEBUG"] = libpath
     cmake_opts["BZIP2_INCLUDE_DIR"] = out_incdir

     def Bzip2Require(env):
        RequireBzip2(env)

     overrides["with-bzip2"] = os.path.dirname(os.path.dirname(libpath))
     overrides["bzip2-static"] = bz2static
     overrides["bzip2-libname"] = Bzip2Name()

  else:
     Bzip2Require = rv

# LIBPNG Setup ===================================================================

if with_libpng:
  def PngLibname(static):
     return ("bz2" if sys.platform != "win32" else "libbz2")

  rv = excons.cmake.ExternalLibRequire(cmake_opts, name="libpng", libnameFunc=PngLibname, varPrefix="PNG_")
  if rv is None:
     excons.PrintOnce("freetype: Build libpng from sources ...")
     excons.Call("libpng", overrides=overrides, imp=["RequireLibpng", "LibpngName", "LibpngPath"])

     cfg_deps.append(excons.cmake.OutputsCachePath("libpng"))

     pngstatic = excons.GetArgument("libpng-static", 1, int)
     libpath = LibpngPath(static=pngstatic)

     cmake_opts["PNG_LIBRARY"] = libpath
     cmake_opts["PNG_INCLUDE_DIR"] = out_incdir

     def PngRequire(env):
        RequireLibpng(env, static=True)

     overrides["with-libpng"] = os.path.dirname(os.path.dirname(libpath))
     overrides["libpng-libname"] = LibpngName(static=pngstatic)

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
  freetype-static=0|1  : Toggle between static and shared library build. [1]
  freetype-zlib=0|1    : Use zlib. [1]
  freetype-bzip2=0|1   : Use bzip2. [1]
  freetype-libpng=0|1  : Use libpng. [1]""")
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
        if with_libpng:
            PngRequire(env)
        if with_zlib:
            ZlibRequire(env)
        if with_bzip2:
            Bzip2Require(env)
    excons.Link(env, FreetypeName(), static=staticlib, force=True, silent=True)

Export("FreetypeName FreetypePath RequireFreetype")
