# MSCCL

Microsoft Collective Communication Library (MSCCL) is a platform to execute custom collective communication algorithms for multiple accelerators supported by Microsoft Azure. The research prototype of this project is [microsoft/msccl](https://github.com/microsoft/msccl).

## Introduction

MSCCL vision is to provide a unified, efficient, and scalable framework for executing collective communication algorithms across multiple accelerators. To achieve this, MSCCL has multiple components:

- [MSCCL toolkit](https://github.com/microsoft/msccl-tools): Inter-connection among accelerators have different latencies and bandwidths. Therefore, a generic collective communication algorithm does not necessarily well for all topologies and buffer sizes. In order to provide the flexibility, we provide the MSCCL toolkit, which allows a user to write a hyper-optimized collective communication algorithm for a given topology and a buffer size. MSCCL toolkit contains a high-level DSL (MSCCLang) and a compiler which generate an IR for the MSCCL executor([msccl-executor-nccl](https://github.com/Azure/msccl-executor-nccl)) to run on the backend. [Example](#Example) provides some instances on how MSCCL toolkit with the runtime works. Please refer to [MSCCL toolkit](https://github.com/microsoft/msccl-tools) for more information.

- [MSCCL scheduler](https://github.com/Azure/msccl-scheduler): MSCCL scheduler provides an example design and implementation of how to select optimal MSCCL algorithms for MSCCL executors.

- MSCCL executor([msccl-executor-nccl](https://github.com/Azure/msccl-executor-nccl)): msccl-executor-nccl is an inter-accelerator communication framework that is built on top of [NCCL](https://github.com/nvidia/nccl) and uses its building blocks to execute custom-written collective communication algorithms.

- MSCCL test toolkit([msccl-tests-nccl](https://github.com/Azure/msccl-tests-nccl)): These tests check both the performance and the correctness of MSCCL operations.

## Performance
For reference, FP16 All-Gather algorithms were tested and compared on ND H100 v5 VM, using msccl-tests-nccl.

<table>
  <tr>
    <th colspan="4">All-Gather Latency (us)</th>
  </tr>
  <tr>
    <th>Message  Size</th>
    <th>NCCL</th>
    <th>MSCCL</th>
    <th>MSCCL Speedup</th>
  </tr>
  <tr>
    <td>1KB</td>
    <td>9.54</td>
    <td>5.65</td>
    <td>1.69x</td>
  </tr>
  <tr>
    <td>2KB</td>
    <td>9.8</td>
    <td>5.7</td>
    <td>1.72x</td>
  </tr>
  <tr>
    <td>4KB</td>
    <td>9.78</td>
    <td>5.43</td>
    <td>1.80x</td>
  </tr>
  <tr>
    <td>8KB</td>
    <td>9.78</td>
    <td>5.47</td>
    <td>1.81x</td>
  </tr>
  <tr>
    <td>16KB</td>
    <td>10.29</td>
    <td>5.53</td>
    <td>1.86x</td>
  </tr>
  <tr>
    <td>32KB</td>
    <td>12.49</td>
    <td>5.75</td>
    <td>2.17x</td>
  </tr>
  <tr>
    <td>64KB</td>
    <td>12.87</td>
    <td>5.95</td>
    <td>2.16x</td>
  </tr>
  <tr>
    <td>128KB</td>
    <td>13.16</td>
    <td>6.38</td>
    <td>2.06x</td>
  </tr>
  <tr>
    <td>256KB</td>
    <td>13.23</td>
    <td>7.26</td>
    <td>1.82x</td>
  </tr>
  <tr>
    <td>512KB</td>
    <td>13.39</td>
    <td>8.71</td>
    <td>1.54x</td>
  </tr>
  <tr>
    <td>1MB</td>
    <td>18.33</td>
    <td>12.3</td>
    <td>1.49x</td>
  </tr>
  <tr>
    <td>2MB</td>
    <td>23.18</td>
    <td>17.75</td>
    <td>1.31x</td>
  </tr>
  <tr>
    <td>4MB</td>
    <td>33.66</td>
    <td>23.37</td>
    <td>1.44x</td>
  </tr>
  <tr>
    <td>8MB</td>
    <td>44.7</td>
    <td>38.54</td>
    <td>1.16x</td>
  </tr>
  <tr>
    <td>16MB</td>
    <td>67.19</td>
    <td>67.16</td>
    <td>1.00x</td>
  </tr>
  <tr>
    <td>32MB</td>
    <td>104.7</td>
    <td>98.4</td>
    <td>1.06x</td>
  </tr>
  <tr>
    <td>64MB</td>
    <td>192.4</td>
    <td>181.9</td>
    <td>1.06x</td>
  </tr>
  <tr>
    <td>128MB</td>
    <td>368.3</td>
    <td>348.4</td>
    <td>1.06x</td>
  </tr>
  <tr>
    <td>256MB</td>
    <td>699.5</td>
    <td>680.7</td>
    <td>1.03x</td>
  </tr>
  <tr>
    <td>512MB</td>
    <td>1358.6</td>
    <td>1339.3</td>
    <td>1.01x</td>
  </tr>
  <tr>
    <td>1GB</td>
    <td>2663.8</td>
    <td>2633</td>
    <td>1.01x</td>
  </tr>
</table>
 
## Example

In order to use MSCCL, you may follow these steps to use two different MSCCL algorithms for AllReduce on Azure NDv4 which has 8xA100 GPUs:

####1. Download the source code of msccl and related submodules

```sh
$ git clone https://github.com/Azure/msccl.git --recurse-submodules
```

####2. Below is the steps to install MSCCL executor:

```sh
$ git clone https://github.com/Azure/msccl.git --recurse-submodules
$ cd msccl/executor/msccl-executor-nccl
$ make -j src.build
$ cd ../
$ cd ../
```

####3. Below is the steps to install msccl-tests-nccl for performance evaluation:

```sh
$ cd tests/msccl-tests-nccl/
$ make MPI=1 NCCL_HOME=$HOME/msccl/executor/msccl-executor-nccl/build/ -j
$ cd ../
$ cd ../
```

####4. Apply the msccl algo when using msccl external scheduler 
- for ndv4, we already have algo optimized, you can use msccl scheduler to apply this algo directly to the executor, below is the steps to apply the scheduler
```sh
$ sudo apt-get install libcurl4-openssl-dev nlohmann-json3-dev
$ cd scheduler/msccl-scheduler

for nccl:
$ CXX=/path/to/nvcc BIN_HOME=/path/to/nccl/binary SRC_HOME=/path/to/nccl/source make
for rccl:
$ CXX=/path/to/nvcc BIN_HOME=/path/to/nccl/binary SRC_HOME=/path/to/nccl/source make PLATFORM=RCCL

$ make install 
```    

- for customize the msccl algo for your system, you can install [MSCCL toolkit](https://github.com/microsoft/msccl-tools) to compile a few custom algorithms:

```sh
$ git clone https://github.com/microsoft/msccl-tools.git
$ cd msccl-tools/
$ pip install .
$ cd ../
$ python msccl-tools/examples/mscclang/allreduce_a100_allpairs.py --protocol=LL 8 2 > test.xml
$ cd ../
```

The compiler's generated code is an XML file (`test.xml`) that is fed to MSCCL runtime. To evaluate its performance, copy the `test.xml` to the msccl/exector/msccl-executor-nccl/build/lib/msccl-algorithms/ and execute the following command line on an Azure NDv4 node or any 8xA100 system:

####5. Below is the command to run test using msccl-executor-nccl
```sh
$ mpirun -np 8 -x LD_LIBRARY_PATH=msccl/exector/msccl-executor-nccl/build/lib/:$LD_LIBRARY_PATH -x NCCL_DEBUG=INFO -x NCCL_DEBUG_SUBSYS=INIT,ENV tests/msccl-tests-nccl/build/all_reduce_perf -b 128 -e 32MB -f 2 -g 1 -c 1 -n 100 -w 100 -G 100 -z 0
```
  
####6. If everything is installed correctly, you should see the following output in log:

```sh
[0] NCCL INFO Connected 1 MSCCL algorithms
```

You may evaluate the performance of `test.xml` by comparing in-place (the new algorithm) vs out-of-place (default ring algorithm) and it should up-to 2-3x faster on 8xA100 NVLink-interconnected GPUs. [MSCCL toolkit](https://github.com/microsoft/msccl-tools) has a rich set of algorithms for different Azure SKUs and collective operations with significant speedups over vanilla NCCL.

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
