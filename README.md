**🌐 Language / 语言:** [English](README_EN.md) | [中文 (Chinese)](README_CN.md)

# Tianji MARVIN Robot Control SDK

Open-source control and kinematics SDK for Tianji MARVIN-series robots, covering C++/Python usage on Windows/Linux.

## Contents

- [I. SDK Overview](#i-sdk-overview)
- [II. Compilation Methods](#ii-compilation-methods)
- [III. SDK Updates](#iii-sdk-updates)
- [IV. Controller Version Updates](#iv-controller-version-updates)
- [V. App Updates](#v-app-updates)
- [VI. Precautions](#vi-precautions)
- [VII. Main Problems and Solutions](#vii-main-problems-and-solutions)

## Attention
    1. First become familiar with MarvinPlatform. Using the app will help you understand how to operate the Marvin robot and make subsequent code development easier.
    2. DEMO_C++/ and DEMO_PYTHON/ contain interface usage demos. The top of each demo explains the example and its usage logic. Be sure to read this first, then modify the demo for your situation before running it.
        The logic and parameters in these demos were developed for research and development testing. They are for reference only and are not production code.
            For example:
                a. For safety, both the speed percentage and acceleration percentage are set to 10%. After extensive testing, you can adjust them to full speed, 100%.
                b. The demos sleep for 1 second or 500 milliseconds between parameter settings. In practice, a 1-millisecond sleep between parameter settings is sufficient.
                c. After setting target joint positions, the tests sleep for a few seconds to wait for the arm to reach its target. In production, you can repeatedly subscribe to the arm's current position to determine whether it has reached the specified point, or subscribe to the low-speed flag to determine this.
                d. The stiffness and damping coefficients are also reference values. These values may increase with different controller versions; consult technical personnel for details.

## I. SDK overview

    MARVIN SDK description:
         1. The SDK for MARVIN-series robots is divided into the control SDK and the robot kinematics SDK.
         2. The control SDK supports C++/Python use and development on Windows/Linux.
         3. The kinematics SDK supports C++/Python on Windows/Linux (open-source kinematics SDK code: forward kinematics, inverse kinematics, inverse-kinematics null space, Jacobian matrices, straight-line planning movL, and tool-load dynamics identification. Contact us for commercial inquiries about dynamics calculation and floating-base interfaces).
         4. Our Linux development and testing use only x_86 machines. Please compile and test for other architectures.
         5. Host control applications for Ubuntu x_86/Windows are provided (open-source application code).

    Special notes:
            1. To operate our robots more smoothly, be sure to read the documentation and examples first.
            2. Use the host application before developing business and production scripts for your control requirements.

     The main robot control sequence is:
        Connect to the robot over UDP and confirm a valid connection by checking updates to received data
        |
        Set the parameters for the intended control state (speed, acceleration, stiffness, damping, etc.), then set the control state
        |
        Send joint commands/force commands
        |
        ...
        |
        When the task is complete, release the robot so that other programs or users can connect


    The following robot control states are currently available:
        1) Position mode/joint following mode (high stiffness and high precision; collisions are dangerous in this mode)
        2) PVT mode/offline trajectory replay mode (plan a 500 Hz trajectory in advance; speed and acceleration must also be planned)
        3) Torque mode/impedance mode, subdivided into joint impedance, Cartesian impedance, and force control
        4) Collaborative release mode, used to separate arms entangled after a collision or to change the robot configuration manually
        5) Disable/reset: resetting is required when switching between states for safety. Switching without a reset is possible while stationary (hybrid control).

    Operating parameters must be set first for both position and torque modes:
        1) In position mode, set the speed and acceleration percentages.
        2) In torque mode, set stiffness and damping in addition to the speed and acceleration percentages.
        3) For the special force-control mode, set the force-control travel range (millimeters).

    1 kHz data collection
        1) Data collection is independent of the robot control state and is available in any mode.
        2) A collection can contain 35 columns, i.e. 35 features, and up to 1 million rows. Start a new collection when it is full:
            Left-arm feature indices:
                        0-6     Left-arm joint positions
                        10-16   Left-arm joint velocities
                        20-26   Left-arm external encoder positions
                        30-36   Left-arm commanded joint positions
                        40-46   Left-arm joint currents (per mille)
                        50-56   Left-arm joint sensor torques, Nm
                        60-66   Left-arm friction estimates
                        70-76   Left-arm friction velocity estimates
                        80-86   Left-arm joint external-force estimates
                        90-95   Left-arm end-effector external-force estimates
            For the corresponding right-arm feature indices, add 100.

    
    In torque mode, the external button on the end effector can also be used for dragging:
        1) In joint impedance mode, select joint dragging for compliant joint dragging.
        2) In Cartesian impedance mode, select one Cartesian dragging direction: X, Y, Z, or rotation. Exit dragging before switching to another direction (otherwise the control behavior will be confused).


## Robot motion control modes
    MarvinPro robots (humanoid arms) support several motion control modes:
        - Position mode
        - Torque/impedance mode, subdivided into joint impedance, Cartesian impedance, and force-controlled impedance
        - PD mode
        - Collaborative release (zero-force dragging mode)
        - Trajectory replay mode


### Position Mode
    Using closed-loop servo control, each robot joint follows preset target position, velocity, and acceleration trajectories for precise point-to-point motion or continuous path tracking. This mode is suitable for tasks requiring high trajectory repeatability, such as handling, dispensing adhesive, and welding. It does not actively regulate external contact forces; position deviation is mainly determined by stiffness.

    Enabling conditions:
    C/C++ (connection omitted; only mode-switching code is shown):
