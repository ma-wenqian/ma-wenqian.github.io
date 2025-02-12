---
layout: post
title: Compile OpenSeesPy on Windows using cmake
date: 2025-02-12 17:14 +0800
tags: [OpenSeesPy]
description: Visual Studio 2019, Python=3.x, conan<2.0
---

## Clone source code

```bash
git clone https://github.com/OpenSees/OpenSees.git
cd OpenSees
```

## 1. Setup environment

 1.1 install CMake
 1.2 install visual studio 2019 (default dictionary, C:\Program Files (x86)\Microsoft Visual Studio\2019)
 1.3 install IntelOne Basic & HPC Toolkits (default dictionary,C:\Program Files (x86)\Intel\oneAPI)

 1.4 install python 3.11 or create conda env

### 1.5 install conan

```bash
pip install conan<2.0
```


### 1.6 build MUMPS

```bash
git clone https://github.com/OpenSees/mumps.git
cd mumps
mkdir build
cd build
call "C:\Program Files (x86)\Intel\oneAPI\setvars.bat" intel64 mod
cmake .. -Darith=d -DCMAKE_MSVC_RUNTIME_LIBRARY="MultiThreaded" -G Ninja
cmake --build . --config Release --parallel 4
```


## 2. Building the OpenSees Applications and Python module

### Prepare cmake

```bash
cd ..\..
mkdir build
cd build
conan install .. --build missing --settings compiler="Visual Studio" --settings compiler.runtime="MT" --settings compiler.version=16

call "C:\Program Files (x86)\Intel\oneAPI\setvars.bat" intel64 mod
cmake .. -DBLA_STATIC=ON -DMKL_LINK=static -DMKL_INTERFACE_FULL=intel_lp64 -DMUMPS_DIR="..\..\mumps\build"
```

### 2.1 opensees

```bash
cmake --build . --config Release --target OpenSees -j8
```

### 2.2 openseespy

```bash
cmake --build . --config Release --target OpenSeesPy -j8

### powershell|
mv ./bin/OpenSeesPy.dll ./bin/opensees.pyd
cp "C:\Program Files (x86)\Intel\oneAPI\compiler\2024.1\bin\libiomp5md.dll" ./bin/
```

> Reference: 
> https://github.com/OpenSees/OpenSees/blob/master/.github/workflows/build_cmake.yml









## 3. Compile custom uniaxial material

- Add / create / edit the source files
  - Material -> `OpenSees/SRC/material/<uniaxial or nD>/`
  
    example: new uniaxial material named mym

    copy `mym.cpp` and `mym.h` to `SRC/material/uniaxial/`

  - Add the class tag to `mym.h`
  
    `#define MAT_TAG_SDBolt 20240518`

- Update the CMakeLists.txt

  The editing is simple for materials, e.g., for a uniaxial material add a line in `OpenSees/SRC/material/uniaxial/CMakeLists.txt`: `mym.cpp`,`mym.h`

- Add the updates to the interpreters

  Add to `OpenSees/SRC/interpreter/OpenSeesUniaxialMaterialCommands.cpp` for the Python interpreter. Near the top of the file:

    ```void* OPS_SLModel();```

    and further down in the file:

    ``uniaxialMaterialsMap.insert(std::make_pair("SLModel", &OPS_SLModel));``

-  Compile OpenSees
    
    ```bash
    call "C:\Program Files (x86)\Intel\oneAPI\setvars.bat" intel64 mod
    cmake .. -DBLA_STATIC=ON -DMKL_LINK=static -DMKL_INTERFACE_FULL=intel_lp64 -DMUMPS_DIR="..\..\mumps\build"
    cmake --build . --config Release --target OpenSeesPy -j8
    ```