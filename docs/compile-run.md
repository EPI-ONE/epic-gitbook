# Joining the network

We provide instructions for joining the network either by running a full node or just use command lines tool to get some information, e.g. inquire a block or transaction, or send transactions.

## System and Network Requirement

If you only intend to use command line tools, then a laptop with wifi connect suffice. If you want to run a full node, here is a basic estimate on the system and network requirement. 

### Computing

Compile and run the code does not require intensive computing. A normal computer with relative recent CPU and gigabytes of memory will do. However, if you intend to create blocks, a.k.a "mining", GPU with 4 Gb memory seems necessary. In that case, you may use a normal computer/server to run the node and another GPU server to mine blocks via RPC. 

### Storage

If you do not have the full data, there is no way to participate creating new blocks. Simple multiplication shows that 1000TPS, together with database storage overhead, lead to data accumulation at the rate of 15TB per year. Of curse, when the capacity is not fully utilized, this rate will be less. 

### Network

Those who used internet in the late 90's probably remember waiting for hours downloading a 4 magabyte song in MP3 format. In order to reach concensus on about 1000 Transactions per second, exchange this amount of information over the current Internet infrusturcture is a necessary condition. A node will likely receive the same block (essentially transaction container) multiple times from different neighbours due to the peer-to-peer network protocol. See our network protocol for some improvement in order to recude such an effect. Still, an estimated 200Mb of bandwidth is required for running a node. 

## Compile

Make sure dependencies have all been successfully installed following the [instruction](./install).

```bash
git clone https://github.com/epi-one/epic/
cd epic && mkdir build && cd build
# Debug mode by default
# cmake ..
# Release mode
cmake -DCAMKE_BUILD_TYPE=Release ..
make -j
```

### GPU mining

The project is enabled with CPU mining by default, if no CUDA installation found on the system.
It compiles the CUDA code with NVCC if CUDA is found, and GPU mining is then enabled automatically.
To disable GPU mining at all, add the flag `-DEPIC_ENABLE_CUDA=OFF` to the `cmake` command before compiling the codes.

If you are experiencing the following runtime error: `GPUassert(2): out of memory <EPIC PATH>/src/miner.h 32` while you do have adequate GPU memory, set the following shell environment variable:


``` shell
$ export ASAN_OPTIONS="protect_shadow_gap=0"
```

## Run

After successful compile, you may run the test

```bash
cd ../bin
# you may run the following test
./epictest
```

You may simply run the following command to start the daemon. 

```bash
# runing the daemon
./epic --configpath /path/to/toml
# a default config.toml is provided in the project root folder
```

Essentially, your server is participating in the network by syncing data with peers, verifying received blocks/transactions, maintaining UTXO by building ledgers, responding to RPC request. If you have GPU on your server, you should be able to mine blocks from time to time depending on the how the computing power of your GPU compared to that over the whole network. You may also connect another GPU server to your daemon by running

```bash
#...
```

We also provide some command line tools to communicate with a daemon

```bash
# send rpc command to daemon
./epic-cli [OPTIONS] [COMMAND]
```