```c
// Set speed and acceleration percentages
OnClearSet();
OnSetJointLmt_A(10, 10) ;
OnSetJointLmt_B(10, 10) ;
OnSetSend();
SLEEP(200);
// Switch to position mode
OnClearSet();
OnSetTargetState_A(1) ;
OnSetTargetState_B(1) ;
OnSetSend();
SLEEP(1000);
```
    Python (connection omitted; only mode-switching code is shown):
```python
'''Set speed and acceleration percentages'''
robot.clear_set()
robot.set_vel_acc(arm='A',velRatio=10, AccRatio=10)
robot.set_vel_acc(arm='B',velRatio=10, AccRatio=10)
robot.send_cmd()
time.sleep(0.2)
'''Switch to position mode'''
robot.clear_set()
robot.set_state(arm='A',state=1)
robot.set_state(arm='B',state=1)
robot.send_cmd()
time.sleep(1)
```


### Joint Impedance Mode
    Establishes a dynamic relationship between torque and position deviation in joint space, producing spring-damper behavior.

    The user must first set these parameters:
        Stiffness for each joint (range 0~22, unit N*m/deg). Higher stiffness makes the joint harder.
        Damping coefficient for each joint (range 0~1, recommended value 0.3).
    
    Higher damping reduces oscillation amplitude faster but slows the response to force and displacement, making motion feel more resistant and viscous. Lower damping reduces vibration suppression but makes motion smoother with less resistance, while leaving residual oscillation when stopping at a position.
    Joint impedance damping is calculated in modal space and can be viewed as the damping response of a second-order system in joint space. A value of 1 means critical damping, greater than 1 means overdamping, and less than 1 means underdamping. This describes step-response analysis; for a continuous system, underdamping can provide a degree of stability.
    
    This mode is suitable for assembly, polishing, and obstacle-avoidance tasks requiring joint-level compliance. It can absorb impacts and adapt to irregular surfaces.
    
    Enabling conditions:
    C/C++ (connection omitted; only mode-switching code is shown):
```c
// Set the key joint impedance parameters
double k[7] = {12, 12, 12, 10, 9, 9, 7};
double d[7] = {0.2, 0.2, 0.2, 0.2, 0.2, 0.2, 1};
OnClearSet();
OnSetJointLmt_A(10, 10);
OnSetJointKD_A(k, d);
OnSetJointLmt_B(10, 10);
OnSetJointKD_B(k, d);
OnSetSend();
SLEEP(200);
// Switch to joint impedance control mode
OnClearSet();
OnSetTargetState_A(3); 
OnSetImpType_A(1);
OnSetTargetState_B(3); 
OnSetImpType_B(1);
OnSetSend();
SLEEP(1000);
```
    Python (connection omitted; only mode-switching code is shown):
```python
'''Set the key joint impedance parameters'''
robot.clear_set()
robot.set_joint_kd_params(arm='A',K=[12, 12, 12, 10, 9, 9, 7], D=[0.3,0.3,0.3,0.2,0.2,0.2,0.2]）
robot.set_vel_acc(arm='A',velRatio=10, AccRatio=10)
robot.set_joint_kd_params(arm='B',K=[12, 12, 12, 10, 9, 9, 7], D=[0.3,0.3,0.3,0.2,0.2,0.2,0.2]）
robot.set_vel_acc(arm='B',velRatio=10, AccRatio=10)
robot.send_cmd()
time.sleep(0.2)
'''Switch to joint impedance mode'''
robot.clear_set()
robot.set_state(arm='A',state=3)
robot.set_impedance_type(arm='A',type=1) 
robot.set_state(arm='B',state=3)
robot.set_impedance_type(arm='B',type=1) 
robot.send_cmd()
time.sleep(1)
```

### Cartesian Impedance Mode
    Builds a compliant control model in the end-effector Cartesian space (X/Y/Z directions, rotational axes, and null space), giving the end effector adjustable stiffness and damping in response to external forces.
    
    The user must first set these parameters:
        Translational stiffness (range 0~1200 N*m) and damping (range 0~1, recommended 0.3);
        Rotational stiffness (range 0~600 N*m/rad) and damping (range 0~1, recommended 0.3);
        Overall null-space stiffness (range 20~100 N*m/rad) and overall null-space damping coefficient (range 0~1, recommended 0.3).

    Cartesian impedance damping is calculated in Cartesian modal space and can be viewed as the damping response of a second-order system in Cartesian space. A value of 1 means critical damping, greater than 1 means overdamping, and less than 1 means underdamping. This describes step-response analysis; for a continuous system, underdamping can provide a degree of stability.
        
    This mode is suitable for end-effector interaction with the environment, such as grinding, deburring, and force-controlled assembly. It can actively comply with external forces while maintaining trajectory precision, improving contact safety.
     
    Enabling conditions:
    C/C++ (connection omitted; only mode-switching code is shown):
```c
// Set the key Cartesian impedance parameters
double k[7] = {10000, 10000, 10000, 600, 600, 600, 20};
double d[7] = {0.2, 0.2, 0.2, 0.2, 0.2, 0.2, 1};
OnClearSet();
OnSetJointLmt_A(10, 10);
OnSetJointKD_A(k, d);
OnSetJointLmt_B(10, 10);
OnSetJointKD_B(k, d);
OnSetSend();
SLEEP(200);
// Switch to Cartesian control mode
OnClearSet();
OnSetTargetState_A(3); 
OnSetImpType_A(2);
OnSetTargetState_B(3); 
OnSetImpType_B(2);
OnSetSend();
SLEEP(1000);
```
    Python (connection omitted; only mode-switching code is shown):
