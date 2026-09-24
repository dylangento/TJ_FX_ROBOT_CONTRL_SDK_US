# ATTENTION

1. Please first become familiar with MARVIN_APP or MarvinPlatform. Using the app will help you understand how to operate the Marvin robot and make subsequent code development easier.
2. `DEMO_C++/` and `DEMO_PYTHON/` contain interface usage demos. The top of each demo explains the example and its usage logic. Be sure to read this first, then modify the demo for your situation before running it.
   The logic and parameters in these demos were developed for research and development testing. They are for reference only and are not production code.
   For example:
   a. For safety, both the speed percentage and acceleration percentage are set to 10%. After extensive testing, you can adjust them to full speed, 100%.
   b. The demos sleep for 1 second or 500 milliseconds between parameter settings. In practice, a 1-millisecond sleep between parameter settings is sufficient.
   c. After setting target joint positions, the tests sleep for a few seconds to wait for the arm to reach its target. In production, you can repeatedly subscribe to the arm's current position to determine whether it has reached the specified point, or subscribe to the low-speed flag to determine this.
   d. The stiffness and damping coefficients are also reference values. These values may increase with different controller versions; consult technical personnel for details.

## I. SDK documentation

Please read the SDK documentation for a comprehensive understanding of robot operation, interface functions, updates, and precautions.

[SDK home](../README_EN.md)

[C++ control SDK documentation](../c++_doc_contrl_EN.md)
[Python control SDK documentation](../python_doc_contrl_EN.md)

[C++ kinematics SDK documentation](../c++_doc_kine_EN.md)
[Python kinematics SDK documentation](../python_doc_kine_EN.md)

## II. Files in the SDK library folder

The files in `SDK_PYTHON` are:

```text
TJ_FX_ROBOT_CONTRL_SDK-master
|————DEMO_PYTHON  # SDK usage examples in Python
|————SDK_PYTHON
        |————fx_kine.py # Kinematics interfaces
        |————fx_robot.py # Control interfaces
        |————libKine.dll # Kinematics shared library for Windows
        |————libKine.so # Kinematics shared library for Linux
        |————libMarvinSDK.dll # Control shared library for Windows
        |————libMarvinSDK.so # Control shared library for Linux
```

Note: Check whether the shared libraries in `SDK_PYTHON` are the latest builds.

## III. SDK libraries

`SDK_PYTHON` is the Python SDK for Tianji dual-arm robots and humanoid robots. It consists of:

- Control SDK: `SDK_PYTHON/fx_robot.py`
- Kinematics SDK: `SDK_PYTHON/fx_kine.py`

### 3.1 Using automated build scripts

Running `marvinSDK_windows.bat` on the master branch automatically compiles the DLL files used by C++ and Python.
Running `marvinSDK_ubuntu.sh` on the master branch automatically compiles the SO files used by C++ and Python.

### 3.2.1 Compiling SO shared libraries

Compile on a Linux device:

Control SDK (`contrlSDK`), using either method:

1. `g++ *.cpp  -Wall -w -O2 -fPIC -shared -o libMarvinSDK.so -lpthread -lrt -DCMPL_LIN`
2. Use `./contrlSDK/makefile` to generate `libMarvinSDK.so`.

Kinematics SDK (`kinematicsSDK`), using either method:

1. `g++ *.cpp  -Wall -w -O2 -fPIC -shared -o libKine.so -lpthread -lrt`
2. Use `./kinematicsSDK/makefile` to generate `libKine.so`.

The compiled `libKine.so` and `libMarvinSDK.so` are for use by C++ and Python on the machine where they were compiled.

### 3.2.2 Compiling DLL libraries for C++

1) Use MinGW on Windows to compile the DLL libraries:

Control SDK (`contrlSDK`):

```text
g++ *.cpp -Wall -w -O2 -shared -o libMarvinSDK.dll -lws2_32 -lwinmm -DCMPL_WIN
```

Kinematics SDK (`kinematicsSDK`):

```text
g++ *.cpp -Wall -w -O2 -fPIC -shared -o libKine.dll
```

The compiled `libKine.dll` and `libMarvinSDK.dll` are for use by C++ on Windows.

### 3.2.3 Compiling DLL libraries for Python

1) Compile DLL libraries on Linux:

Control SDK (`contrlSDK`):

```text
x86_64-w64-mingw32-g++ *.cpp -Wall -O2 -shared -o libMarvinSDK.dll -DBUILDING_DLL -DCMPL_WIN -static -static-libgcc -static-libstdc++ -lws2_32 -lpthread -lwinmm
```

Kinematics SDK (`kinematicsSDK`):

```text
g++ *.cpp -Wall -w -O2 -fPIC -shared -o libKine.dll
```

2) Use MinGW on Windows to compile the DLL libraries:

Control SDK (`contrlSDK`):

```text
g++ *.cpp -Wall -w -O2 -shared -o libMarvinSDK.dll -DBUILDING_DLL -D_WIN32 -DCMPL_WIN -fPIC -static -static-libgcc -static-libstdc++ -lws2_32 -lwinmm
```

Kinematics SDK (`kinematicsSDK`):

