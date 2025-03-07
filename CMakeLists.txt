cmake_minimum_required(VERSION 3.22.0)
project(cproto)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED True)

if(MSVC)
# disable C4141: inline used more than once
  add_compile_definitions(PROTOBUF_NO_INLINE)
# disable C4996 warning: of not using *_s functions
  add_compile_definitions(_CRT_SECURE_NO_WARNINGS)
  if("${CMAKE_GENERATOR_PLATFORM}" STREQUAL "x64")
    set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} /ZI")
  endif()
  set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} /Gy /FC /utf-8")
  message("CMAKE_CXX_FLAGS_DEBUG: ${CMAKE_CXX_FLAGS_DEBUG}")
  set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} /utf-8")
# disable optimization for RelWithDebug
  set(CMAKE_CXX_FLAGS_RELWITHDEBINFO "${CMAKE_CXX_FLAGS_RELWITHDEBINFO} /Zi /Od /utf-8")
  add_link_options("$<$<CONFIG:Debug>:/SAFESEH:NO>")
  set(CMAKE_CXX_FLAGS_DEFAULT "/w /utf-8")
endif(MSVC)

# protobuf
include(FetchContent)
FetchContent_Declare(
  protobuf
  GIT_REPOSITORY https://github.com/protocolbuffers/protobuf.git
  GIT_TAG        v29.3
  OVERRIDE_FIND_PACKAGE
  EXCLUDE_FROM_ALL
)
set(protobuf_BUILD_SHARED_LIBS OFF CACHE INTERNAL "" FORCE)
set(protobuf_MSVC_STATIC_RUNTIME ON CACHE INTERNAL "" FORCE)
set(protobuf_WITH_ZLIB OFF CACHE INTERNAL "" FORCE)
set(protobuf_BUILD_TESTS OFF CACHE INTERNAL "" FORCE)
set(protobuf_BUILD_LIBUPB OFF CACHE INTERNAL "" FORCE)
FetchContent_MakeAvailable(protobuf)

# input/outut locations
set(PROTO_DIR ${CMAKE_CURRENT_SOURCE_DIR}/proto)
set(PROTO_FILES ${PROTO_DIR}/cproto.proto)
set(CXX_DIR ${CMAKE_CURRENT_SOURCE_DIR}/cxx)
set(CXX_SRC ${CXX_DIR}/src)
set(CXX_SRC_FILES ${CXX_SRC}/cproto.pb.cc ${CXX_SRC}/cproto.pb.h)
set(TS_DIR ${CMAKE_CURRENT_SOURCE_DIR}/ts)
set(TS_SRC ${TS_DIR}/src)
set(TS_SRC_FILES ${TS_SRC}/cproto.ts)

# generate c++ support
add_custom_target(cproto-cpp
  DEPENDS ${CXX_SRC_FILES}
  SOURCES ${PROTO_FILES}
)
add_custom_command(
  OUTPUT ${CXX_SRC_FILES}
  DEPENDS ${PROTO_FILES} protoc
  COMMAND protoc -I=${PROTO_DIR} --cpp_out=${CXX_SRC} ${PROTO_FILES}
)
add_library(cproto-lib STATIC ${CXX_SRC_FILES})
target_include_directories(cproto-lib PUBLIC ${CXX_SRC})
target_link_libraries(cproto-lib libprotobuf)

# generate typescript support
add_custom_target(cproto-ts
  DEPENDS ${TS_SRC_FILES}
  SOURCES ${PROTO_FILES}
)
add_custom_command(
  OUTPUT ${TS_SRC_FILES}
  DEPENDS ${PROTO_FILES} protoc
  COMMAND ${CMAKE_COMMAND} -E rm -f ${TS_SRC_FILES}
  COMMAND protoc -I=${PROTO_DIR} --ts_out=${TS_SRC} ${PROTO_FILES}
)
