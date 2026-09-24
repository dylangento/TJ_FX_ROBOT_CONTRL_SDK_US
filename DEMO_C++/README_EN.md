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

# II. Required files in DEMO_C++/

Check whether the following files were compiled for this machine:

```text
libKine.dll
libMarvinSDK.dll

libKine.so
libMarvinSDK.so
```

Check whether the following header files are present:

```text
FXDG.h
FxRobot.h
MarvinSDK.h
PointSet.h
```

## 2.1 How to compile the control and kinematics libraries

## 2.1.1 Using automated build scripts

Running `marvinSDK_windows.bat` on the master branch automatically compiles the DLL files used by C++ and Python.
Running `marvinSDK_ubuntu.sh` on the master branch automatically compiles the SO files used by C++ and Python.

### 2.1.2 Compiling SO shared libraries

Compile on a Linux device:

Control SDK (`contrlSDK`), using either method:

1. `g++ *.cpp  -Wall -O2 -fPIC -shared -o libMarvinSDK.so -lpthread -lrt -DCMPL_LIN`
2. Use `./contrlSDK/makefile` to generate `libMarvinSDK.so`.

Kinematics SDK (`kinematicsSDK`), using either method:

1. `g++ *.cpp  -Wall -O2 -fPIC -shared -o libKine.so -lpthread -lrt`
2. Use `./kinematicsSDK/makefile` to generate `libKine.so`.

The compiled `libKine.so` and `libMarvinSDK.so` are for use by C++ and Python on the machine where they were compiled.

### 2.1.3 Compiling DLL libraries for C++

1) Use MinGW on Windows to compile the DLL libraries:

Control SDK (`contrlSDK`):

```text
g++ *.cpp -Wall -O2 -shared -o libMarvinSDK.dll -lws2_32 -lwinmm -DCMPL_WIN
```

Kinematics SDK (`kinematicsSDK`):

```text
g++ *.cpp -Wall -O2 -fPIC -shared -o libKine.dll
```

The compiled `libKine.dll` and `libMarvinSDK.dll` are for use by C++ on Windows.

### 2.1.4 Compiling DLL libraries for Python

1) Compile DLL libraries on Linux:

Control SDK (`contrlSDK`):

```text
x86_64-w64-mingw32-g++ *.cpp -Wall -O2 -shared -o libMarvinSDK.dll -DBUILDING_DLL -DCMPL_WIN -static -static-libgcc -static-libstdc++ -lws2_32 -lpthread -lwinmm
```

Kinematics SDK (`kinematicsSDK`):

```text
g++ *.cpp -Wall -O2 -fPIC -shared -o libKine.dll
```

2) Use MinGW on Windows to compile the DLL libraries:

Control SDK (`contrlSDK`):

```text
g++ *.cpp -Wall -O2 -shared -o libMarvinSDK.dll -DBUILDING_DLL -D_WIN32 -DCMPL_WIN -fPIC -static -static-libgcc -static-libstdc++ -lws2_32 -lwinmm
```

Kinematics SDK (`kinematicsSDK`):

```text
g++ *.cpp -Wall -O2 -shared -o libKine.dll -DBUILDING_DLL -D_WIN32 -fPIC -static -static-libgcc -static-libstdc++ -lws2_32 -lwinmm
```

The compiled `libKine.dll` and `libMarvinSDK.dll` are for use by Python on Windows.

### 2.1.5 Using source code without shared libraries

Using `contrlSDK` as an example, suppose the calling code file `main.cpp` is in a `workspace` folder at the same directory level as `contrlSDK`. The directory tree is:

```text
...
|---contrlSDK
|---workspace
    |--- main.cpp
...
```

The compilation commands are:

1) On Windows:

```text
g++ -Wall main.cpp ../contrlSDK/*.cpp -I../contrlSDK -o main.exe -lws2_32 -lwinmm -DCMPL_WIN
```

2) On Linux:

```text
g++ -Wall main.cpp ../contrlSDK/*.cpp -I../contrlSDK -o main -lpthread -lrt -DCMPL_LIN
```

After compilation, the following is generated:

```text
...
|---contrlSDK
|---workspace
    |--- main.cpp
    |--- main.exe or main
...
```

# III. Compiling and running examples