```python
'''Set the key Cartesian impedance parameters'''
robot.clear_set()
robot.set_joint_kd_params(arm='A',K=[10000, 10000, 10000, 600, 600, 600, 20], D=[0.2, 0.2, 0.2, 0.2, 0.2, 0.2, 1]）
robot.set_vel_acc(arm='A',velRatio=10, AccRatio=10)
robot.set_joint_kd_params(arm='B',K=[10000, 10000, 10000, 600, 600, 600, 20], D=[0.2, 0.2, 0.2, 0.2, 0.2, 0.2, 1]）
robot.set_vel_acc(arm='B',velRatio=10, AccRatio=10)
robot.send_cmd()
time.sleep(0.2)

'''Switch to Cartesian impedance mode'''
robot.clear_set()
robot.set_state(arm='A',state=3)
robot.set_impedance_type(arm='A',type=2) 
robot.set_state(arm='B',state=3)
robot.set_impedance_type(arm='B',type=2) 
robot.send_cmd()
time.sleep(1)
```

### Force-controlled Impedance Mode
    Performs closed-loop control directly targeting the desired contact force in specified Cartesian directions (X, Y, and Z), while retaining impedance compliance.
    
    The user must first set these parameters:
        Force-control direction: a single direction or a combination of directions;
        Force-control range: 0~50 N;
        Force action distance (the allowable displacement deviation window): -50 mm ~ +50 mm.
        
    This mode is suitable for constant-force tracking. Parameters can be adjusted to accommodate surface variations and keep the contact force stable and controllable.
     
    Enabling conditions:
    C/C++ (connection omitted; only mode-switching code is shown):
```c
// Set end-effector force-control parameters: force control along Z
int fcType=0;// Base-frame force control
double fcCtrlPara[7] = {0.0};
double fxDir[6] = {0, 0, 1, 0, 0, 0};
double fcAdjLmt = 50;
double force = 10;
OnClearSet();
OnSetForceCtrPara_A(0, fxDir, fcCtrlPara, fcAdjLmt);
OnSetForceCmd_A(force);
OnSetSend();
SLEEP(200);
// Switch to force-control mode
OnClearSet();
OnSetTargetState_A(3); 
OnSetImpType_A(3);
OnSetTargetState_B(3); 
OnSetImpType_B(3);
OnSetSend();
SLEEP(1000);

```
    Python (connection omitted; only mode-switching code is shown):
```python
'''Set force-control parameters'''
robot.clear_set()
# Set an adjustment range of 5 centimeters along the Y axis
robot.set_force_control_params(arm='A',fcType=0, fxDirection=[0, 1, 0, 0, 0, 0], fcCtrlpara=[0, 0, 0, 0, 0, 0, 0],
                                        fcAdjLmt=5.)
time.sleep(0.5)
'''Switch to force-control mode'''
robot.clear_set()
robot.set_state(arm='A',state=3)
robot.set_impedance_type(arm='A',type=3) 
robot.set_state(arm='B',state=3)
robot.set_impedance_type(arm='B',type=3) 
robot.send_cmd()
time.sleep(1)
```
### PD feedforward mode
    PD mode provides extremely low-latency tracking together with the compliance of joint impedance mode. It is mainly intended for teleoperation.
    The speed of each joint in the user's joint trajectory must not exceed 180 degrees/second.
    Enabling conditions:
        - Set JointPIDCtlType=1 in the robot.ini configuration file;
        - Use joint impedance mode and set speed and acceleration to their maximum values to avoid limiting the trajectory;
            Set the joint impedance parameters:
                Stiffness (N*m/deg): maximum [20, 20, 20, 15, 8, 8, 8], minimum [2, 2, 2, 1.5, 0.8, 0.8, 0.8], commonly used [14, 14, 14, 10.5, 5.6, 5.6, 5.6]
                Damping coefficients [0.3, 0.3, 0.3, 0.3, 0.3, 0.3, 0.3]
            Set speed and acceleration to 100 to avoid limiting the trajectory.
            Switch to joint impedance.

        - Before sending the trajectory, enable feedforward control through FX_OnSetVelEstStep().

    Damping in PD mode is a scaling factor for actual damping and is related to maximum torque and maximum speed. Actual damping can be calculated as D=1.5*(1+useD)*(Torque/Velmax), where useD is the user-supplied damping parameter and D is the calculated damping, which is then multiplied by the velocity error.
     
    Enabling conditions:
    C/C++ (connection omitted; only mode-switching code is shown):
```c
// Switch to joint impedance, set speed and acceleration to maximum, and set stiffness and damping
// Three recommended stiffness parameter sets; choose as needed
double k_max[7] = {20, 20, 20, 15, 8, 8, 8};//max
double k_min[7] = {2, 2, 2, 1.5, 0.8, 0.8, 0.8 };//min
double k_normal[7]={ 14, 14, 14, 10.5, 5.6, 5.6, 5.6}
double d[7] = { 0.3, 0.3, 0.3, 0.3, 0.3, 0.3, 0.3};
OnClearSet();
OnSetJointLmt_A(100,100)
OnSetJointKD_A(k_normal, d)
OnSetJointLmt_B(100, 100);
OnSetJointKD_B (k_normal, d)
OnSetTargetState_A(3)
OnSetImpType_A(1)
OnSetTargetState_B(3)
OnSetImpType_B(1)
OnSetSend();
SLEEP(200);
// Enable PD feedforward
// The control period ControlPeriod ranges from 0~20 ms; 0 disables PD feedforward. A value of 5 ms is recommended, and the transmitted trajectory speed should stay within the maximum speed limit as far as possible.
int ControlPeriod = 5;
OnClearSet();
FX_OnSetVelEstStep("A"，ControlPeriod);
FX_OnSetVelEstStep("B"，ControlPeriod);
OnSetSend();
SLEEP(1000);
```
    Python (connection omitted; only mode-switching code is shown):
