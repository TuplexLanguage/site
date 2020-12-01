---
layout: default
comments: true
---
## How to Build the Tuplex Compiler

The current working state of the Tuplex compiler is availabile on GitHub:

<a href="https://github.com/TuplexLanguage/tuplex" target="_blank">https://github.com/TuplexLanguage/tuplex</a>

So far the build has only been tested on Linux/Ubuntu. If you have a Ubuntu system or are an experienced developer it should be straight-forward to get it to build.

These are the prerequisite tools, and the version it currently builds with. There is a script below (also included in the repo) to automatically install them for you if you are on an Ubuntu system. The script uses apt-get for most of them; the llvm 5 installation is a bit more involved but is also handled by the script.

* cmake 3.16.3
* make 4.2.1
* bison 3.5.1
* c++17 compilation tool chain (it's been tested with gcc and clang)
* python3, if you want to run the test suite
* llvm 11.0.1

### Install and build

This is an install-from-scratch command sequence that has been tested on a completely fresh Ubuntu 16.x installation.

The PATH settings should work as-is, but you may want to make the paths absolute and put them into your .bashrc or similar.

**Step 0:** If you don't have git installed you need to do that first:

```
sudo apt update
sudo apt install git
```

**Step 1:** cd to the directory you want to put the "tuplex" repository directory and run this:

```
# Clone the tuplex repository to the current directory:
git clone https://github.com/TuplexLanguage/tuplex.git
```

**Step 2:**
The following script ensures the necessary build tools are installed and then builds Tuplex.
Either copy-paste it into your console, or run as a bash script.
It is in the repo and can be run directly from the command line: `tuplex/compiler/scripts/txinstall`

> Check [https://apt.llvm.org/](https://apt.llvm.org/) if you have trouble.

```
#/bin/bash
# Ensures the necessary build tools are installed and then builds tuplex.
# This script is in the repo and can be run directly:  tuplex/compiler/scripts/txinstall

set -x  # command echo on

###################
## Install the packages with the build tools needed to build the tuplex compiler:
sudo apt update
sudo apt install cmake make gcc g++ bison


###################
## Install LLVM 11, also needed to build Tuplex:

# Required for add-apt-repository (needed for LLVM installation):
sudo apt install software-properties-common

# Install LLVM 11:
# !!!!! Note: Check https://apt.llvm.org/ for different Linux flavors and LLVM versions, and some installation tips
apt-get install libllvm-11-ocaml-dev libllvm11 llvm-11 llvm-11-dev llvm-11-doc llvm-11-examples llvm-11-runtime
```

**Step 3:** Build tuplex:

```
# Add LLVM 11 to the command path:
# (To make this permament for all bash sessions, it can be appended to ~/.bashrc or similar file.)
export PATH=$PATH:/usr/lib/llvm-11/bin

# cd to the compiler's directory:
cd tuplex/compiler

# create a build directory (for out-of-source building):
mkdir build
cd build

# build it:
cmake ..
make
```

> When building Tuplex, if any tool, library or include isn't found, see if the paths in `compiler/src/CMakeLists.txt` need to be adapted for your setup.

I had some trouble myself to make it work with CLion, my IDE, so I had to set this line to my LLVM install path:
set(LLVM_CFG_CMD "/usr/lib/llvm-11/bin/llvm-config")

After this file has been changed, you need to run `cmake ..` again followed by `make`.

**Step 4:** Add command paths:

To add tuplex commands (including the `txc` compiler) and scripts to the path:
<br>(You may want to do this with absolute paths in your ~/.bashrc)

```
export PATH=$PATH:$PWD/scripts

# optional:
export PATH=$PATH:$PWD/bin
```

Note that there is a wrapper script called scripts/txc which attempts to locate your current build's txc executable. If your current working directory is the build directory it will work just fine even without the bin directory in the path.

### Trying it out

Suggestions for starting testing the compiler:

Run the test suite:

    txts

Compile the Hello, world! program from the test suite and run it directly in JIT mode:

    txc autotest/lib/helloworld.tx -nobc -jit

Run the Tuplex program build script (produces an executable):

    txb autotest/lib/helloworld.tx

---

### Files of interest

These might be some files you'd like to check out:

`<builddir>/bin/txc` The compiler. You may want to add this to your path. *Run with -h to print the command line usage.* See also the Paths section below.

`compiler/autotest/lib/helloworld.tx` An example program that prints "Hello, world!".

There are also a bunch of other test programs in `compiler/autotest/`, this is the suite of test source files used in the automated tests of the compiler (i.e. unit tests). These have plenty of syntax examples covering the implemented capabilities of the language.

`compiler/tx` The Tuplex foundation library's source code (under the reserved namespace `tx`). This contains quite advanced code and ties together the built-in types with interfaces and implementations for collections, iteration, I/O, and more.


### Paths

The txc compiler has two special paths that are controlled by command options and used to locate source modules.

* The foundation library code: The `tx` module and some of its submodules. These are expected to be found in a directory named `tx` in the current directory, or in a location specified by the `-tx <path>` option. If you run the compiler with `compiler/` as the current directory you can skip this option since it is located there.

* The user source code path(s). These are searched to find the source code for imported modules. If not set then the current directory is default. Set using the `-sp <pathlist>` or `-sourcepath <pathlist>` options.


### Scripts

There are some ready-made scripts in the `compiler/scripts/` directory. If you add it to your path you can run them directly.

* `txb`
Build script that runs the compiler, the LLVM optimizer and linker to produce a stand-alone executable. Command line args are forwarded to txc.

* `txts`
Runs the test suite.

* `copyrelease`
Intended to be run in the build directory, after a successful build. Copies binaries, scripts and library sources and makes a release tarball.
