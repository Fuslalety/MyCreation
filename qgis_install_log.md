# Try to building on Ubuntu 18.04
First install some tools you will need for this instructions:
- sudo apt install gnupg software-properties-common

Now install the QGIS Signing Key, so QGIS software from the QGIS repo will be trusted and installed:
- wget -qO - https://qgis.org/downloads/qgis-2021.gpg.key | sudo gpg --no-default-keyring --keyring gnupg-ring:/etc/apt/trusted.gpg.d/qgis-archive.gpg --import
- sudo chmod a+r /etc/apt/trusted.gpg.d/qgis-archive.gpg

Add the QGIS repo for the latest stable QGIS (3.20.x Odense).Note: "lsb_release -c -s" in those lines will return your distro name:
- sudo add-apt-repository "deb https://qgis.org/ubuntu $(lsb_release -c -s) main"

Update your repository information to reflect also the just added QGIS one:
- sudo apt update
Now, install QGIS:
- sudo apt install qgis qgis-plugin-grass python-qgis


# 4. Building on Windows

## 4.1. Building with Microsoft Visual Studio

This section describes how to build QGIS using Visual Studio (MSVC) 2015 on Windows.
This is currently also how the binary QGIS packages are made (earlier versions used MinGW).

This section describes the setup required to allow Visual Studio to be used to
build QGIS.

### 4.1.1. Visual Studio 2015 Community Edition

