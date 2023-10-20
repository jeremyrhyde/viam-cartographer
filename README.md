# viam-cartographer

## User documentation

For user documentation, see [Cartographer Modular Resource](https://docs.viam.com/services/slam/cartographer/).

## (In)stability Notice
> **Warning**
> This is an experimental feature. Stability is not guaranteed. Breaking changes are likely to occur, and occur often.

## Overview

This repo wraps [Cartographer](https://github.com/cartographer-project/cartographer) as a [modular resource](https://docs.viam.com/program/extend/modular-resources/) so it is easily usable with the rest of Viam's ecosystem. Cartographer is a system that provides real-time Simultaneous Localization
And Mapping ([SLAM](https://en.wikipedia.org/wiki/Simultaneous_localization_and_mapping)) in 2D and 3D across multiple platforms and sensor configurations.

## Getting started

### Stable AppImages

Install viam-cartographer:

* Linux aarch64:
    ```bash
    sudo curl -o /usr/local/bin/cartographer-module https://storage.googleapis.com/packages.viam.com/apps/slam-servers/cartographer-module-stable-aarch64.AppImage
    sudo chmod a+rx /usr/local/bin/cartographer-module
    ```
 * Linux x86_64:
    ```bash
    sudo curl -o /usr/local/bin/cartographer-module https://storage.googleapis.com/packages.viam.com/apps/slam-servers/cartographer-module-stable-x86_64.AppImage
    sudo chmod a+rx /usr/local/bin/cartographer-module
    ```
* MacOS/Linux
    ```bash
    brew tap viamrobotics/brews && brew install cartographer-module
    ```
Note that AppImages require FUSE to run. See [documentation](https://docs.viam.com/appendix/troubleshooting/#appimages-require-fuse-to-run).

## Development

You can either install the latest AppImages for testing, or build the code from source.

### Latest AppImages

You can install the latest AppImages using:
* Linux aarch64:
    ```bash
    sudo curl -o /usr/local/bin/cartographer-module https://storage.googleapis.com/packages.viam.com/apps/slam-servers/cartographer-module-latest-aarch64.AppImage
    sudo chmod a+rx /usr/local/bin/cartographer-module
    ```
 * Linux x86_64:
    ```bash
    sudo curl -o /usr/local/bin/cartographer-module https://storage.googleapis.com/packages.viam.com/apps/slam-servers/cartographer-module-latest-x86_64.AppImage
    sudo chmod a+rx /usr/local/bin/cartographer-module
    ```
Note that AppImages require FUSE to run. See [documentation](https://docs.viam.com/appendix/troubleshooting/#appimages-require-fuse-to-run).

### Build from source

#### Download
```bash
git clone --recurse-submodules https://github.com/viamrobotics/viam-cartographer.git
```

If you happened to use `git clone` only, you won't see the `cartographer` folder and will need to fetch it:

`git submodule update --init`

### (Optional) Using Canon Images

If desired, Viam's canon tool can be used to create a docker container to build `arm64` or `amd64` binaries of the SLAM server. The canon tool can be installed by running the following command: 

```bash
go install github.com/viamrobotics/canon@latest
```

And then by running one of the following commands in the viam-cartographer repository to create the container:

```bash
canon -arch arm64
```

```bash
canon -arch amd64
```

These containers are set to persist between sessions via the `persistent` parameter in the `.canon.yaml` file located in the root of viam-cartographer. More details regarding the use of Viam's canon tool can be found [here](https://github.com/viamrobotics/canon).

#### Setup, build, and run the binary

```bash
# Install dependencies using brew (or apt-get)
make setup
# Build
make build
# Install lua files and binaries to /usr/local
make install
# Run the binary
cartographer-module
```

### Linting

```bash
make lint-setup
make lint
```
### Testing

```bash
make test
```
### Working with submodules

#### Commit and push
1. Commit and push changes in the `cartographer` submodule first.
2. Commit and push changes in the `viam-cartographer` library last.

Or, alternatively:
1. Commit changes in the `cartographer` submodule
2. Commit changes in the main repo
3. Push all changes by running `git push --recurse-submodules=on-demand`

#### Changing branches in a submodule
When changing branches in a submodule, update `.gitmodules`, e.g., changing to a branch called `kk/fix-install`:

```bash
...
[submodule "viam-cartographer/cartographer"]
        path = viam-cartographer/cartographer
        url = git@github.com:kkufieta/cartographer.git
        branch=kk/fix-install
```

Commit & push the changes.

When pulling those changes, run the following:
```bash
git pull
git submodule update --init --recursive
```

### Troubleshooting Build Failures

#### Protobuf errors

When building on `MacOS` and seeing errors related to `protobuf` such as

```
Undefined symbols for architecture arm64:
  "google::protobuf::internal::InternalMetadata::~InternalMetadata()", referenced from:
      google::protobuf::MessageLite::~MessageLite() in libviam-cartographer.a(common.pb.cc.o)
      google::protobuf::MessageLite::~MessageLite() in libviam-cartographer.a(slam.pb.cc.o)
      google::protobuf::MessageLite::~MessageLite() in libviam-cartographer.a(http.pb.cc.o)
```

or

```
/tmp/cartographer-module-20230421-29152-arkug3/viam-cartographer/../grpc/cpp/gen/google/api/http.pb.h:12:2: error: This file was generated by a newer version of protoc which is
/tmp/cartographer-module-20230421-29152-arkug3/viam-cartographer/../grpc/cpp/gen/google/api/http.pb.h:13:2: error: incompatible with your Protocol Buffer headers. Please update
/tmp/cartographer-module-20230421-29152-arkug3/viam-cartographer/../grpc/cpp/gen/google/api/http.pb.h:14:2: error: your headers.
```

Then it possible your `brew` installation is using the incorrect version of `protobuf`. To check if you are seeing this issue run

```
protoc --version
```

If your version is `3.20.3` then you may be having this error. To fix take the following steps

1. unlink protobuf@3

```
brew unlink protobuf@3
```

2. link normal protobuf

```
brew unlink protobuf && brew link protobuf
```

3. echo your path & confirm you are not manually pathing to protobuf@3. If you see protobuf@3 in your path, track down where that is being added to your path & remove it (try .zshrc / .bashrc)

```
echo $PATH
```

4. Create a new terminal window to get a clean shell state

5. Confirm you have a new version installed. The minimum version you should see on your Mac is `3.21.12`

```
protoc --version
```

#### CMake error about CMakeCache.txt

If you see an error during `make build` like:

```
$ make build
Submodule found successfully
grep -q viam-patched viam-cartographer/cartographer/CMakeLists.txt || \
	(cd viam-cartographer/cartographer && git checkout . && git apply ../cartographer_patches/carto.patch)
buf generate --template ./buf/buf.gen.yaml buf.build/viamrobotics/api
buf generate --template ./buf/buf.grpc.gen.yaml buf.build/viamrobotics/api
buf generate --template ./buf/buf.gen.yaml buf.build/googleapis/googleapis
# Touch this file so that we don't regenerate the buf compiled C++ files
# every time we build
cd viam-cartographer && cmake -Bbuild -G Ninja -DCMAKE_PREFIX_PATH=/opt/homebrew -DQt5_DIR=/opt/homebrew/opt/qt@5/lib/cmake/Qt5 && cmake --build build
CMake Error: The current CMakeCache.txt directory /Users/eshamaharishi/Documents/code/viam-cartographer/viam-cartographer/build/CMakeCache.txt is different than the directory /host/viam-cartographer/build where CMakeCache.txt was created. This may result in binaries being created in the wrong place. If you are not sure, reedit the CMakeCache.txt
CMake Error: The source "/Users/eshamaharishi/Documents/code/viam-cartographer/viam-cartographer/CMakeLists.txt" does not match the source "/host/viam-cartographer/CMakeLists.txt" used to generate cache.  Re-run cmake with a different source directory.
```

then you probably previously built for a different architecture. You should run `make clean` and then run `make build` again.

## License
Copyright 2023 Viam Inc.

Apache 2.0 - See [LICENSE](https://github.com/viamrobotics/slam/blob/main/LICENSE) file