```python
'''Switch to joint impedance, set speed and acceleration to maximum, and set stiffness and damping'''
# Three recommended stiffness parameter sets; choose as needed
k_max=[20, 20, 20, 15, 8, 8, 8]
k_min=[2, 2, 2, 1.5, 0.8, 0.8, 0.8]
k_normal=[ 14, 14, 14, 10.5, 5.6, 5.6, 5.6]
d=[0.3, 0.3, 0.3, 0.3, 0.3, 0.3, 0.3]
robot.clear_set()
robot.set_joint_kd_params(arm='A',K=k_normal, D=d）
robot.set_vel_acc(arm='A',velRatio=100, AccRatio=100)
robot.set_joint_kd_params(arm='B',K=k_normal, D=d）
robot.set_vel_acc(arm='B',velRatio=100, AccRatio=100)
robot.send_cmd()
time.sleep(0.2)
robot.clear_set()
robot.set_state(arm='A',state=3)
robot.set_impedance_type(arm='A',type=1) 
robot.set_state(arm='B',state=3)
robot.set_impedance_type(arm='B',type=1) 
robot.send_cmd()
time.sleep(1)
'''Enable PD feedforward
The control period ControlPeriod ranges from 0~20 ms; 0 disables PD feedforward. A value of 5 ms is recommended, and the transmitted trajectory speed should stay within the maximum speed limit as far as possible.'''
ControlPeriod = 5
robot.clear_set()
robot.set_PD_vel_est_step(arm='A',step=ControlPeriod)
robot.set_PD_vel_est_step(arm='B',step=ControlPeriod)
robot.send_cmd()
time.sleep(1)
```
    
### Collaborative Release Mode
    A safety response mode designed for human-robot collaboration. When a collision is detected or external force exceeds the threshold, the robot immediately stops and actively releases the braking torque of all joints, placing each axis in a zero-force floating state to minimize collision impact energy. The operator can also trigger this mode manually for emergency disengagement or manual drag teaching. The robot must be enabled again after recovery before operation can continue.
     
    Enabling conditions:
    C/C++ (connection omitted; only mode-switching code is shown):
```c
// Enable collaborative release mode
OnClearSet();
OnSetTargetState_A(4) ;
OnSetTargetState_B(4) ;
OnSetSend();
SLEEP(1000);
// After enabling, drag the arm to adjust its position, then reset when finished
OnClearSet();
OnSetTargetState_A(0) ;
OnSetTargetState_B(0) ;
OnSetSend();
SLEEP(1000);
```
    Python (connection omitted; only mode-switching code is shown):
```python
'''Enable collaborative release mode'''
robot.clear_set()
robot.set_state(arm='A',state=1)
robot.set_state(arm='B',state=1)
robot.send_cmd()
time.sleep(1)
'''After enabling, drag the arm to adjust its position, then reset when finished'''
robot.clear_set()
robot.set_state(arm='A',state=0)
robot.set_state(arm='B',state=0)
robot.send_cmd()
time.sleep(1)
```

### Position-Velocity-Time Replay Mode (PVT)
    Using trajectory points recorded through teaching or offline programming, each containing position, velocity, and timestamp information, this mode reproduces the complete motion path in its original time sequence in joint or Cartesian space through high-precision interpolation. PVT mode ensures trajectory continuity and smooth velocity, making it suitable for repetitive tasks that must closely follow a taught path, such as spraying, grinding, and spot welding. Parameters include the interpolation period and speed and acceleration limits to ensure replay accuracy and dynamic performance.
    Note: First move the arm to the starting point of the trajectory.
         
    Enabling conditions:
    C/C++ (connection omitted; only mode-switching code is shown):
```c
// Set PVT mode
OnClearSet();
OnSetTargetState_A(2) ;
OnSetSend();
SLEEP(200);
// Select the PVT trajectory file and set the PVT ID
char path[] = "LoadData_ccs_right/LoadData/IdenTraj/LoadIdenTraj_MarvinCCS_Left.fmv"; // Change this to your absolute path
long serial=27;
bool re=false;
re=OnSendPVT_A(path,serial);
printf("send pvt return =%d\n",re);
SLEEP(200);
// Execute the specified PVT ID
int id=27;
OnClearSet();
OnSetPVT_A(id);
OnSetSend();
// Wait for trajectory execution to finish
```
    Python (connection omitted; only mode-switching code is shown):
```python
'''Set PVT mode'''
robot.clear_set()
robot.set_state(arm='A',state=2)# PVT uses its own speed and acceleration, unaffected by external control.
robot.send_cmd()
time.sleep(0.5)
'''Set the local PVT trajectory path and PVT ID'''
pvt_file='/LoadData_ccs_right/LoadData/IdenTraj/LoadIdenTraj_MarvinCCS_Left.fmv'
robot.send_pvt_file('A',pvt_file, 2)
time.sleep(1)
'''Set the PVT ID to execute'''
robot.clear_set()
robot.set_pvt_id('A',2)
robot.send_cmd()
# Wait for trajectory execution to finish
```



## 1.1 Robot control SDK documentation
[C++ control SDK documentation](c++_doc_contrl_EN.md)