Download the [free (as in free beer) Community installer](https://download.microsoft.com/download/D/2/3/D23F4D0F-BA2D-4600-8725-6CCECEA05196/vs_community_ENU.exe)

Select "Custom" install and add the following packages:

* "Common Tools for Visual C++ 2015" under "Visual C++"
* "Tools (1.4.1) and Windows 10 SDK (10.0.14393)" under "Universal Windows App Development Tools".

### 4.1.2. Other tools and dependencies

Download and install following packages:

* [CMake](https://cmake.org/files/v3.12/cmake-3.12.3-win64-x64.msi)
* GNU flex, GNU bison and GIT with cygwin [32bit](https://cygwin.com/setup-x86.exe) or [64bit](https://cygwin.com/setup-x86_64.exe)
* OSGeo4W [32bit](https://download.osgeo.org/osgeo4w/osgeo4w-setup-x86.exe) or [64bit](https://download.osgeo.org/osgeo4w/osgeo4w-setup-x86_64.exe)
* [ninja](https://github.com/ninja-build/ninja/releases/download/v1.7.2/ninja-win.zip): Copy the `ninja.exe` to `C:\OSGeo4W64\bin\`

For the QGIS build you need to install following packages from cygwin:

* bison
* flex
* git (even if you already have Git for Windows installed)

and from OSGeo4W (select *Advanced Install*):

* qgis-dev-deps

  * This will also select packages the above packages depend on.

  * Note: If you install other packages, this might cause issues. Particularly, make sure
    **not** to install the msinttypes package. It installs a stdint.h file in
    OSGeo4W[64]\include, that conflicts with Visual Studio own stdint.h, which for
    example breaks the build of the virtual layer provider.

Earlier versions of this document also covered how to build all above
dependencies.  If you're interested in that, check the history of this page in the Wiki
or the SVN repository.

`****************************************`

`*************** bookmark ***************`

`*************STAYING IN HERE************`

`****************************************`

### 4.1.3. Clone the QGIS Source Code

Choose a directory to store the QGIS source code.
For example, to put it in the OSGeo4W64 install, navigate there:

```cmd
cd C:\OSGeo4W64
```

This directory will be assumed for all instructions
that follow.

On the command prompt clone the QGIS source from
git to the source directory `QGIS`:

```cmd
git clone git://github.com/qgis/QGIS.git
```

This requires Git. If you have Git for Windows on your PATH already,
you can do this from a normal command prompt. If you do not, you can
use the Git package that was installed as part of Cygwin by opening
a Cygwin[64] Terminal

And, to avoid Git in Windows reporting changes to files not actually modified:

```cmd
cd QGIS
git config core.filemode false
```

### 4.1.4. Configure and build with CMake from command line

**Note:** Consider this section as example.  It tends to outdate, when OSGeo4W and
SDKs move on.  `ms-windows/osgeo4w/package-nightly.cmd` is used for the
nightly builds and constantly updated and hence might contain necessary
updates that are not yet reflected here.

To start a command prompt with an environment that both has the VC++ and the OSGeo4W
variables create the following batch file (assuming the above packages were
installed in the default locations):

```cmd
@echo off
call C:\OSGeo4W64\QGIS\ms-windows\osgeo4w\msvc-env.bat x86_64
@cmd
```

Save the batch file as `C:\OSGeo4W64\OSGeo4W-dev.bat` and run it.

#### 4.1.4.1 Using configonly.bat to create the MSVC solution file
We will be using the file `ms-windows/osgeo4w/configonly.bat` to create an MSVC solution file.
There are two supported CMake generators for creating a solution file: Ninja, and native MSVC.
The advantage of using native MSVC solution is that you can find the root of build problems much more easily.
configonly.bat is meant to create a configured build directory with a MSVC solution file:

```cmd
cd C:\OSGeo4W64\QGIS\ms-windows\osgeo4w
configonly.bat
```

#### 4.1.4.2 Compiling QGIS with MSVC
We will need to run MSVC with all the environment variables set, thus we will run it as follows:
* Run the batch file OSGeo4W-dev.bat you created before.
* On the command prompt run `call gdal-dev-env.bat` to add the release gdal and proj libraries to your PATH.
* On the command prompt run `devenv` to open MSVC.
* From MSVC, open the solution file `C:\OSGeo4W64\QGIS\ms-windows\osgeo4w\build-qgis-test-x86_64\qgis.sln`.
* Try to build the solution (go grab a cup of tea, it may take a while).
* If it fails, run it again and again until there are (hopefully) no errors.

Running QGIS from within MSVC:
* Edit the properties of the project ALL_BUILD to include the path to the executable:
* Debugging -> Command -> `C:\OSGeo4W64\QGIS\ms-windows\osgeo4w\build-qgis-test-x86_64\output\bin\RelWithDebInfo\qgis.exe`.
* To run, use the menu commands: Debug -> Start Debugging (F5) or Start Without Debugging (Ctrl+F5).
* Ignore the "These projects are out of date" message, it appears even if no files were changed.

### 4.1.5 Old alternative method that might still work using cmake-gui
Create a 'build' directory somewhere. This will be where all the build output
will be generated.

Now run `cmake-gui` (still from `cmd`) and in the *Where is the source code:*
box, browse to the top level QGIS directory.

In the *Where to build the binaries:* box, browse to the `build` directory you
created.

If the path to bison and flex contains blanks, you need to use the short name
for the directory (i.e. `C:\Program Files` should be rewritten to
`C:\Progra~n`, where `n` is the number as shown in `dir /x C:\`).

Verify that the `BINDINGS_GLOBAL_INSTALL` option is not checked, so that python
bindings are placed into the output directory when you run the `INSTALL` target.

Hit `Configure` to start the configuration and select `Visual Studio 9 2008`
and keep `native compilers` and click `Finish`.

The configuration should complete without any further questions and allow you to
click `Generate`.

Now close `cmake-gui` and continue on the command prompt by starting
`vcexpress`.  Use File / Open / Project/Solutions and open the
qgis-x.y.z.sln File in your project directory.

Change `Solution Configuration` from `Debug` to `RelWithDebInfo` (Release
with Debug Info)  or `Release` before you build QGIS using the `ALL_BUILD`
target (otherwise you need debug libraries that are not included).

After the build completed you should install QGIS using the `INSTALL` target.

Install QGIS by building the `INSTALL` project. By default this will install to
`C:\Program Files\qgis<version>` (this can be changed by changing the
`CMAKE_INSTALL_PREFIX` variable in `cmake-gui`).

You will also either need to add all the dependency DLLs to the QGIS install
directory or add their respective directories to your `PATH`.

### 4.1.6. Packaging

To create a standalone installer there is a perl script named `creatensis.pl`
in `qgis/ms-windows/osgeo4w`.  It downloads all required packages from OSGeo4W
and repackages them into an installer using NSIS.

The script can be run on both Windows and Linux.

On Debian/Ubuntu you can just install the `nsis` package.

NSIS for Windows can be downloaded at:

https://nsis.sourceforge.io/Main_Page

And Perl for Windows (including other requirements like `wget`, `unzip`, `tar`
and `bzip2`) is available at:

https://cygwin.com

### 4.1.7. Packaging your own build of QGIS

Assuming you have completed the above packaging step, if you want to include
your own hand built QGIS executables, you need to copy them in from your
windows installation into the ms-windows file tree created by the creatensis
script.

```cmd
cd ms-windows/
rm -rf osgeo4w/unpacked/apps/qgis/*
cp -r /tmp/qgis1.7.0/* osgeo4w/unpacked/apps/qgis/
```

Now create a package.

```cmd
./quickpackage.sh
```

After this you should now have a nsis installer containing your own build
of QGIS and all dependencies needed to run it on a windows machine.

### 4.1.8. Osgeo4w packaging

The actual packaging process is currently not documented, for now please take a
look at `ms-windows/osgeo4w/package.cmd`.