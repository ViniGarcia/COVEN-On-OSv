# COVEN-On-OSv: a Flexible Platform for Running Vitualized Network Functions

COVEN-On-OSv (COO) is a novel platform that expands the capabilities of the previously proposed [Click-On-OSv](https://github.com/lmarcuzzo/click-on-osv)[1] platform. In this project, we implemented the concept of Virtual Network Subsystem (VNS) and Packet Processing Subsystem (PPS) from [COmprehensive VirtualizEd NF](https://github.com/ViniGarcia/COVEN)[2]. In the current version, the COO enables the users to use two virtual network tools ([VirtIO](https://www.linux-kvm.org/page/Virtio) and [DPDK](https://www.dpdk.org/)) and two packet processing frameworks ([Python 2.7](https://www.python.org/download/releases/2.7) and [Click Modular Router](https://github.com/kohler/click)). These components are compiled into the unikernel [OSv](https://github.com/cloudius-systems/osv).

## Limitations
The compilation has been executed in a Debian 8 environment. An error can be raised when linking DPDK and Click in a compiling environment with Glib > 2.19 (it can be tested with ```ldd -d click``` on the Click binary that should show a libintel_dpdk.so library on the dependencies). The GCC compiler must be at least version 4.8. The GCC versions of 5.x and 6.x raise error while compiling DPDK.

## Platform Usage

A precompiled version is available for download as a "qcow2" disk or an "ova" appliance. These images and the installation and usage guide are located in [images](./images).

## Platform Compilation

"build.sh" script is used to compile DPDK, Click and OSv and provides in **binary** folder the DPDK library (*libintel_dpdk.so*) and Click binary (*click*), as well images in (*images*). Example VNFs can be found in [click_confs](click_confs) folder.

Before compiling, download the submodules. From the root folder, run:

```
git submodule update --init --recursive
```

You should install the dependencies requested by the OSv before compiling the COO platform. This installation can be done by running scritps/setup.py inside the [osv](./osv) folder.

### DPDK Packet Acceleration

For DPDK's compiling process:

Environment variables definition at root folder ->
```
#DPDK's Folder
export RTE_SDK=`readlink -f osv-dpdk`
#DPDK's Target
export RTE_TARGET=x86_64-native-osvapp-gcc
#OSv's Folder
export OSV_SDK=`readlink -f osv`
```

After setting environment variables and initializing submodules ->
```
cd osv-dpdk
#Checkout OSv branch
git checkout osv-head
#Compile
make install T=$RTE_TARGET OSV_SDK=$OSV_SDK
```

A folder with the target name will be generated with the necessary library. Copy this library to [binary](./binary) ->
```
cp -fa x86_64-native-osvapp-gcc/lib/libintel_dpdk.so ../binary
```

### Click Modular Router

With DPDK's and OSv's environment variables already defined, initialized submodules and compiled DPDK library ->
```
cd click
#If there is an old build, clean it
make clean
#Configure click to compile with DPDK, OSv and fPIC (for shared libraries) support
./configure --enable-dpdk --enable-osv --enable-user-multithread --disable-linuxmodule CXXFLAGS="-fPIC -std=gnu++11" CFLAGS="-fPIC" LDFLAGS="-fPIC -std=gnu++11" CPPFLAGS="-fPIC -std=gnu++11"
#Compile userlevel application
cd userlevel
make
#Copy the generated binary to [binary](./binary) folder
cp -fa click ../../binary
```

### OSv Operational System

With DPDK and OSv compiled ->
```
#Copy binaries to OSv's modules folder
cp binary/* osv/modules/click
#Execute OSv's dependencies setup script
cd osv
./scripts/setup.py
#Compile OSv with Click and Web Interface modules
./scripts/build modules=click,httpserver-click_plugin
```

### Other Components and Frameworks

Other components (e.g., VirtIO drivers, Python 2.7 framework) are compiled as OSv applications and modules. Thus, they are compiled at the same time as the platform's core operational system.

Finally, a "qcow2" image will be generated at build/last/usr.img.
Execute "scripts/gen-vbox-ova.sh" to generate an "ova".