[Python control SDK documentation](python_doc_contrl_EN.md)

    The documentation includes demo descriptions.

## 1.2 Robot kinematics SDK documentation
[C++ kinematics SDK documentation](c++_doc_kine_EN.md)

[Python kinematics SDK documentation](python_doc_kine_EN.md)

    The documentation includes demo descriptions.


## II. Compilation methods

### Development and compilation environment

        Windows: Windows 11, MinGW compiler tools
        Linux: ubuntu20.04（glibc2.31）x_86

        Python version: >=3.10

        The SDK has been tested on machines with x86 CPUs. For ARM machines, modify and compile it before use.

        The contrlSDK100343/contrlSDK and kinematicsSDK code is compatible with Windows and Linux.

### 2.1 Compilation
    2.1.1 Compiling SO shared libraries:
    Compile on a Linux device:
        Control SDK (contrlSDK100343), using either method:
			1. g++ *.cpp -Wall -O2 -fPIC -shared -o libMarvinSDK.so -lpthread -lrt -DCMPL_LIN
			2. Use ./contrlSDK100343/makefile to generate libMarvinSDK.so.
        Kinematics SDK (kinematicsSDK), using either method:
			1. g++ *.cpp -Wall -O2 -fPIC -shared -o libKine.so -lpthread -lrt 
			2. Use ./kinematicsSDK/makefile to generate libKine.so.
	The compiled libKine.so and libMarvinSDK.so are for use by C++ and Python on the machine where they were compiled.

    2.1.2 Compiling DLL libraries for C++:
    1) Use MinGW on Windows to compile the DLL libraries:
			Control SDK (contrlSDK100343): g++ *.cpp -Wall -O2 -shared -o libMarvinSDK.dll -lws2_32 -lwinmm -DCMPL_WIN
            Kinematics SDK (kinematicsSDK): g++ *.cpp -Wall -O2 -fPIC -shared -o libKine.dll
    The compiled libKine.dll and libMarvinSDK.dll are for use by C++ on Windows.

			
	2.1.3 Compiling DLL libraries for Python
    1) Compile DLL libraries on Linux:
        Control SDK (contrlSDK100343): x86_64-w64-mingw32-g++ *.cpp -Wall -O2 -shared -o libMarvinSDK.dll -DBUILDING_DLL -DCMPL_WIN -static -static-libgcc -static-libstdc++ -lws2_32 -lpthread -lwinmm
        Kinematics SDK (kinematicsSDK): g++ *.cpp -Wall -O2 -fPIC -shared -o libKine.dll

	2) Use MinGW on Windows to compile the DLL libraries:
			Control SDK (contrlSDK100343): g++ *.cpp -Wall -O2 -shared -o libMarvinSDK.dll -DBUILDING_DLL -D_WIN32 -DCMPL_WIN -fPIC -static -static-libgcc -static-libstdc++ -lws2_32 -lwinmm
			Kinematics SDK (kinematicsSDK): g++ *.cpp -Wall -O2 -shared -o libKine.dll -DBUILDING_DLL -D_WIN32 -fPIC -static -static-libgcc -static-libstdc++ -lws2_32 -lwinmm
    The compiled libKine.dll and libMarvinSDK.dll are for use by Python on Windows.

### 2.2 Automated compilation of shared libraries
    Using contrlSDK100343 and the kinematics library as an example:
	1) On Linux, use marvinSDK_ubuntu_100343.sh to automatically compile and replace the .so files.
			# Grant execution permission to the script
            chmod +xmarvinSDK_ubuntu_100343.sh
			# Run the automated build script
			./marvinSDK_ubuntu_100343.sh

	2) On Windows, use marvinSDK_windows_100343.bat to automatically compile and replace the .dll files.
			# Run the batch script directly
			./marvinSDK_windows_100343.bat
        
### 2.3 Usage examples
    LINUX:
        C++: 
            ./DEMO_C++/README_EN.md
        Python code is cross-platform; see DEMO_PYTHON/README_EN.md.

    WINDOWS:

        C++: 
            ./DEMO_C++/README_EN.md
        Python code is cross-platform; see DEMO_PYTHON/README_EN.md.


