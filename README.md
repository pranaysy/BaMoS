# BaMoS Toolbox
This readme documents some basic steps for compiling the BaMoS toolbox in a Linux environment

# Prepare environment
The following instructions assume the operating system is a recent Debian-based one (such as Debian 13 "Trixie" or Ubuntu 24.04 etc). All steps are to be executed in an interactive terminal.

## Make sure some packages are installed on the system
```bash
sudo apt install build-essential cmake git
```

(Optionally install python at a system level or in a conda/pip/uv environment)

## Use this repo, or fork the original one on GitHub, or use your own fork
 - Original: https://github.com/csudre/BaMoS
 - This fork: https://github.com/pranaysy/BaMoS
 - Your fork: https://github.com/your_user_name/BaMoS
Note that git clone later uses SSH connection and not https, adjust URL path accordingly

## Clone the repository
```bash
cd ~/Projects
git clone git@github.com:pranaysy/BaMoS.git
cd BaMoS
```

# Compile and build
The BaMoS code uses `cmake` to handle the build process. A note on the flags used here with cmake, the following are specified by CMakeLists.txt in the BaMoS source repository:
 - `BUILD_ALL=ON`: Required to build the LoAd segmentation tool
 - `BUILD_SHARED=OFF`: Build internal libraries as 'static' archives (ending in .a) instead of 'shared' objects (ending in .so) and link them to the executable binaries produced.
 - `USE_OPENMP`=ON: Build with OpenMP support for parallel compute
(The file also specifies the flags `INSTALL_NIFTYREG` and `INSTALL_PRIORS`, but the former didn't work out of the box, and the latter expects some data in a 'priors' folder, not sure where they're supposed to come from)

The following additional flags enable optimisations for the specific CPU on which the package is being compiled.
 - `CMAKE_BUILD_TYPE=Release`: Enable release level optimisations, this sets O3 and disables debug features
 - `CMAKE_INSTALL_PREFIX`: Install built binaries and libraries to this path, differs for user and system installations
 - `CMAKE_C_FLAGS="-march=native -mtune=native"`: Optimize for native CPU instruction set
 - `CMAKE_CXX_FLAGS="-march=native -mtune=native"`: Optimize for native CPU instruction set

The build process can be configured for a user-level installation or a system-wide installation (if desired, and if sudo available).

## OPTION A: USER INSTALLATION
### Configure compilation and build
```bash
cmake -S . -B ./build -DBUILD_ALL=ON -DUSE_OPENMP=ON -DBUILD_SHARED=OFF -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=$PWD/build/install -DCMAKE_C_FLAGS="-march=native -mtune=native" -DCMAKE_CXX_FLAGS="-march=native -mtune=native"

cmake --build build --parallel $(nproc)  # Compile with all available cores
cmake --install build   # Perform installation to specified path
```
### Check install manifest
```bash
cat build/install_manifest.txt
```

### Check if compiled binaries work
```bash
$PWD/build/install/bin/Seg_BiASM --help
$PWD/build/install/bin/Seg_Analysis --help
```
### Configure environment to find these binaries (add to .bashrc for persistence)
```bash
export PATH="$PWD/build/install/bin:$PATH"
```
## OPTION B: SYSTEM INSTALLATION
### Configure compilation and build
```bash
cmake -S . -B build -DBUILD_ALL=ON -DUSE_OPENMP=ON -DBUILD_SHARED=OFF -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=/usr/local -DCMAKE_C_FLAGS="-march=native -mtune=native" -DCMAKE_CXX_FLAGS="-march=native -mtune=native"

cmake --build build --parallel $(nproc)  # Compile with all available cores
sudo cmake --install build   # Perform installation to specified path
```
### Check install manifest
```bash
cat build/install_manifest.txt
```

### Check if binaries are correctly found on system path
```bash
which Seg_Analysis Seg_BiASM
```

### Check if compiled binaries work
```bash
Seg_Analysis --help
Seg_BiASM --help
```
