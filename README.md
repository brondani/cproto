# [PoC] cproto
csolution IPC support over Protocol Buffers


## Prerequisites

Required by all targets:
- C++ compiler as required by [devtools](https://github.com/Open-CMSIS-Pack/devtools?tab=readme-ov-file#prerequisites)

Additionally required for TypeScript support:
- Protoc Gen Typescript plugin [protoc-gen-ts](https://www.npmjs.com/package/protoc-gen-ts)
```
npm install -g protoc-gen-ts
```

## Generate configuration files

- CMake configuration as described in [devtools](https://github.com/Open-CMSIS-Pack/devtools?tab=readme-ov-file#generate-configuration-files)

## CMake Targets

- For C++ support library generation run in the build directory:
```
cmake --build . --target cproto-lib
```
- For TypeScript support generation run in the build directory:
```
cmake --build . --target cproto-ts
```