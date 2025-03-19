cmake_minimum_required(VERSION 3.29)

project(vcpkg-nanovdb-hello-world LANGUAGES CXX)

set_property(GLOBAL PROPERTY USE_FOLDERS ON)

find_package(OpenVDB CONFIG REQUIRED)

include(compiler_flags)

add_subdirectory(src)
