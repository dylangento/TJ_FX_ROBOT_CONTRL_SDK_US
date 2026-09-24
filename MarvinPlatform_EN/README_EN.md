# MarvinPlatform

## Before using the application

Place exactly one unique robot model configuration file in `../CommonConfig/config`:

- CCS 6 kg, version 3.1: `ccs_m6_31.MvKDCfg`
- CCS 6 kg, version 4.0: `ccs_m6_40.MvKDCfg`
- CCS 3 kg: `ccs_m3.MvKDCfg`
- SRS: `srs.MvKDCfg`

Multiple `*.MvKDCfg` files will cause parsing errors.

## Install and run

1. Tested executable packages are provided for Windows and Ubuntu 20.04 x86. For other environments, build the libraries from source.
2. The basic environment requires Python 3.10 or later, PyInstaller, and Pillow.
3. Confirm that `SDK_PYTHON` contains the latest `libMarvinSDK.so` and `libKine.so` built from `contrlSDK100343` and `kinematicsSDK`.
4. Run the source application with `ui_EN.py`.
5. To package an executable for computers without Python, run `python setup.py`.

## Application documentation

[MarvinPlatform user guide](天机Marvin系列MarvinPlatform软件使用说明2601.pptx)
