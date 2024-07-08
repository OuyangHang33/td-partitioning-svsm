## TDP environment setup

For host Kernel, QEMU and guest image setup, please follow [INSTALLATION_GUIDE] https://github.com/intel-staging/td-partitioning-svsm/blob/svsm-tdp-patches/INSTALLATION_GUIDE.md.

vTPM requires additional patches for OVMF and TDP guest Linux Kernel. Please follow [Setup OVMF](#Setup-OVMF) and [Setup TDP guest Linux Kernel](#Setup-TDP-guest-Linux-Kernel) to build the images.

## Setup OVMF

### Prepare OVMF tree

Download the source code:

```
git clone --branch edk2-stable202402 --single-branch https://github.com/tianocore/edk2.git
cd edk2 && git checkout -b ovmf-svsm-tdp
```

The TDP OVMF patches in [Patches/OVMF](Patches/OVMF), apply the patches:
```
git am --ignore-whitespace <path-to-patches>/*\.patch
```

### Build OVMF

Please note that `-DTPM2_ENABLE` needs to be enabled for vTPM.

```
git submodule update --init --recursive
make -C BaseTools clean && make -C BaseTools
source ./edksetup.sh
build -a X64 -b DEBUG -t GCC5 -D FD_SIZE_2MB -DTPM2_ENABLE -D DEBUG_ON_SERIAL_PORT -D DEBUG_VERBOSE -p OvmfPkg/OvmfPkgX64.dsc
```

OVMF image can be found at Build/OvmfX64/DEBUG_GCC5/FV/OVMF.fd

## Setup TDP guest Linux Kernel

### Prepare Linux Kernel source code

```
git clone --branch v6.6 --single-branch https://github.com/torvalds/linux.git
cd linux/
git switch -c v6.6
```

The TDP guest Kernel patches in [Patches/Kernel](Patches/Kernel), apply the patches:

```
git am --ignore-whitespace <path-to-patches>/*\.patch
```

### Build TDP guest Kernel

Enable TDX_GUEST config:

```
scripts/config --enable CONFIG_INTEL_TDX_GUEST
```

Build Kernel image:

```
make
```

## Features included
 - vTPM CRB MMIO interface
 - vTPM CA generation with TDX remote attestation
 - vTPM Endorsement Key certificate and CA provision
 - TDP L2 guest vTPM detection through TDVMCALL
 - SVSM TPM startup and measurement (SVSM version and TDVF).

## What has been tested:
 - TPM event log/PCR replay in L2 Linux
 - Linux IMA
 - Endorsement Key certificate and CA certificate read and verify.
 - Quote verification

## Known Issues:
 - Potential `Page fault` error observed when running `tpm2_createek` in guest Linux. ( Run `make TEST_VTPM=1` and build `svsm.bin`, test createek flow in L1.)
 - TSS reports `out of memory for object contexts` when running `keylime` in guest Linux.