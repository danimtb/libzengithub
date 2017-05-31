# libzengithub - an example how to build portable [conan.io](https://www.conan.io/) C/C++ packages

libzengithub prints out a random [Zen of Github](http://ben.balter.com/2015/08/12/the-zen-of-github/) whenever you call its only function.

```c
#include <zengithub.h>

int main() {
    zen_of_github();
}
```

will provide you something like

```
               MMM.           .MMM
               MMMMMMMMMMMMMMMMMMM
               MMMMMMMMMMMMMMMMMMM      _________________________________________
              MMMMMMMMMMMMMMMMMMMMM    |                                         |
             MMMMMMMMMMMMMMMMMMMMMMM   | Anything added dilutes everything else. |
            MMMMMMMMMMMMMMMMMMMMMMMM   |_   _____________________________________|
            MMMM::- -:::::::- -::MMMM    |/
             MM~:~ 00~:::::~ 00~:~MM
        .. MMMMM::.00:::+:::.00::MMMMM ..
              .MM::::: ._. :::::MM.
                 MMMM;:::::;MMMM
          -MM        MMMMMMM
          ^  M+     MMMMMMMMM
              MMMMMMM MM MM MM
                   MM MM MM MM
                   MM MM MM MM
                .~~MM~MM~MM~MM~~.
             ~~~~MM:~MM~~~MM~:MM~~~~
            ~~~~~~==~==~~~==~==~~~~~~
             ~~~~~~==~==~==~==~~~~~~
                 :~==~==~==~==~~
```

See the [zenofgithub](https://github.com/jonico/zenofgithub) for a conan.io based application that makes use of this package.

# conan.io package dependencies

libzengithub is relying on [libcurl](https://github.com/lasote/conan-libcurl) to call the ```https://api.github.com/octocat``` endpoint and retrieve a random Zen of GitHub.
Its main purpose is to demonstrate how to build portable [conan.io](https://www.conan.io/) packages that rely on conan.io packages (libcurl) themselves.
conan.io is a great package manager for C and C++ libraries which works for various platforms including Linux, Mac OS and Windows.

The only thing needed for ```libzengithub``` to declare a dependency to ```libcurl``` is to declare that dependency in its [conanfile.py](https://github.com/jonico/libzengithub/blob/master/conanfile.py):

```python
...
class ZenGithubConan(ConanFile):
    name = "ZenGitHub"
    version = "1.0"
    license = "Apache 2.0"
    url = "https://github.com/jonico/libzengithub"
    settings = "os", "compiler", "build_type", "arch"
    options = {"shared": [True, False]}
    default_options = "shared=False"
    generators = "cmake"
    exports_sources = "zengithub/*"
    requires = "libcurl/7.50.3@lasote/stable"
...
```

conan will then automatically build the following dependency tree:

```
conan info . --graph deps.html
open deps.html
```

![image](https://cloud.githubusercontent.com/assets/1872314/26656503/385db5f8-4614-11e7-9ceb-8ff8cbb8527b.png)

and automatically downloads the packages it depends upon from the conan repository. If it should find pre-built packages for the dependencies, it will build them locally.

# Consuming the package

See [zenofgithub](https://github.com/jonico/zenofgithub) for an example application that consumes this library.

## Basic setup

First, [install](http://docs.conan.io/en/latest/installation.html) the conan.io package manager locally.
Then, add the remote to [my conan.io repo](https://api.bintray.com/conan/conan-jonico/libzengithub):

```
conan remote add conan-jonico https://api.bintray.com/conan/conan-jonico/libzengithub
```

Now, you can install the library:

`
conan install ZenGitHub/1.0@jonico/stable
`

If conan complains about missing dependencies (like libCurl or libz or OpenSSL), this is an indication that your current conan.io do not have a pre-built version of those dependent packages and you can build them locally instead:

`
conan install ZenGitHub/1.0@jonico/stable --build missing
`

If you rather want to install the shared library instead of a static library, use

`
conan install ZenGitHub/1.0@jonico/stable -o ZenGitHub:shared=True --build missing
`

## Project setup

If you handle multiple dependencies in your project is better to add a [conanfile.txt](https://github.com/jonico/zenofgithub/blob/master/conanfile.txt)

```
    [requires]
    ZenGitHub/1.0@jonico/stable

    [options]
    ZenGitHub:shared=False # True
    
    [generators]
    cmake

    [imports]
bin, *.dll -> ./bin # Copies all dll files from packages bin folder to my "bin" folder
lib, *.dylib* -> ./bin # Copies all dylib files from packages lib folder to my "bin" folder
```

Complete the installation of requirements for your project running:</small></span>

`
conan install .
`

Project setup installs the library (and all his dependencies) and generates the files *conanbuildinfo.txt* and *conanbuildinfo.cmake* with all the paths and variables that you need to link with your dependencies.

# Building, testing and installing locally

If you are a consumer of ```libzengithub```, you would not need to build this package by yourself but get it as part of your consumer build or pre-built from [my conan.io repo](https://api.bintray.com/conan/conan-jonico/libzengithub)
Checkout [zenofgithub](https://github.com/jonico/zenofgithub) for an example application that consumes this library.

If you still like to build, test and install ```libzengithub``` directly, clone this repository, [get the conan CLI](http://docs.conan.io/en/latest/installation.html) and run

```
conan test_package
```

## Upload packages to your own conan.io server

First of all, [configure the remote](http://conanio.readthedocs.io/en/latest/reference/commands/remote.html) to your local conan.io server.

```
conan remote add <your conan remote> <conan.io, artifactory or bintray url>
```

```
conan upload libzengithub/1.0@jonico/stable --all -r=<your conan remote>
```