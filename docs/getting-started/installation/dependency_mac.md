# Mac

The `epic` project is compatible with MacOS Mojave. In order to compile and run `epic`, the following depencencies are needed.

## LLVM

```bash
brew install llvm
```

You may need to set the following path in `~/.zshrc` or `~/.bash_profile`

```bash
export PATH="/usr/local/opt/llvm/bin:$PATH"
export LDFLAGS="-L/usr/local/opt/llvm/lib"
export CPPFLAGS="-I/usr/local/opt/llvm/include"
export LDFLAGS="-L/usr/local/opt/llvm/lib -Wl,-rpath,/usr/local/opt/llvm/lib"
```

## Cmake

```bash
brew install cmake
```

## RocksDB

It is quite simple on MacOS, though it's really complicated on Linux.

```bash
brew install rocksdb
```

## libsecp256k1

This package is installed by autoconfig so the following tools are required and can be simply installed using `brew`.

```bash
brew install automake autoconf libtool
```

`libsecp256k1` alse requires `gmp`, which can be installed by `brew`

```bash
brew install gmp
```

Then we can build it from source

```bash
git clone https://github.com/bitcoin-core/secp256k1.git
cd secp256k1
./autogen.sh
./configure --enable-module-recovery=yes
make
./tests
sudo make install
```

## Libevent

Despite it can be installed by `brew`, `epic` requires `libevent-release-2.1.8` be be manually installed from source.

For this, you need to first install `openssl` by

```bash
brew install openssl
```

and set the following in `~/.zshrc` or `~/.bash_profile`

```bash
export OPENSSL_ROOT_DIR="/usr/local/opt/openssl"
export OPENSSL_LIBRARIES="/usr/local/opt/openssl/lib"
```

Then, download `libevent-release-2.1.8` and build from source as follows

```bash
wget https://github.com/libevent/libevent/archive/release-2.1.8-stable.zip
unzip -q release-2.1.8-stable.zip && cd libevent-release-2.1.8-stable
mkdir build && cd build
cmake ..
make
sudo make install
make verify  # (optional)
```

## GoogleTest

Despite it can be installed by `brew`, `epic` requires `release-1.8.1` be be manually installed from source.

```bash
wget https://github.com/google/googletest/archive/release-1.8.1.zip
unzip -q release-1.8.1.zip && cd googletest-release-1.8.1/
mkdir build && cd build
cmake .. 
make && sudo make install
```

## gRPC

Although the latest release 1.22 can be installed via `brew`, `epic` need to install it from source. First need to install dependency

```bash
brew install c-ares
```

Then some C header files are required. This step is complicated since MacOS Mojave removed some system headers. First, you may check whether you already have the required C headers by

```bash
ls /Library/Developer/CommandLineTools/Packages
```

If you have something like `macOS_SDK_headers_for_macOS_10.14.pkg` in this folder, then you can skip the next step. Otherwise, please go to [https://developer.apple.com/download/more/](https://developer.apple.com/download/more/) and download `Command Line Tools (macOS 10.14) for Xcode 10.2.1.dmg` and install. After this, you will find `/Library/Developer/CommandLineTools/Packages/macOS_SDK_headers_for_macOS_10.14.pkg`. After install this package, you will have the required system header files. Now, you can build from source.

```bash
git clone -b $(curl -L https://grpc.io/release) https://github.com/grpc/grpc
cd grpc
git submodule update --init
make
sudo make install
```

## Protocol Buffers

```bash
brew install protobuf
```

