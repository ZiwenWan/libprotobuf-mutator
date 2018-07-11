# libprotobuf-mutator

## Overview
The repository is forked from [google/libprotobuf-mutator](https://github.com/google/libprotobuf-mutator) 
libprotobuf-mutator is a library to randomly mutate [protobuffers](https://github.com/google/protobuf) messages. 
It could be used together with guided fuzzing engine: [libFuzzer](http://libfuzzer.info). 
Customized for fuzzing test against Baidu [Apollo](https://github.com/ApolloAuto/apollo) self-driving functionalities
## Prerequisite

To use libfuzzer, install clang-6.0 in the Apollo docker. 

```
sudo apt-add-repository "deb http://apt.llvm.org/trusty/ llvm-toolchain-trusty-6.0 main"
wget -O - http://apt.llvm.org/llvm-snapshot.gpg.key | sudo apt-key add -
sudo apt-get update
sudo apt-get install clang-6.0 lldb-6.0 lld-6.0
```

After merging the pull request, you should have the following **requirements** ready
* `/apollo/tools/fuzz-compiler
* Several fuzz driver derived from unit test under `/apollo/modules/controller/` and `/apollo/modules/integration_test/` with the name contains `_fuzz`
* Updated `WORKSPACE.in` by adding a new git repository for a modified version of libprotbuf-mutator
* BUILD file for the third-party library located in `/apollo/third_party/libprotobuf-mutator.BUILD`
```
mkdir build
cd build
cmake .. -GNinja -DCMAKE_C_COMPILER=clang -DCMAKE_CXX_COMPILER=clang++ -DCMAKE_BUILD_TYPE=Debug
ninja check
```
Clang is only needed for libFuzzer integration. <BR>
By default, the system-installed version of
[protobuf](https://github.com/google/protobuf) is used.  However, on some
systems, the system version is too old.  You can pass
`LIB_PROTO_MUTATOR_DOWNLOAD_PROTOBUF=ON` to cmake to automatically download and
build a working version of protobuf.

## Usage

To use libprotobuf-mutator simply include
[mutator.h](/src/mutator.h) and
[mutator.cc](/src/mutator.cc) into your build files.

The `ProtobufMutator` class implements mutations of the protobuf
tree structure and mutations of individual fields.
The field mutation logic is very basic --
for better results you should override the `ProtobufMutator::Mutate*`
methods with more sophisticated logic, e.g.
using [libFuzzer](http://libfuzzer.info)'s mutators.

To apply one mutation to a protobuf object do the following:

```
class MyProtobufMutator : public protobuf_mutator::Mutator {
 public:
  MyProtobufMutator(uint32_t seed) : protobuf_mutator::Mutator(seed) {}
  // Optionally redefine the Mutate* methods to perform more sophisticated mutations.
}
void Mutate(MyMessage* message) {
  MyProtobufMutator mutator(my_random_seed);
  mutator.Mutate(message, 200);
}
```

See also the `ProtobufMutatorMessagesTest.UsageExample` test from
[mutator_test.cc](/src/mutator_test.cc).

## Integrating with libFuzzer
LibFuzzerProtobufMutator can help to integrate with libFuzzer. For example 

```
#include "src/libfuzzer/libfuzzer_macro.h"

DEFINE_PROTO_FUZZER(const MyMessageType& input) {
  // Code which needs to be fuzzed.
  ConsumeMyMessageType(input);
}
```

Please see [libfuzzer_example.cc](/examples/libfuzzer/libfuzzer_example.cc) as an example.

## UTF-8 strings
"proto2" and "proto3" handle invalid UTF-8 strings differently. In both cases
string should be UTF-8, however only "proto3" enforces that. So if fuzzer is
applied to "proto2" type libprotobuf-mutator will generate any strings including
invalid UTF-8. If it's a "proto3" message type, only valid UTF-8 will be used.
