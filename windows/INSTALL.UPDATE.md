# Building Kaldi for Windows 10 x64

## Requirements
- [Microsoft Visual Studio 2017](https://visualstudio.microsoft.com/downloads/)
- [Git Bash](https://git-scm.com/downloads)
- BLAS library ([OpenBLAS](https://sourceforge.net/projects/openblas/files/v0.2.14/) or [Intel MKL](https://software.intel.com/en-us/mkl/choose-download/windows))
- [OpenFST](https://github.com/kkm000/openfst/)
- [Kaldi](https://github.com/kaldi-asr/kaldi)

### Microsoft Visual Studio 2017
Install Microsoft Visual Studio 2017 and add the following to the installer:
- Workload: Desktop Development with C++
- Windows Universal CRT SDK

### Git Bash
Install [Git Bash](https://git-scm.com/downloads).
Stick to the default choices of the installer if you are in doubt of any options.

### BLAS library
- OpenBLAS is a good choice for compiling Kaldi as it is free, open source and has good performance
- Intel MKL has great performance and was recently made available for free

Depending on your preference you will most likely not experience huge differences between the two.

Other BLAS libraries are available such as [ATLAS](https://sourceforge.net/projects/math-atlas/) but is not covered in this guide.

For OpenBLAS, download version 0.2.14 (Win64-int32 and mingw64): https://sourceforge.net/projects/openblas/files/v0.2.14/

For Intel MKL you must register with Intel before downloading (I used **2019 Update 3 for Windows**): https://software.intel.com/en-us/mkl/choose-download/windows

## Compiling OpenFST
Acquire OpenFST winport branch using Git bash:

    git clone -b winport https://github.com/kkm000/openfst.git

Open `openfst.sln` with Visual Studio 2017.

Once the solution is open, start by reading the comments in the file under the **"READ ME BEFORE BUILD"** solution folder. It is here you must select the toolset and SDK for compiling. 

Set property to **true** for `v141`, to use the toolset that comes with Visual Studio 2017.

Set property to **true** for the version of Windows 10 SDK you have installed. If your version of SDK is missing on the list, add it yourself by copying one of the lines and adding the version number for the SDK you have installed.

Example:

``` js
<PropertyGroup>
  <!-- Windows 10 Update 1809 (October 2018) -->
  <WindowsTargetPlatformVersion Condition="true" >10.0.17763.0</WindowsTargetPlatformVersion>
</PropertyGroup>
```

Enabling [AVX](https://en.wikipedia.org/wiki/Advanced_Vector_Extensions) will increase performance.

Build the solution for both **Debug|x64** and **Release|x64**.
The output is placed in `*\openfst\build_output\x64`

Copy the file `libfst.lib` from `*\openfst\build_output\x64\Debug\lib` and place it into `*\openfst\src\lib\Debug` and **rename** it `fst.lib`

Copy the file `libfst.lib` from `*\openfst\build_output\x64\Release\lib` and place it into `*\openfst\src\lib\Release` and **rename** it `fst.lib`

## Compiling Kaldi
1. Acquire Kaldi using Git bash:

        git clone https://github.com/kaldi-asr/kaldi

3. If using OpenBLAS, extract `OpenBLAS-v0.2.14-Win64-int32.zip` and `mingw64_dll.zip` to `(kaldi)/tools`

4. Enter the `(kaldi)/windows` folder

5. Make a copy of `variables.props.dev` and rename the copy `variables.props`.

6. Open `variables.props` in a text editor.

   If you are using Intel MKL, you can ignore `OPENBLASDIR`.
If you are using OpenBlas, you can ignore `MKLDIR`.

   `OPENFST` should point to the location of OpenFST.

   `OPENFSTLIB` should point to the location of OpenFST as well (do not link further into the available folders)

   Ignore `CUBDIR` and `NVTOOLSDIR` as we will not use them in this guide

   Example:

        <MKLDIR>C:\Program Files (x86)\IntelSWTools\compilers_and_libraries\windows\mkl\</MKLDIR>
        <OPENBLASDIR>C:\dev\kaldi\tools\OpenBLAS-v0.2.14-Win64-int32</OPENBLASDIR>
        <OPENFST>C:\dev\openfst</OPENFST>
        <OPENFSTLIB>C:\dev\openfst</OPENFSTLIB>

6. For MKL support, make a copy of the file `kaldiwin_mkl.props` and rename the copy `kaldiwin.props`

7. For OpenBLAS support, make a copy of the file `kaldiwin_openblas.props` and rename the copy `kaldiwin.props`

8. Create the Microsoft Visual Studio solution using Git Bash
Using Git Bash, navigate to the folder containing kaldi/windows

   Example:

       cd C:/dev/kaldi/windows
   Call the script that generates the solution:

    For Intel MKL: 

        perl generate_solution.pl --vsver vs2017 --enable-mkl

    For OpenBLAS: 

        perl generate_solution.pl --vsver vs2017 --enable-openblas

9. The generated solution will be placed in a parent folder looking something like this: `C:/dev/kaldi/kaldiwin_vs2017_*`
Open the generated solution in Visual Studio and switch to **Debug|x64** (or **Release|x64**) and build. (You may need to retarget solution. Do so if necessary)
Expect 11 projects or so to fail due to missing `portaudio.h`. The tests will most likely fail as well.