### 2.4 Using source code without shared libraries

    Using contrlSDK100343 as an example, suppose the calling code file main.cpp is in a workspace folder at the same directory level as contrlSDK100343.
    The directory tree is:
    ...
    |---contrlSDK100343
    |---workspace
        |--- main.cpp
    ...


    The compilation commands are:
    1) On Windows:
    g++ -Wall main.cpp ../contrlSDK100343/*.cpp -I../contrlSDK100343 -o main.exe -lws2_32 -lwinmm -DCMPL_WIN

    2) On Linux:
    g++ -Wall main.cpp ../contrlSDK100343/*.cpp -I../contrlSDK100343 -o main -lpthread -lrt -DCMPL_LIN

    After compilation, the following is generated:
    ...
    |---contrlSDK100343
    |---workspace
        |--- main.cpp
        |--- main.exe or main
    ...


                 
## III. SDK updates
## contrlSDK is no longer maintained; maintenance and development continue for version 100343.

### contrlSDK100343 version
    1. contrlSDK100343 requires underlying control system version 100343 or later. It does not support versions before 100343, such as 100341.
    2. The main change in contrlSDK100343 is the communication protocol, with added validation to prevent garbled commands when interfaces are called from multiple threads.
    3. Existing demos on this branch and users' existing scripts do not need modification. Simply recompile SDK100343 and replace the original shared libraries. The compilation commands are unchanged.
    4. Note that control system version 100343 is not backward compatible and can only use SDK version 100343.

## 3.1 Example updates
### Convert joint torques to six-dimensional end-effector force
[Python example](DEMO_PYTHON/showcase_jointsTorque2EefTorque.py)

## 3.2 Control SDK
### New simplified control SDK interfaces
# Simplified interfaces are provided to make the control SDK easier to use.
[Original SDK interface introduction](c++_doc_contrl_EN.md#L118)

[Simplified interface introduction](c++_doc_contrl_EN.md#L840)

[Control SDK MarvinSDK.h](contrlSDK/MarvinSDK.h)

[Simplified control example in C++](DEMO_C++/showcase_new_control_sdk_usage.cpp)
[Simplified control example in Python](DEMO_PYTHON/showcases_new_control_sdk.py)
    
### Send joint commands using planning to eliminate jitter
    // Send commands using joint-space PLN
    FX_DLL_EXPORT bool OnInitPlnLmt(char * path);
	FX_DLL_EXPORT bool OnSetPlnJoint_A(double start_joints[7], double stop_joints[7],double vel_ratio,double acc_ratio);
	FX_DLL_EXPORT bool OnSetPlnJoint_B(double start_joints[7], double stop_joints[7],double vel_ratio,double acc_ratio);

### Send commands using planning to follow a straight line
    // Send commands using Cartesian-space PLN
	FX_DLL_EXPORT void* FX_CPointSet_Create();
	FX_DLL_EXPORT void FX_CPointSet_Destroy(void* pset);
	FX_DLL_EXPORT bool OnSetPlnCart_A(void* pset);
	FX_DLL_EXPORT bool OnSetPlnCart_B(void* pset);

### Interrupt planned motion
	FX_DLL_EXPORT bool OnStopPlnJoint_A();
	FX_DLL_EXPORT bool OnStopPlnJoint_B();

### Set the end-effector force-control type and rotation of the Cartesian directions
	// Set the left-arm force-control type to fcType=1. Cartesian directions: set the first three CartCtrlPara parameters to the end-effector rotation relative to the base in X Y Z order. The last four parameters are reserved; set them to 0.
	FX_DLL_EXPORT bool OnSetEefRot_A(int fcType, double CartCtrlPara[7]);
	// Set the right-arm force-control type to fcType=1. Cartesian directions: set the first three CartCtrlPara parameters to the end-effector rotation relative to the base in X Y Z order. The last four parameters are reserved; set them to 0.
	FX_DLL_EXPORT bool OnSetEefRot_B(int fcType, double CartCtrlPara[7]);

### Soft reset the servo of a specified joint
	// Soft reset the servo of a specified left-arm joint
	FX_DLL_EXPORT void OnServoReset_A(int axis);
	// Soft reset the servo of a specified right-arm joint
	FX_DLL_EXPORT void OnServoReset_B(int axis);

## 3.3 Kinematics SDK
### Updated online planning functions

     C++ interfaces:
        FX_BOOL  FX_Robot_PLN_MOVL(FX_INT32L RobotSerial, Vect6 Start_XYZABC, Vect6 End_XYZABC, Vect7 Ref_Joints, FX_DOUBLE Vel, FX_DOUBLE ACC, FX_INT32L Freq, FX_CHAR* OutPutPath);
        FX_BOOL  FX_Robot_PLN_MOVL_KeepJ(FX_INT32L RobotSerial, Vect7 startjoints, Vect7 stopjoints, FX_DOUBLE vel, FX_DOUBLE acc, FX_INT32L Freq, FX_CHAR* OutPutPath);
        FX_BOOL FX_Robot_PLN_MOVLA(FX_INT32L RobotSerial, Vect6 Start_XYZABC, Vect6 End_XYZABC,Vect7 Ref_Joints, FX_DOUBLE Vel, FX_DOUBLE ACC, FX_INT32L Freq, CPointSet* ret_pset);
        FX_BOOL  FX_Robot_PLN_MOVL_KeepJA(FX_INT32L RobotSerial, Vect7 startjoints, Vect7 stopjoints,FX_DOUBLE vel, FX_DOUBLE acc, FX_INT32L Freq, CPointSet* ret_pset);

     c++ demo: 
          1. Demonstrate the left arm's offline and online planning interfaces: showcase_online_and_offline_pln_all_function.cpp
          2. Execute an offline straight-line planning file for the left arm in joint impedance mode at 50 Hz: showcase_offline_movl_execution.cpp
          3. Execute online straight-line planning points for the left arm in joint impedance mode at 50 Hz: showcase_online_movla_execution.cpp
          4. Execute an offline straight-line planning file with a constrained configuration for the arm in joint impedance mode at 50 Hz: showcase_offline_movl_keepj_execution.cpp
          5. Execute online straight-line planning points with a constrained configuration for the left arm in joint impedance mode at 50 Hz: showcase_online_movl_keepja_execution.cpp

     Python interfaces:
        Straight-line interpolation planning
       - movL(start_xyzabc: list, end_xyzabc: list, ref_joints: list, vel: float, acc: float, freq_hz:int, save_path)
    
        Straight-line interpolation planning with constrained start and end joint configurations
        - movL_KeepJ(start_joints:list, end_joints:list,vel:float,acc: float,freq_hz:int, save_path)
    
          Online straight-line interpolation planning
        - movLA(start_xyzabc: list, end_xyzabc: list, ref_joints: list, vel: float, acc: float,freq_hz:int )
    
          Online straight-line interpolation planning with constrained start and end joint configurations
        - movL_KeepJA(start_joints:list, end_joints:list,vel:float,acc: float,freq_hz:int)

       py demo:
            showcase_online_pln_movl.py
            showcase_online_pln_movl_keepj.py
            showcase_online_pln_movl_with_specific_rot.py
          

### Get the controller version number through code
     C++:
          char paraName[30]="VERSION";
          long retValue=0;
          OnGetIntPara(paraName,&retValue);
          printf("CONTRL VERSION: %ld\n", retValue);

     PYTHON:
          ret,version=robot.get_param('int','VERSION')
          print(f'controller version:{version}')

     Displayed as 1003xx, for example 100335: major version 1003, subversion 35.



## IV. Controller version updates

     Features added in version 1003_37:
     1. Added axis external-force detection in any state. These axis external forces can be used to calculate the external force on the end effector.

     
    Features added in version 1003_35:
    1. Added internal and external encoder detection.
    2. Fixed all axes being disabled after a servo error.
    https://github.com/cynthia-you/TJ_FX_ROBOT_CONTRL_SDK/releases/tag/marvin_tool_1003_35

    
    Features added in version 1003_34:
    1. Zero internal and external encoders and clear encoder errors.
    2. Added support for position-only control through parameters R.A0.BASIC.CtrlType and R.A1.BASIC.CtrlType. 0 enables all control modes; 1 enables position control only (edit in the robot configuration file *.ini).
    
    These features have also been updated in MARVIN_APP and FX-STATION.

    1003_34 URL:
        https://github.com/cynthia-you/TJ_FX_ROBOT_CONTRL_SDK/releases/tag/marvin_tool_1003_34
        

### 4.1 Example of zeroing robot motor internal/external encoders and clearing internal encoder errors
    The controller must be upgraded to version 1003_34.
       
### 4.2 Updated versions and parameters are published under releases
    https://github.com/cynthia-you/TJ_FX_ROBOT_CONTRL_SDK/releases


## V. App updates
[MarvinPlatform source code](MarvinPlatform_EN/ui_EN.py)
[MarvinPlatform Windows host application](MarvinPlatform_EN\MarvinPlatform_win_100343.exe)
[MarvinPlatform Linux host application](MarvinPlatform_EN\MarvinPlatform_linux_100343)
[MarvinPlatform usage instructions](MarvinPlatform_EN/天机Marvin系列MarvinPlatform软件使用说明2601.pptx)


## VI. Precautions
    1. Successful communication with the robot does not mean data transmission and reception have started. The controller begins sending periodic status data to the host at 1000 Hz only after receiving transmitted data.

    2. Do not use the application and SDK simultaneously. Do not use the application and SDK simultaneously, to prevent port conflicts and data transmission/reception failures.

    3. Before use, configure the network interface to be on the same subnet as the controller.

    4. Releasing the robot disconnects it and relinquishes control. You must reconnect to use it again.

    5. The robot consists of servo drives and a controller. We recommend connecting both power supplies to one power strip to simplify powering them on/off and restarting them together. After restarting, allow 30-60 seconds for warm-up before operating the robot to avoid unresponsive servos.

    6. After use, you must release the robot through code or the application (the release interface, the application's disconnect button, or closing the application). Otherwise, a process that has not released the robot may prevent connections and subscriptions from other processes from taking effect.

    7. In the C++ control SDK, suffix _A indicates the left arm and _B the right arm. If you have only one arm, it is _A, the left arm.

    8. Clear errors when the subscribed robot state is 100 or when the subscribed robot error indicates a servo error.

    9. End-effector module (485/CAN) control: be sure to use the module supplier's manual and test software. Test the control commands before sending protocol commands through our SDK.



## VII. Main problems and solutions
### 7.1 MARVIN SDK and app problems and solutions
     [Tencent Docs] MARVIN SDK & APP problem collection and solutions
     https://docs.qq.com/sheet/DUmdJck1zQkJVT0tw

### 7.2 Other common problems
    1. Connection
    Q: Why can't I ping the robot?
    A: Check whether the network cable is connected, whether other devices or processes are occupying the connection, and whether a static IP on the same subnet as the robot controller has been configured.

    2. Subscriptions
    Q: Why does the robot subscription interface return no data, only zeros?
    A: Connect to the robot before subscribing, then sleep for half a second to receive real-time data. Check whether another process such as ROS is occupying the subscription process and whether the firewall is disabled.

    3. Repeated callbacks
    Q: Why do repeated CALLBACK calls not work, with motion occurring only the first time?
    A: Connecting to and releasing the robot do not require repeated callbacks. At high frequencies, the servo cannot respond in time and reports errors. Motion commands can be sent at frequencies below 1 kHz.

    4. Determining motion status
    Q: How can I determine in code whether the robot has reached my specified point?
    A: In C++, subscribe to the data interface and check m_FB_Joint_Pos in the subscribed data structure, or check the robot's low-speed flag m_LowSpdFlag.
        When the speed of every joint is below 0.5 degrees/second, m_LowSpdFlag=1.

        In Python, check sub_data["outputs"][0]["fb_joint_pos"] in the subscribed data structure to determine whether the target has been reached,
    or check sub_data["outputs"][0]["low_speed_flag"]. When the speed of every joint is below 0.5 degrees/second, low_speed_flag=1.

    5. Determining robot states and errors
    C++: The integer value of m_CurState in the subscribed data indicates the current arm state:
        0,             //////// Servo disabled
        1,             //////// Position following
        2,				//////// PVT
        3,             //////// Torque
        4,             //////// Collaborative release

        100, // Error; clear the error
        ARM_STATE_TRANS_TO_POSITION = 101, // Normal, during the transition
        ARM_STATE_TRANS_TO_PVT = 102,// Normal, during the transition
        ARM_STATE_TRANS_TO_TORQ = 103,// Normal, during the transition
        ARM_STATE_TRANS_TO_TORQ = 104,// Normal, during the transition

        The subscribed m_ERRCode data consists of 7 double values in decimal.
        Convert them to hexadecimal and consult the servo error Excel file to identify the error.
        The application already converts them to hexadecimal; the C++ interface returns raw data.

        The integer value of m_ERRCode in the subscribed data indicates the current arm error state:
             ARM_ERR_BusPhysicAbnoraml = 1, // "Bus topology abnormal": EtherCAT communication is disconnected or in another error state
             ARM_ERR_ServoError = 2,// "Servo fault": 1) An axis is in a fault state, 2) Incorrect axis parameter configuration, 3) Axis communication error
             ARM_ERR_InvalidPVT = 3,// "PVT abnormal": 1) Internal data read error or length mismatch in PVT mode, 2) Linux scheduling causes a data exchange error between PSI and SI processes in position mode
             ARM_ERR_RequestPositionMode = 4,// "Request to enter position mode failed": 1) Servo initialization state error, 2) Servo state transition in progress, 3) Encoder state error, 4) Servo feedback state transition failed, 5) Emergency stop active, 6) For versions before 100341, same cause as 6
             ARM_ERR_PositionModeOK = 5,// "Entering position mode failed": 1) Servo feedback operating-mode switch failed, 2) Motor state error, 3) Controller system memory state error, 4) Incorrect internal arm-count setting in the controller
             ARM_ERR_RequestSensorMode = 6,// "Request to enter torque mode failed": 1) Tool dynamics parameters not set, 2) Arm in hard contact with the external environment, 3) Other causes are the same as 4 and 5
             ARM_ERR_SensorModeOK = 7,// "Entering torque mode failed": Same causes as 4 and 5
             ARM_ERR_RequestEnableServo = 8,// "Request to enable servo failed": Same causes as 4 and 5
             ARM_ERR_EnableServoOK = 9,// "Enabling servo failed": Same causes as 4 and 5

             ARM_ERR_RequestDisableServo = 10, // "Request to disable servo failed": Same causes as 4 and 5
             ARM_ERR_DisableServoOK = 11, // "Disabling servo failed": Same causes as 4 and 5
             ARM_ERR_InvalidSubState = 12, // "Internal error": 1) Operating system memory or scheduling error, 2) Variable calculation error, 3) Some memory pointers are null
             ARM_ERR_Emcy = 13, // "Emergency stop"
             ARM_DYNA_FLOAT_NO_GYRO = 14,// "Floating-base option selected in the configuration file, but no IMU hardware is connected to the controller"
             ARM_ERR_PdoAbnormal = 15, // "PDO not working normally"


    Python: The subscribed value a_state=sub_data["states"][0]["cur_state"] indicates the current servo state:
        0,             //////// Servo disabled
        1,             //////// Position following
        2,				//////// PVT
        3,             //////// Torque

        ARM_STATE_ERROR = 100, // Error; clear the error
        ARM_STATE_TRANS_TO_POSITION = 101, // Normal, during the transition
        ARM_STATE_TRANS_TO_PVT = 102,// Normal, during the transition
        ARM_STATE_TRANS_TO_TORQ = 103,// Normal, during the transition



        The subscribed value a_state=sub_data["states"][0]["err_code"] indicates the current arm error state:
             ARM_ERR_BusPhysicAbnoraml = 1, // "Bus topology abnormal"
             ARM_ERR_ServoError = 2,// "Servo fault"
             ARM_ERR_InvalidPVT = 3,// "PVT abnormal"
             ARM_ERR_RequestPositionMode = 4,// "Request to enter position mode failed"
             ARM_ERR_PositionModeOK = 5,// "Entering position mode failed"
             ARM_ERR_RequestSensorMode = 6,// "Request to enter torque mode failed"
             ARM_ERR_SensorModeOK = 7,// "Entering torque mode failed"
             ARM_ERR_RequestEnableServo = 8,// "Request to enable servo failed"
             ARM_ERR_EnableServoOK = 9,// "Enabling servo failed"
             ARM_ERR_RequestDisableServo = 10, // "Request to disable servo failed"
             ARM_ERR_DisableServoOK = 11, // "Disabling servo failed"
             ARM_ERR_InvalidSubState = 12, // "Internal error"
             ARM_ERR_Emcy = 13, // "Emergency stop"
             ARM_DYNA_FLOAT_NO_GYRO = 14,// "Floating-base option selected in the configuration file, but the UMI setting is not enabled in the configuration file"

        Use error_codes=get_servo_error_code('A') to get errors.
        Consult the servo error PDF to identify the error.
        The application and Python already convert the values to hexadecimal; the C++ interface returns raw data.

    6. Commands do not respond after an emergency stop
    The servos are automatically disabled after an emergency stop. Clear errors, then enable the servos again.

    7. End-effector gripper communication
    Currently only modbus485 and CAN/CANFD communication are supported.
    !!! Do not send demo commands directly to an end-effector gripper or dexterous hand. Protocol differences may cause the module to freeze. Be sure to use the module supplier's manual and test software, and establish the correct control commands before sending them.
    Points to note:
        Send HEX data to CAN.
        Check the command protocol provided by the control module:
            If a 32-bit CANID is 0x01, send it in HEX as: 01 00 00 00
            If a 64-bit CANID is 0x01, send it in HEX as: 01 00

## 📄 License

This project is open source under the Apache License 2.0. See [LICENSE](LICENSE) for details.