## 3.1 Linux examples

Using only the kinematics library:

```text
g++ showcase_kinematics_all_functions.cpp -o kine -L. -lKine -Wl,-rpath=.
./kine
```

Using both the control and kinematics libraries:

```text
g++ showcase_offline_movl_keepj_execution.cpp -o offline_movl_keepj -L. -lKine -lMarvinSDK -Wl,-rpath=.
./offline_movl_keepj
```

## 3.2 Windows examples

Using only the kinematics library:

```text
g++ showcase_kinematics_all_functions.cpp -o kine.exe -L. -lKine
kine.exe
```

Using both the control and kinematics libraries:

```text
g++ showcase_offline_movl_keepj_execution.cpp -o offline_movl_keepj.exe -L. -lKine -lMarvinSDK
offline_movl_keepj.exe
```

# IV. Control showcases

## 0. Check SDK type compatibility

    showcase_check_sdk_type_compat.cpp

## 1. Force brake engagement and release

    showcase_apply_brake_release_brake.cpp

## 2. Put the robot into collaborative release

    showcase_collaborative_release.cpp

## 3. Enter dragging along the Cartesian Y direction in Cartesian impedance mode, drag, and save data

    showcase_drag_CartImpedance_save_data.cpp

## 4. Dragging control

    showcase_drag_joint.cpp

## 5. Enter joint dragging in joint impedance mode, drag, and save data

    showcase_drag_JointImpedance_save_data.cpp

## 6. Get and set parameters

    showcase_get_set_param_demo.cpp

## 7. Connection check

    showcase_link_check.cpp

## 8. Joint position following control

    showcase_position_two_arms.cpp

## 9. Run a PVT trajectory and save data

    showcase_pvt.cpp

## 10. Cartesian impedance control

    torque_cart_impedance_demo.cpp

## 11. Force control

    torque_force_impedance_demo.cpp

## 12. Joint impedance control

    torque_joint_impedance_demo.cpp

## 13. Enter joint dragging in joint impedance mode, drag, and save data

    showcase_drag_JointImpedance_save_data.cpp

## 14. Check command transmission latency

    showcase_cmd_delay.cpp

## 15. Send commands using joint-space planning to eliminate jitter

    showcase_pln_jointSpace_PositionMode.cpp

## 16. Send commands using Cartesian-space planning to enforce a straight-line constraint

    showcase_pln_cartSpace_PositionMode.cpp

## 17. Send commands using joint-space planning and interrupt the planned motion

    showcase_pln_jointSpace_PositionMode_with_break.cpp

## 18. Send commands using Cartesian-space planning and interrupt the planned motion

    showcase_pln_cartSpace_PositionMode_with_break.cpp

## 19. Tool Cartesian impedance control

    showcase_torque_EefCart_impedance.cpp

## 20. Move the arm's end effector through specified positional and orientation distances with a given force and torque; adjustments to the force direction and magnitude can be triggered in real time

    showcase_Force_field_Control.cpp

# VI. Simplified control SDK interface example

    showcase_new_control_sdk_usage.cpp

# V. Kinematics showcases

## 1. Complete demonstration of the kinematics SDK functional modules

    showcase_kinematics_all_functions.cpp

## 2. Summary of inverse-kinematics calculation failures

    showcase_ik_failed_conclusion.cpp

## 3. Calculate for two arms simultaneously

    showcase_kine_two_arms.cpp

## 5. Inverse-kinematics reference baseline

    showcase_ik_nsp_two_arms.cpp

## 6. Demonstrate the left arm's offline and online planning interfaces

    showcase_online_and_offline_pln_all_function.cpp

## 7. Execute an offline straight-line planning file for the left arm in joint impedance mode at 50 Hz

    showcase_offline_movl_execution.cpp

## 8. Execute online straight-line planning points for the left arm in joint impedance mode at 50 Hz

    showcase_online_movla_execution.cpp

## 9. Execute an offline straight-line planning file with a constrained configuration for the left arm in joint impedance mode at 50 Hz

    showcase_offline_movl_keepj_execution.cpp

## 10. Execute online straight-line planning points with a constrained configuration for the left arm in joint impedance mode at 50 Hz

    showcase_online_movl_keepja_execution.cpp
