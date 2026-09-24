# Control SDK

## Attention

1. Become familiar with MARVIN_APP or MarvinPlatform before developing with the SDK. The application explains the robot's operating logic.
2. The `DEMO_C++/` and `DEMO_PYTHON/` directories contain interface examples. Read the description at the top of each demo and adapt it to your site before running it. These examples were created for development testing and are not production code.
3. For safety, the examples use 10% velocity and acceleration. Increase them only after sufficient testing.
4. Example delays are intentionally long. Production code can use shorter delays when the controller state is monitored correctly.
5. Stiffness and damping values are reference values. Valid values may differ between controller versions; contact technical support when necessary.

## 1. SDK documentation

Read the SDK documentation for the robot control logic, API details, updates, and cautions.

[SDK home](../README_EN.md)

[C++ control SDK documentation](../c++_doc_contrl_EN.md)

[Python control SDK documentation](../python_doc_contrl_EN.md)

[C++ kinematics SDK documentation](../c++_doc_kine_EN.md)

[Python kinematics SDK documentation](../python_doc_kine_EN.md)

## 2. Building SDK libraries

The control SDK header is `MarvinSDK.h`.

### 2.1 Automated build scripts

Run `marvinSDK_windows.bat` on Windows to build DLLs for C++ and Python.

Run `marvinSDK_ubuntu.sh` on Ubuntu to build shared libraries for C++ and Python.

### 2.2 Build shared libraries on Linux

Control SDK (`contrlSDK`):

```text
g++ *.cpp -Wall -O2 -fPIC -shared -o libMarvinSDK.so -lpthread -lrt -DCMPL_LIN
```

Kinematics SDK (`kinematicsSDK`):

```text
g++ *.cpp -Wall -O2 -fPIC -shared -o libKine.so -lpthread -lrt
```

The resulting `libMarvinSDK.so` and `libKine.so` can be used by C++ and Python on the build machine.

### 2.3 Build DLLs for C++ on Windows

Use MinGW:

```text
g++ *.cpp -Wall -O2 -shared -o libMarvinSDK.dll -lws2_32 -lwinmm -DCMPL_WIN
g++ *.cpp -Wall -O2 -fPIC -shared -o libKine.dll
```

The resulting DLLs are used by C++ applications on Windows.

### 2.4 Build DLLs for Python

For the control SDK, build with the appropriate MinGW toolchain and these libraries:

```text
x86_64-w64-mingw32-g++ *.cpp -Wall -O2 -shared -o libMarvinSDK.dll -DBUILDING_DLL -DCMPL_WIN -static -static-libgcc -static-libstdc++ -lws2_32 -lpthread -lwinmm
g++ *.cpp -Wall -O2 -shared -o libMarvinSDK.dll -DBUILDING_DLL -D_WIN32 -DCMPL_WIN -fPIC -static -static-libgcc -static-libstdc++ -lws2_32 -lwinmm
```

For the kinematics SDK:

```text
g++ *.cpp -Wall -O2 -fPIC -shared -o libKine.dll
```

The resulting DLLs can be loaded by Python on Windows.

### 2.5 Compile from source without a dynamic library

Assume `main.cpp` is in a `workspace` directory next to `contrlSDK`:

```text
...
|---contrlSDK
|---workspace
    |---main.cpp
...
```

Windows:

```text
g++ -Wall main.cpp ../contrlSDK/*.cpp -I../contrlSDK -o main.exe -lws2_32 -lwinmm -DCMPL_WIN
```

Linux:

```text
g++ -Wall main.cpp ../contrlSDK/*.cpp -I../contrlSDK -o main -lpthread -lrt -DCMPL_LIN
```

The output is `main.exe` on Windows or `main` on Linux.
