# ATTENTION

1. Please first become familiar with MARVIN_APP or MarvinPlatform. Using the app will help you understand how to operate the Marvin robot and make subsequent code development easier.
2. `DEMO_C++/` and `DEMO_PYTHON/` contain interface usage demos. The top of each demo explains the example and its usage logic. Be sure to read this first, then modify the demo for your situation before running it.
   The logic and parameters in these demos were developed for research and development testing. They are for reference only and are not production code.
   For example:
   a. For safety, both the speed percentage and acceleration percentage are set to 10%. After extensive testing, you can adjust them to full speed, 100%.
   b. The demos sleep for 1 second or 500 milliseconds between parameter settings. In practice, a 1-millisecond sleep between parameter settings is sufficient.
   c. After setting target joint positions, the tests sleep for a few seconds to wait for the arm to reach its target. In production, you can repeatedly subscribe to the arm's current position to determine whether it has reached the specified point, or subscribe to the low-speed flag to determine this.
   d. The stiffness and damping coefficients are also reference values. These values may increase with different controller versions; consult technical personnel for details.

# I. SDK documentation

Please read the SDK documentation for a comprehensive understanding of robot operation, interface functions, updates, and precautions.

[SDK home](../README_EN.md)

[C++ control SDK documentation](../c++_doc_contrl_EN.md)

[Python control SDK documentation](../python_doc_contrl_EN.md)

[C++ kinematics SDK documentation](../c++_doc_kine_EN.md)

[Python kinematics SDK documentation](../python_doc_kine_EN.md)

# II. Compiling the kinematics SDK libraries

The kinematics SDK header is `FxRobot.h`.

## 2.1.1 Compiling SO shared libraries

Compile on a Linux device:

Control SDK (`contrlSDK100343`), using either method:

1. `g++ *.cpp -Wall -O2 -fPIC -shared -o libMarvinSDK.so -lpthread -lrt -DCMPL_LIN`
2. Use `./contrlSDK100343/makefile` to generate `libMarvinSDK.so`.

Kinematics SDK (`kinematicsSDK`), using either method:

1. `g++ *.cpp -Wall -O2 -fPIC -shared -o libKine.so -lpthread -lrt`
2. Use `./kinematicsSDK/makefile` to generate `libKine.so`.

The compiled `libKine.so` and `libMarvinSDK.so` are for use by C++ and Python on the machine where they were compiled.

## 2.1.2 Compiling DLL libraries for C++

1) Use MinGW on Windows to compile the DLL libraries:

Control SDK (`contrlSDK100343`):

```text
g++ *.cpp -Wall -O2 -shared -o libMarvinSDK.dll -lws2_32 -lwinmm -DCMPL_WIN
```

Kinematics SDK (`kinematicsSDK`):

```text
g++ *.cpp -Wall -O2 -fPIC -shared -o libKine.dll
```

The compiled `libKine.dll` and `libMarvinSDK.dll` are for use by C++ on Windows.

## 2.1.3 Compiling DLL libraries for Python

1) Compile DLL libraries on Linux:

Control SDK (`contrlSDK100343`):

```text
x86_64-w64-mingw32-g++ *.cpp -Wall -O2 -shared -o libMarvinSDK.dll -DBUILDING_DLL -DCMPL_WIN -static -static-libgcc -static-libstdc++ -lws2_32 -lpthread -lwinmm
```

Kinematics SDK (`kinematicsSDK`):

```text
g++ *.cpp -Wall -O2 -fPIC -shared -o libKine.dll
```

2) Use MinGW on Windows to compile the DLL libraries:

Control SDK (`contrlSDK100343`):

```text
g++ *.cpp -Wall -O2 -shared -o libMarvinSDK.dll -DBUILDING_DLL -D_WIN32 -DCMPL_WIN -fPIC -static -static-libgcc -static-libstdc++ -lws2_32 -lwinmm
```

Kinematics SDK (`kinematicsSDK`):

```text
g++ *.cpp -Wall -O2 -shared -o libKine.dll -DBUILDING_DLL -D_WIN32 -fPIC -static -static-libgcc -static-libstdc++ -lws2_32 -lwinmm
```

The compiled `libKine.dll` and `libMarvinSDK.dll` are for use by Python on Windows.


