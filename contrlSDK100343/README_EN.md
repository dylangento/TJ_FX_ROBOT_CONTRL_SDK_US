# Control SDK 100343

## Attention

1. SDK `100343001` requires controller version `100343` or later. It does not support earlier controller versions such as `100342`.
2. This version updates the communication protocol and adds validation to prevent command corruption caused by concurrent API calls.
3. Existing demos and user scripts do not need to change. Rebuild and replace the SDK100343 dynamic library using the same build commands.
4. Controller version `100343` is not backward compatible.
5. Read and test the demos before using them with a real robot. Their default velocity and acceleration are intentionally limited to 10%.

## 1. SDK documentation

[SDK home](../README_EN.md)

[C++ control SDK documentation](../c++_doc_contrl_EN.md)

[Python control SDK documentation](../python_doc_contrl_EN.md)

[C++ kinematics SDK documentation](../c++_doc_kine_EN.md)

[Python kinematics SDK documentation](../python_doc_kine_EN.md)

## 2. Build SDK libraries

The control SDK header is `MarvinSDK.h`.

### 2.1 Build shared libraries on Linux

```text
g++ *.cpp -Wall -O2 -fPIC -shared -o libMarvinSDK.so -lpthread -lrt -DCMPL_LIN
```

### 2.2 Build Windows DLLs for C++

```text
g++ *.cpp -Wall -O2 -shared -o libMarvinSDK.dll -lws2_32 -lwinmm -DCMPL_WIN
```

### 2.3 Build Windows DLLs for Python

Use the MinGW command appropriate to your host:

```text
x86_64-w64-mingw32-g++ *.cpp -Wall -O2 -shared -o libMarvinSDK.dll -DBUILDING_DLL -DCMPL_WIN -static -static-libgcc -static-libstdc++ -lws2_32 -lpthread -lwinmm
g++ *.cpp -Wall -O2 -shared -o libMarvinSDK.dll -DBUILDING_DLL -D_WIN32 -DCMPL_WIN -fPIC -static -static-libgcc -static-libstdc++ -lws2_32 -lwinmm
```
