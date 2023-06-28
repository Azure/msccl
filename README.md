# MSCCL

Microsoft Collective Communication Library (MSCCL) is a platform to execute custom collective communication algorithms for multiple accelerators supported by Microsoft Azure.

## Introduction

MSCCL vision is to provide a unified, efficient, and scalable framework for executing collective communication algorithms across multiple accelerators. To achieve this, MSCCL has multiple components:

- [MSCCL toolkit](https://github.com/microsoft/msccl-tools): Inter-connection among accelerators have different latencies and bandwidths. Therefore, a generic collective communication algorithm does not necessarily well for all topologies and buffer sizes. In order to provide the flexibility, we provide the MSCCL toolkit, which allows a user to write a hyper-optimized collective communication algorithm for a given topology and a buffer size. MSCCL toolkit contains a high-level DSL (MSCCLang) and a compiler which generate an IR for the MSCCL executor([msccl-executor-nccl](https://github.com/Azure/msccl-executor-nccl)) to run on the backend. [Example](#Example) provides some instances on how MSCCL toolkit with the runtime works. Please refer to [MSCCL toolkit](https://github.com/microsoft/msccl-tools) for more information.
 
- MSCCL executor([msccl-executor-nccl](https://github.com/Azure/msccl-executor-nccl)): msccl-executor-nccl is an inter-accelerator communication framework that is built on top of [NCCL](https://github.com/nvidia/nccl) and uses its building blocks to execute custom-written collective communication algorithms.

- MSCCL test toolkit([msccl-tests-nccl](https://github.com/Azure/msccl-tests-nccl)): These tests check both the performance and the correctness of MSCCL operations.

Please consider citing our work if you use MSCCL in your work. Also, please contact us if you have any questions or need an optimized collective communication algorithm for a specific topology.

## Example

In order to use MSCCL, you may follow these steps to use two different MSCCL algorithms for AllReduce on Azure NDv4 which has 8xA100 GPUs:

Follow below steps to download the source code of msccl and related submodules

```sh
$ git clone https://github.com/microsoft/msccl.git --recurse-submodules
```

Steps to install MSCCL executor:

```sh
$ git clone https://github.com/microsoft/msccl.git --recurse-submodules
$ cd msccl/executor/msccl-executor-nccl
$ make -j src.build
$ cd ../
$ cd ../
```

Then, follow these steps to install msccl-tests-nccl for performance evaluation:

```sh
$ cd tests/msccl-tests-nccl/
$ make MPI=1 NCCL_HOME=$HOME/msccl/executor/msccl-executor-nccl/build/ -j 
$ cd ../
$ cd ../
```

Next install [MSCCL toolkit](https://github.com/microsoft/msccl-tools) to compile a few custom algorithms:

```sh
$ git clone https://github.com/microsoft/msccl-tools.git
$ cd msccl-tools/
$ pip install .
$ cd ../
$ python msccl-tools/examples/mscclang/allreduce_a100_allpairs.py --protocol=LL 8 2 > test.xml
$ cd ../
```

The compiler's generated code is an XML file (`test.xml`) that is fed to MSCCL runtime. To evaluate its performance, copy the `test.xml` to the msccl/exector/msccl-executor-nccl/build/lib/msccl-algorithms/ and execute the following command line on an Azure NDv4 node or any 8xA100 system:

```sh
$ mpirun -np 8 -x LD_LIBRARY_PATH=msccl/exector/msccl-executor-nccl/build/lib/:$LD_LIBRARY_PATH -x NCCL_DEBUG=INFO -x NCCL_DEBUG_SUBSYS=INIT,ENV tests/msccl-tests-nccl/build/all_reduce_perf -b 128 -e 32MB -f 2 -g 1 -c 1 -n 100 -w 100 -G 100 -z 0
```

If everything is installed correctly, you should see the following output in log:

```sh
[0] NCCL INFO Connected 1 MSCCL algorithms
```

You may evaluate the performance of `test.xml` by comparing in-place (the new algorithm) vs out-of-place (default ring algorithm) and it should up-to 2-3x faster on 8xA100 NVLink-interconnected GPUs. [MSCCL toolkit](https://github.com/microsoft/msccl-tools) has a rich set of algorithms for different Azure SKUs and collective operations with significant speedups over vanilla NCCL.

## Build

To build the library:

```sh
$ cd msccl/exector/msccl-executor-nccl
$ make -j src.build
```

If CUDA is not installed in the default /usr/local/cuda path, you can define the CUDA path with :

```sh
$ make src.build CUDA_HOME=<path to cuda install>
```

MSCCL will be compiled and installed in `build/` unless `BUILDDIR` is set.

By default, MSCCL is compiled for all supported architectures. To accelerate the compilation and reduce the binary size, consider redefining `NVCC_GENCODE` (defined in `makefiles/common.mk`) to only include the architecture of the target platform :
```sh
$ make -j src.build NVCC_GENCODE="-gencode=arch=compute_80,code=sm_80"
```

## Install

To install MSCCL on the system, create a package then install it as root.

Debian/Ubuntu :
```sh
$ # Install tools to create debian packages
$ sudo apt install build-essential devscripts debhelper fakeroot
$ # Build msccl-executor-nccl deb package
$ make pkg.debian.build
$ ls build/pkg/deb/
```

RedHat/CentOS :
```sh
$ # Install tools to create rpm packages
$ sudo yum install rpm-build rpmdevtools
$ # Build msccl-executor-nccl rpm package
$ make pkg.redhat.build
$ ls build/pkg/rpm/
```

OS-agnostic tarball :
```sh
$ make pkg.txz.build
$ ls build/pkg/txz/
```

## Contributing

This project welcomes contributions and suggestions.  Most contributions require you to agree to a
Contributor License Agreement (CLA) declaring that you have the right to, and actually do, grant us
the rights to use your contribution. For details, visit [CLA](https://cla.opensource.microsoft.com).

When you submit a pull request, a CLA bot will automatically determine whether you need to provide
a CLA and decorate the PR appropriately (e.g., status check, comment). Simply follow the instructions
provided by the bot. You will only need to do this once across all repos using our CLA.

This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/).
For more information see the [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/) or
contact [opencode@microsoft.com](mailto:opencode@microsoft.com) with any additional questions or comments.

## Trademarks

This project may contain trademarks or logos for projects, products, or services. Authorized use of Microsoft 
trademarks or logos is subject to and must follow 
[Microsoft's Trademark & Brand Guidelines](https://www.microsoft.com/en-us/legal/intellectualproperty/trademarks/usage/general).
Use of Microsoft trademarks or logos in modified versions of this project must not cause confusion or imply Microsoft sponsorship.
Any use of third-party trademarks or logos are subject to those third-party's policies.
