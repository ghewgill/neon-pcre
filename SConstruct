import os
import shutil
import sys
import tarfile

env = Environment()

libpcre = None

pcreenv = Environment()
if "RELEASE" not in env or not env["RELEASE"]:
    if sys.platform == "win32":
        pcreenv.Append(CFLAGS=[
            "/MTd",
            "/Zi",
            "/Od",
        ])

if GetOption("clean"):
    shutil.rmtree("pcre2-10.10", ignore_errors=True)
elif not os.path.exists("pcre2-10.10/configure"):
    tarfile.open("pcre2-10.10.tar.gz").extractall(".")
    if sys.platform == "win32":
        shutil.copyfile("pcre2-10.10/src/config.h.generic", "pcre2-10.10/src/config.h")
        shutil.copyfile("pcre2-10.10/src/pcre2.h.generic", "pcre2-10.10/src/pcre2.h")

if sys.platform == "win32":
    pcreenv.Append(CPPFLAGS=[
        "-DHAVE_CONFIG_H",
        "-DPCRE2_CODE_UNIT_WIDTH=8",
        "-DPCRE2_STATIC",
    ])
    pcreenv.Command("pcre2-10.10/src/pcre2_chartables.c", "pcre2-10.10/src/pcre2_chartables.c.dist", lambda target, source, env: shutil.copyfile(source[0].path, target[0].path))
    libpcre = pcreenv.Library("pcre2-10.10/pcre2.lib", [
        "pcre2-10.10/src/pcre2_auto_possess.c",
        "pcre2-10.10/src/pcre2_chartables.c",
        "pcre2-10.10/src/pcre2_compile.c",
        "pcre2-10.10/src/pcre2_config.c",
        "pcre2-10.10/src/pcre2_context.c",
        "pcre2-10.10/src/pcre2_dfa_match.c",
        "pcre2-10.10/src/pcre2_error.c",
        "pcre2-10.10/src/pcre2_jit_compile.c",
        "pcre2-10.10/src/pcre2_maketables.c",
        "pcre2-10.10/src/pcre2_match.c",
        "pcre2-10.10/src/pcre2_match_data.c",
        "pcre2-10.10/src/pcre2_newline.c",
        "pcre2-10.10/src/pcre2_ord2utf.c",
        "pcre2-10.10/src/pcre2_pattern_info.c",
        "pcre2-10.10/src/pcre2_serialize.c",
        "pcre2-10.10/src/pcre2_string_utils.c",
        "pcre2-10.10/src/pcre2_study.c",
        "pcre2-10.10/src/pcre2_substitute.c",
        "pcre2-10.10/src/pcre2_substring.c",
        "pcre2-10.10/src/pcre2_tables.c",
        "pcre2-10.10/src/pcre2_ucd.c",
        "pcre2-10.10/src/pcre2_valid_utf.c",
        "pcre2-10.10/src/pcre2_xclass.c",
    ])
    env.Append(CPPPATH=["pcre2-10.10/src"])
else:
    conf = Configure(env)
    if not conf.CheckLibWithHeader("pcre2", "pcre2.h", "C++"):
        libpcre = pcreenv.Command("pcre2-10.10/.libs/libpcre2-8.a", "pcre2-10.10/configure", "cd lib/regex/pcre2-10.10 && env CFLAGS=-fPIC ./configure && make")
        env.Append(CPPPATH=["pcre2-10.10/src"])
    conf.Finish()

env.Append(CPPPATH=["../../src"])
env.Append(LIBS=[libpcre])
env.Depends("regex.cpp", libpcre) # Ensure that header files are created.
env.SharedLibrary("neon_regex", "regex.cpp")
