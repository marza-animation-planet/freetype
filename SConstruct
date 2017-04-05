import excons
import sys
import os


env = excons.MakeBaseEnv()


staticlib = (excons.GetArgument("freetype-static", 1, int) != 0)
out_incdir = excons.OutputBaseDirectory() + "/include"
out_libdir = excons.OutputBaseDirectory() + "/lib"


prjs = []


prjs.append({"name": "freetype",
             "type": "cmake",
             "cmake-opts": {"BUILD_SHARED_LIBS": (0 if staticlib else 1)},
             "cmake-srcs": excons.CollectFiles("freetype", patterns=["*.c"], recursive=True)})

excons.AddHelpOptions(libtiff="""FREETYPE OPTIONS
  freetype-static=0|1  : Toggle between static and shared library build [1]""")


excons.DeclareTargets(env, prjs)


def RequireFreetype(env):
    env.Append(CPPPATH=[out_incdir])
    env.Append(LIBPATH=[out_libdir])
    if staticlib:
        if sys.platform == "win32":
            env.Append(LIBS=["freetype"])
        else:
            if not env.StaticallyLink(env, "freetype", silent=True):
                env.Append(LIBS=["freetype"])
    else:
        env.Append(LIBS=["freetype"])


def FreetypeName():
    if sys.platform == "win32":
        basename = "freetype.lib"
    else:
        basename = "libfreetype.a"

    return out_libdir + "/" + basename


Export("RequireFreetype FreetypeName")