# CoWasm: Collaborative WebAssembly for Servers and Browsers

URL: https://github.com/sagemathinc/cowasm

# Welcome 

CoWasm, or "Collaborative WebAssembly," goes far beyond just Python. The collaboration aspects will be part of [CoCalc](https://cocalc.com) at some point in the future. CoWasm will support various technologies (such as libgit2 and realtime sync) that are important foundations for collaboration.

The underlying software components that CoWasm is built on (i.e., those we didn't write) are mostly extremely stable and mature. Zig is less stable, but we mostly use Zig for its amazing cross-compilation support and packaging of clang/llvm and musl-libc, which are themselves both very mature. Many other components, such as Python, Dash, and Numpy, are extremely mature, multi-decade-old projects. Moreover, other components of CoWasm, such as memfs, are libraries with 10M+ downloads per week that are heavily used in production.

The goal of CoWasm is overall somewhat similar to emscripten, [WebAssembly.sh](http://WebAssembly.sh), [wapm.io](http://wapm.io), and [Pyodide](https://pyodide.org/en/stable/) combined, in various ways:

- Unlike [WebAssembly.sh](http://WebAssembly.sh) and [wapm.io](http://wapm.io) (but similar to [Pyodide](https://pyodide.org/en/stable/)), we make **very heavy use of shared dynamic libraries** (e.g., `-fPIC` code), which is only possible because of a plugin contributed from emscripten to LLVM. The "Co" in CoWasm suggests "collaboration" or "sharing," which also reflects how the binaries in this project are structured.
- We use actual editline (similar to readline) instead of a JavaScript terminal. Moreover, unlike other WebAssembly shells, we use a real command-line shell (dash = Debian Almquist Shell). We also have a user space including ports of many coreutils, e.g., `ls`, `head`, `tail`, etc.
- Unlike emscripten, we use modern TypeScript, our code is more modular, and we make use of existing components when possible (e.g., the Node.js memfs project), instead of writing our own.
- A core design constraint is to efficiently run on a wide range of platforms-not just in the browser like emscripten, and not just on servers like wasmer. CoWasm should run on servers, desktops (e.g., as an Electron app), iPad/iOS apps, and in web browsers.
- **There is no business-unfriendly GPL code in CoWasm.** CoWasm itself is extremely liberally licensed and business-friendly. The license of all new code and most components is 3-clause BSD. CoWasm will serve as a foundation for other projects with more restrictive licenses:
  - CoCalc will build on top of CoWasm to provide a graphical interface and real-time collaboration, and that will be a commercial product.

  - Products like GP/PARI SageMath will build on CoWasm to provide GPL-licensed mathematics software.

 



## Packages

There are several subdirectories that each contain packages:
- **core** – Core functionality, including dynamic linking, a WASI implementation, OpenSSL, and nontrivial POSIX extensions to Node.js written in Zig. (This is analogous to [emscripten.org](https://emscripten.org/).)
- **python** – A WebAssembly build of Python, along with some nontrivial scientific libraries. (This is analogous to [pyodide.org](https://pyodide.org).)
- **web** – Examples of how to use the other packages in web applications; this includes building https://cowasm.org.
- **desktop** – A native Electron app that provides a sandboxed WebAssembly Python terminal running on your native filesystem.
- **sagemath** – The beginning of a port of https://sagemath.org to WebAssembly. Vast amounts of work remain.



### Python

An exciting package in CoWasm is [python-wasm](https://www.npmjs.com/package/python-wasm), which is a build of Python for WebAssembly that supports both servers and browsers. It also supports extension modules such as numpy. See [python/README.md](python/README.md) for more details.

<!--
[<img src="https://github.com/sagemathinc/cowasm/actions/workflows/docker-image.yml/badge.svg"  alt="Docker Image CI"  width="172px"  height="20px"  style="object-fit:cover"/>](https://github.com/sagemathinc/cowasm/actions/workflows/docker-image.yml)
-->

## Build from Source

We support and regularly test building CoWasm from source on the following platforms:

- x86 macOS and aarch64 macOS (Apple Silicon M1)
- x86 and aarch64 Linux

### Prerequisites

You need Node.js version at least 16.x, pnpm and several standard dev tools listed below. The dependency you need for every possible package are as follows:

- On **MacOS**, install the [XCode command line tools.](https://developer.apple.com/xcode/resources/)

- On **Linux apt\-based system, e.g., on Ubuntu 22.04**:

```sh
sudo apt-get update && sudo apt-get install git make cmake curl dpkg-dev m4 yasm texinfo python-is-python3 libtool tcl zip libncurses-dev
```

If you also want to install node v18 and pnpm on Ubuntu, you could do:

```sh
     sudo curl -sL https://deb.nodesource.com/setup_18.x | bash - \
  && sudo apt-get install -y nodejs \
  && curl -fsSL https://get.pnpm.io/install.sh | sh -
```

- On **Linux RPM based system, e.g., Fedora 37:**

```sh
dnf install git make cmake curl dpkg-dev m4 yasm texinfo libtool tcl zip ncurses-devel perl
```

- On **ArchLinux**:

```sh
pacman -Sy binutils git nodejs npm cmake curl m4 yasm texinfo python libtool tcl zip unzip patch binutils diffutils
```

NOTE: `pnpm` is not in the Arch Linux official package sets. You may want to install it from Arch User Repository (AUR) [AUR (en) - pnpm](https://aur.archlinux.org/packages/pnpm).

- Currently, the only way to build CoWasm from source on MS Windows is to use a Docker container running Linux. Using WSL2 \(maybe\) works but is too slow.

In addition you need to [install Node.js version at least 16.x](https://nodejs.org/en/download/) and [install the pnpm package manager](https://pnpm.io/installation).

#### Notes about Zig

_**You do NOT need to install Zig,**_ and it doesn't matter if
you have a random version of Zig on your system already. A zig binary is download automatically. We use zig \(instead of any system\-wide clang, etc. compilers\) for building all compiled code and write some code in the zig language. Since zig is fairly unstable it is critical to use the exact version that we provide.

### Build

To build any package in the `src/packages` directory, cd into that directory, then:

```sh
make
```

That's it, usually. You do not have to run build at the top level and you can start with building any specific package \-\- it will automatically cause any required dependencies to get installed or built.

You can also force building of every single package and running its test suite if you want:

```sh
~/cowasm$ make test
...
##########################################################
#                                                        #
#   CONGRATULATIONS -- FULL COWASM TEST SUITE PASSED!    #
#
#   Fri Oct 28 12:32:19 AM UTC 2022
#   Linux aarch64
#   Git Branch: main
#                                                        #
##########################################################
```

Depending on your computer, the build should take less than 30 minutes, and about 6GB's of disk space.

This installs a specific tested version of Zig and Nodejs, then builds native and WebAssembly versions of CPython and many dependencies, and also builds all the Typescript code. It also builds many other interesting programs with ports to WebAssembly, though many are not completely finished \(e.g., there is the dash shell and ports of tar and BSD coreutils\). As mentioned, building from source is regularly _**tested on Linux and MacOS with both x86_64 and ARM \(M1\) processors**_:

- Linux: tested on both x86_64 and aarch64 Ubuntu
- MacOS: tested on both x86_64 and M1 mac with standard XCode command live dev tools installed.

CoWasm _**does not**_ use the compilers on the system, and instead uses clang/llvm as shipped with Zig. If you're using Windows, you'll have to use Linux via a virtual machine or Docker.

### Pull latest code, build and test

At the top level run `./bin/rebuild-all` :

```sh
~/cowasm$ ./bin/rebuild-all
```

This does `make clean,` pulls the latest code, then runs the full build and test suite. Fortunately, `zig` caches a lot of build artifacts, so subsequent builds are faster.

**NOTE/WARNING:** Zig's cache is in `~/.cache/zig` and it can get HUGE. As far as I can tell, I think it just grows and grows without bound \(it's not an LRU cache\), and I think there are no tools to "manage" it besides just `rm -rf` it periodically.

#### What is tested?

Note that running `make test` at the top level does NOT run the **full** test suite of every package, since it takes quite a while and there are **still some failing tests**, since CoWasm doesn't support enough of what Python expects. It does run a large supported subset of the cpython test suite \(it's the part that I got to pass so far, which is over 80%\). As an example, the sympy test suite is massive, takes a very long time to run, and doesn't even work for me natively; instead, we just run a handful of small tests to ensure sympy is working at all. Similarly, for Cython, we run all their demos, but not the full test suite. A longer term goal of CoWasm is to support a second more thorough testing regime that runs the full test suite of each package. There will likely always be issues due to WASM not being a multiuser POSIX system, but it's good to know what those issues are!

You can also use the WebAssembly Python REPL directly on the command line.

```sh
~/cowasm$ ./bin/python-wasm 
Python 3.11.0 (main, Oct 27 2022, 10:03:11) [Clang 15.0.3 (git@github.com:ziglang/zig-bootstrap.git 0ce789d0f7a4d89fdc4d9571 on wasi
Type "help", "copyright", "credits" or "license" for more information.
>>> 2 + 3
5
>>> import sys
>>> sys.platform
'wasi'
>>> sys.executable
'/Users/wstein/build/cocalc/src/data/projects/2c9318d1-4f8b-4910-8da7-68a965514c95/cowasm/core/cpython/bin/python-wasm'
>>> ^D
~/cowasm$
```

The above directly runs the \`python.wasm\` executable produced by building cPython. You can instead run an enhanced version \(e.g., with signal support\) with more options in the python\-wasm package:

```py
~/cowasm$ . bin/env.sh 
~/cowasm$ cd core/python-wasm/
~/cowasm/core/python-wasm$ ./bin/python-wasm 
Python 3.11.0 (main, Oct 27 2022, 10:03:11) [Clang 15.0.3 (git@github.com:ziglang/zig-bootstrap.git 0ce789d0f7a4d89fdc4d9571 on wasi
Type "help", "copyright", "credits" or "license" for more information.
>>> import time; t=time.time(); print(sum(range(10**7)), time.time()-t)
49999995000000 0.8989999294281006
>>> ^D
~/cowasm/core/python-wasm$
```

As mentioned above, you can use python\-wasm as a library in node.js. There is a synchronous api that runs in the same thread as the import, and an asynchronous api that runs in a worker thread.

```py
~/cowasm$ . bin/env.sh 
~/cowasm$ cd core/python-wasm/
~/cowasm/core/python-wasm$ node
Welcome to Node.js v19.0.0.
Type ".help" for more information.
> python = require('.')
{
  syncPython: [AsyncFunction: syncPython],
  asyncPython: [AsyncFunction: asyncPython],
  default: [AsyncFunction: asyncPython]
}
> const { exec, repr} = await python.asyncPython();
undefined
> await repr('31**37')
'15148954872646847105498509334067131813327318808179940511'
> await exec('import time; t=time.time(); print(sum(range(10**7)), time.time()-t)')
49999995000000 0.8880000114440918
```

And yes you can run many async Python's in parallel in the same node.js process, with each running in its own thread:

```sh
~/cowasm/core/python-wasm$ nodeWelcome to Node.js v19.0.0.
Type ".help" for more information.
> python = require('.')
{
  syncPython: [AsyncFunction: syncPython],
  asyncPython: [AsyncFunction: asyncPython],
  default: [AsyncFunction: asyncPython]
}
> const { exec, repr} = await python.asyncPython();
undefined
> await exec('import time; t=time.time(); print(sum(range(10**7)), time.time()-t)')
49999995000000 0.8880000114440918
undefined
>
> const v = [await python.asyncPython(), await python.asyncPython(), await python.asyncPython()]; 0;
0
> s = 'import time; t=time.time(); print(sum(range(10**7)), time.time()-t)'
'import time; t=time.time(); print(sum(range(10**7)), time.time()-t)'
> d = new Date(); console.log(await Promise.all([v[0].exec(s), v[1].exec(s), v[2].exec(s)]), new Date() - d)
49999995000000 0.8919999599456787
49999995000000 0.8959999084472656
49999995000000 0.8929998874664307
[ undefined, undefined, undefined ] 905
undefined
```

## What's the goal?

Our **initial goal** is to create a WebAssembly build of the core Python and dependent packages, which runs both on the command line with Node.js and in the major web browsers \(via npm modules that you can include via webpack\). This is done. It should also be relatively easy to _build from source_ on both Linux and MacOS \(x86_64 and aarch64\) and to easily run the cpython test suite,\_ with a clearly defined supported list of passing tests. The compilation system is based on [Zig](https://ziglang.org/), which provides excellent caching and cross compilation, and each package is built using make.

## How does CoWasm compare to Emscripten and Pyodide?

Pyodide currently provides far more packages. However, there is no reason that CoWasm couldn't eventually support as much or more than Pyodide.

Our main longterm application is to make [CoCalc](https://cocalc.com) available on a much wider range of computers. As such, we are building a foundation here on which to support a substantial part of the scientific Python ecosystem and the [SageMath packages](https://www.sagemath.org/) \(a pure math analogue of the scientific Python stack\). _**I'm the founder of SageMath**_, hence this motivation \(some relevant work has been done [here\)](https://github.com/sagemathinc/jsage).

Some of our code will be written in the [Zig](https://ziglang.org/) language. However, we are mostly targeting just the parts that are used for Python, which is a small subset of the general problem. Our software license \-\- _**BSD 3\-clause**_ \-\- is compatible with their's and we hope to at least learn from their solutions to problems.

[More about how Pyodide and python\-wasm differ...](./docs/differences-from-pyodide.md)

## More about building from source

### How to build

Just type make. \(Do **NOT** type `make -j8;` it probably won't work.\)

```sh
...$ make
```

This runs a single top level Makefile that builds all the packages. The build process for each individual package is _also_ accomplished using a Makefile with two includes that impose some structure. We don't use shell scripts or Python code to orchestrate building anything, since `make` is much cleaner and easier to read, maintain and debug... and of course make contains shell scripts in it. \(History lesson: homebrew is a lot more successful than Macports.\)

### What happens

In most subdirectories `foo/` of `core`, this makefile creates some subdirectories:

- `core/foo/dist/[native|wasm]` \-\- a native or WebAssembly build of the package; this has binaries, headers, and libs. These get used by other packages. We rarely build the native version.
- `core/build/[native|wasm]` \- build artifacts for the native or WebAssembly build; can be safely deleted

### No common prefix directory

Unlike some systems, where everything is built and installed into a single `prefix` directory, here we build everything in its own self\-contained package dist directory. When a package like `cpython` depends on another package like `lzma` , our Makefile for `cpython` explicitly references `packages/lzma/dist`. This makes it easier to uninstall packages, update them, etc., without having to track what files are in any package, whereas using a common directory for everything can be a mess with possibly conflicting versions of files, and makes versioning and dependencies very explicit. Of course, it makes the environment variables and build commands potentially much longer. In some cases, we gather together files from these dist directories in distributions, e.g., see `make bin-wasm.`

### Native and Wasm

The build typically create directories `dist/native`and `dist/wasm.` The `dist/native` artifacts are only of value on the computer where you ran the build, since they are architecture dependent and can easily depend on libraries on your system. In contrast, the `dist/wasm` artifacts are platform independent. They can be used nearly everywhere: on servers via WASM, on ARM computers \(e.g., aarch64 linux, Apple Silicon, etc.\), and in modern web browsers.

### Standalone WASM executables

The bin directory has scripts `zcc` and `z++` that are C and C\+\+ compiler wrappers around Zig \+ Node. They create binaries that you can run on the command line as normal. Under the hood there's a wrapper script that calls node.js and the wasi runtime.

```sh
$ . bin/env.sh
$ echo 'int main() { printf("hello from Web Assembly: %d\n", 2+2); }' > a.c
$ zcc a.c
$ ls -l
a.c  a.out  a.out.wasm  ...
$ ./a.out   # this actually runs nodejs + python-wasm
hello from Web Assembly: 4
```

This isn't currently used here for building python-wasm, but it's an extremely powerful tool. \(For example, I used it with JSage to cross compile the NTL library to Web Assembly...\)

### Run a script from the terminal:

```sh
~/cowasm$ echo "import sys; print(f'hi from {sys.platform}')" > a.py
~/cowasm$ bin/python-wasm a.py
hi from wasi
```

The python-wasm package has a bin/python-wasm script that can run
Python programs that including interactive blocking input:

```sh
~/cowasm/core/python-wasm$ echo "name = input('name? '); print(name*3)" > a.py
~/cowasm/core/python-wasm$ ./bin/python-wasm a.py
name? william
williamwilliamwilliam
```

DEMOS:

- https://cowasm.org \- Python using SharedArrayBuffers
- https://zython.org \- Python using Service Workers
- https://cowasm.sh \- Dash Shell with Python, Sqlite, Lua, etc., using SharedArrayBuffers


---

## Contact

Email [wstein@cocalc.com](mailto:wstein@cocalc.com) or [@wstein389](https://twitter.com/wstein389) if find this interesting and want to help out.




**CoWasm is an open source 3\-clause BSD licensed project. It includes components and dependencies that may be licensed in other ways, but nothing is GPL licensed, *except* some code in the sagemath subdirectory (which nothing else depends on).**

