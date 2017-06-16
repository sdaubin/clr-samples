cmake_minimum_required(VERSION 3.8)
project(CorProfiler)
if(${APPLE})
  set(DefaultBuildOS "OSX")
  add_definitions(-DPLATFORM_UNIX)
  add_definitions(-DPAL_STDCPP_COMPAT)
elseif(${WIN32})
  set(DefaultBuildOS "Windows")
else()
  set(DefaultBuildOS "Linux")
  add_definitions(-DPLATFORM_UNIX)
  add_definitions(-DPAL_STDCPP_COMPAT)
endif()
if(${CMAKE_SIZEOF_VOID_P} EQUAL "8")
  set(DefaultBuildArch "x64")
  add_definitions(-DBIT64)
else()
  set(DefaultBuildArch "x86")
endif()
set(CORECLR_PATH "/root/git/coreclr" CACHE PATH "Path to coreclr repository.")
set(BuildOS ${DefaultBuildOS} CACHE STRING "Operating system for build.")
set(BuildArch ${DefaultBuildArch} CACHE STRING "Build architecture (x64, x86)")
set(BuildType "Debug" CACHE STRING "Build type (Release, Debug)")
set(Output "CorProfiler" CACHE STRING "Name of output file")
set(CoreCLRLinkPath "${CORECLR_PATH}/bin/Product/${BuildOS}.${BuildArch}.${BuildType}")
# TODO: Link to the coreclr cmake project as a dep.
if(${WIN32})
  add_compile_options(
    -fPIC
    -fms-extensions
    -std=c++11
  )
else()
  add_compile_options(
    --no-undefined
    -Wno-invalid-noreturn
    -fPIC
    -fms-extensions
    -std=c++11
    -w
  )
endif()
file(GLOB SOURCES
  "*.h"
  "*.cpp"
)

add_library(${Output} SHARED ${SOURCES})
include_directories(
  "${CORECLR_PATH}/src/pal/inc/rt"
  "${CORECLR_PATH}/src/pal/prebuilt/inc"
  "${CORECLR_PATH}/src/pal/inc"
  "${CORECLR_PATH}/src/inc"
  "${CoreCLRLinkPath}/inc"
  "${CoreCLRLinkPath}/lib"
)
if(${APPLE})
  set(LinkedLibs "${CoreCLRLinkPath}/libmscordaccore.dylib;${CoreCLRLinkPath}/libcoreclr.dylib;${CoreCLRLinkPath}/libsuperpmi-shim-simple.dylib")
else()
  set(LinkedLibs "${CoreCLRLinkPath}/libmscordaccore.so;${CoreCLRLinkPath}/libcoreclr.so;${CoreCLRLinkPath}/libsuperpmi-shim-simple.so")
endif()
link_directories(${CoreCLRLinkPath})
target_link_libraries(${Output} ${LinkedLibs})
