# Prerequisites
1. Install [MSYS2](https://www.msys2.org/) and all dependencies (install mingw64/* builds).
  1. Don't forget to install `mingw64/x86_64-w64-mingw32-cmake`
2. Get patched [Surelog](https://github.com/RustamC/Surelog/tree/win-static-build-024)
3. Get patched [Yosys](https://github.com/RustamC/yosys/tree/win-static-build-024)

# Build
1. Open Powershell
2. Add msys2 & mingw64 to path. In my case msys is installed in `D:\msys64`. **IMPORTANT:** If you have Windows Cmake in `Path`, remove it!!!:

```powershell
$originalPath=$Env:Path
Set-Item -Path Env:Path -Value ("D:\msys64\mingw64\bin;D:\msys64\usr\bin;" + $Env:Path)
```

## Surelog
Build & install. Don't forget to set `-DCMAKE_INSTALL_PREFIX`, in my case it's the same dir where yosys will be installed:

```powershell
cd Surelog
cmake -G "MinGW Makefiles" -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=D:/opt/yosys -DCMAKE_POSITION_INDEPENDENT_CODE=ON -S . -B build
cmake --build build 
cmake --install build
```

## Yosys
1. Install GHDL in MSYS2.2. 
2. Create in yosys dir new `Makefile.conf` file:

```Makefile
CONFIG := msys2-64
PREFIX := D:/opt/yosys
ENABLE_GHDL := 1
GHDL_PREFIX := D:/msys64/mingw64

ENABLE_SV := 1
UHDM_INSTALL_DIR := D:/opt/yosys
```

3.Just run make & make install:
```powershell
make install -j4
```

4. Copy required MSYS2 dlls:
```powershell
ldd .\yosys.exe | grep mingw64 | awk 'NF == 4 { system("cp --no-clobber " $3 " /d/opt/yosys/bin") }'
```

5. Copy ghdl libs from `D:\msys64\mingw64\lib\ghdl` to `D:\opt\yosys\share\yosys`

# Changes
## Yosys
1. Added ghdl-yosys-plugin for static build: https://github.com/ghdl/ghdl-yosys-plugin/
2. Added systemverilog-plugin for static build: https://github.com/chipsalliance/yosys-f4pga-plugins

	2.1. Added new Makefile.inc

	2.2. Renamed some functions by adding `sv_` prefix to avoid clashes with verilog plugin.

	2.3. Added `ENABLE_SV` into Makefile

	2.4. CXXFLAGS:
    - Changed `-std=c++17` to `-std=gnu++17`
    - Added `-DWIN32_LEAN_AND_MEAN` to avoid names clashes with Windows std headers
  
	2.5. LDFLAGS:
    - Added to LDLIBS 
  
	2.6. LDLIBS:
    - Renamed `-lutil` to `-luuid`