```text
g++ *.cpp -Wall -w -O2 -shared -o libKine.dll -DBUILDING_DLL -D_WIN32 -fPIC -static -static-libgcc -static-libstdc++ -lws2_32 -lwinmm
```

The compiled `libKine.dll` and `libMarvinSDK.dll` are for use by Python on Windows.

## IV. Control showcases

### 0. Check SDK type compatibility

    showcase_check_sdk_type_compat.py

### 1. Dual-arm joint position following control demonstration

    showcase_position.py

### 2. Execute a PVT trajectory with a single arm and save data

    showcase_pvt_arm_A.py

### 3. Single-arm joint impedance control in torque mode

    showcase_torque_joint_impedance_arm_A.py

### 4. Single-arm Cartesian impedance control in torque mode

    showcase_torque_cart_impedance_arm_A.py

### 5. Single-arm force control in torque mode

    showcase_torque_force_impedance_arm_A.py

### 6. Drag a single arm in joint impedance mode

    showcase_joint_drag_arm_A.py

### 7. Drag a single arm in joint impedance mode and save data

    showcase_drag_JointImpedance_and_save_data_arm_A.py

### 8. Drag a single arm in Cartesian impedance mode

    showcase_cart_drag_arm_A.py

### 9. Drag a single arm in Cartesian impedance mode and save data

    showcase_drag_CartImpedance_and_save_data_arm_A.py

### 10. Save data

    showcase_collect_data.py

### 11. Save data as CSV

    showcase_collect_data_as_csv.py

### 12. Save tool dynamics and kinematics information

    showcase_set_save_tool.py

### 13. Get and set robot configuration parameters

    showcase_get_set_param.py

### 14. Single-arm end-effector 485 communication

    showcase_485_arm_A.py

### 15. Single-arm end-effector CAN/CANFD communication

    showcase_CAN_arm_A.py

### 16. Clear motor errors and zero the motor encoder

    showcase_motor_encoder_clear.py

### 17. Collaborative release

    showcase_collaborative_release.py

### 18. Brake release and engagement

    showcase_apply-brake_release-brake.py

### 19. Get servo error codes and their corresponding causes

    showcase_servo_error.py

### 20. Convert a trajectory recorded during dragging into a PVT file

    showcase_process_collect_data_to_pvt_format.py

### 21. Tool Cartesian impedance

    showcase_torque_EefCart_impedance_arm_A.py

### 22. Soft reset the servo of a specified joint

    showcase_servo_reset.py

### 23. Check command transmission latency

    showcase_check_cmd_delay.py

### 24. Send commands using joint-space planning to eliminate jitter

    showcase_pln_joint_positionMode.py

### 25. Send commands using Cartesian-space planning to enforce a straight-line constraint

    showcase_pln_cart_positionMode.py

### 26. Send commands using joint-space planning and interrupt execution

    showcase_pln_joint_positionMode_with_break.py

### 27. Send commands using Cartesian-space planning and interrupt execution

    showcase_pln_cart_positionMode_with_break.py

### 28. Convert joint torques to six-dimensional end-effector force

    showcase_jointsTorque2EefTorque.py

# Recommended execution order: 29->30->31->32

### 29. Synchronized dual-arm cooperative motion with joint-space planning (setPln_joint_AB)

    showcase_pln_joint_to_joint_two_arms.py

### 30. Synchronized dual-arm cooperative motion with joint-space straight-line planning (movL_KeepJA + setPln_Cart_AB)

    showcase_pln_joint_to_joints_linear_two_arms.py

### 31. Synchronized dual-arm cooperative motion with Cartesian-space straight-line planning (movLA + setPln_Cart_AB)

    showcase_pln_cartesian_linear_two_arms.py

### 32. Synchronized dual-arm cooperative motion with multipoint straight-line planning (multi_movL + setPln_Cart_AB)

    showcase_pln_multi_segment_linear_two_arms.py

## V. Kinematics showcases

### 1. Complete demonstration of the kinematics SDK functional modules

    showcase_kinematics_all_functions.py

### 2. Summary of inverse-kinematics calculation failures

    showcase_ik_failed_conclusion.py

### 3. Calculate for two arms simultaneously

    showcase_kine_two_arms.py

### 4. CCS right-arm tool dynamics identification demonstration script

    showcase_identy_tool_dynamic_CCS_B.py

### 5. SRS right-arm tool dynamics identification demonstration script

    showcase_identy_tool_dynamic_SRS_B.py

### 6. Inverse-kinematics reference baseline

    showcase_ik_nsp_two_arms.py

### 7. Online straight-line planning and point execution in Cartesian impedance mode at 50 Hz

    showcase_online_pln_movl.py

### 8. Online straight-line planning with a constrained configuration and point execution in Cartesian impedance mode at 50 Hz

    showcase_online_pln_movl_keepj.py

### 9. Online straight-line planning with a constrained configuration and point execution in Cartesian impedance mode at 50 Hz, with specified rotation

    showcase_online_pln_movl_with_specific_rot.py

### 10. Online multipoint planning, executed by the controller at 50 Hz

    showcase_pln_cart_multi-segment_positionMode.py
