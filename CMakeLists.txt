cmake_minimum_required(VERSION 3.13)
project(better_containers
	VERSION 0.1.0
	LANGUAGES CXX
)

list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/tools/cmake-scripts/cmake")
include(scripts)

# Package manager
conan_init(cmake)

# Tooling
#enable_clang_tidy()
#enable_iwyu()
set_warnings(TREAT_AS_ERRORS)

# Project ==============================================================================================================
set(target_name ${PROJECT_NAME})
set(namespace ${PROJECT_NAME})

# Dependencies
# find_package(...)

# Sources
file(GLOB_RECURSE sources RELATIVE "${CMAKE_CURRENT_SOURCE_DIR}/src" CONFIGURE_DEPENDS
	"src/*.hpp"
	"src/*.cpp"
)

# Target
add_library(${target_name} INTERFACE)
foreach(source ${sources})
	target_sources(${target_name} INTERFACE $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src/${source}>)
	target_sources(${target_name} INTERFACE $<INSTALL_INTERFACE:$<INSTALL_PREFIX>/include/${source}>)
endforeach()
target_compile_features(${target_name} INTERFACE cxx_std_17)
target_include_directories(${target_name}
	INTERFACE
		$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src>
		$<INSTALL_INTERFACE:include>
)
add_library(${namespace}::${target_name} ALIAS ${target_name})

# Installation
install_targets(${target_name})
install(DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/src"
	DESTINATION include
	COMPONENT Devel
)

# Generate configuration files and install the package
configure_and_install(${CMAKE_SCRIPTS_ROOT}/simple-config.cmake.in ${namespace} SameMajorVersion)

# Examples =============================================================================================================
add_executable(example)
target_sources(example PRIVATE "examples/example1.cpp")
target_link_libraries(example PRIVATE ${namespace}::${target_name})

# Tests ================================================================================================================
enable_testing()
# check if conan installed a test framework
#conan_check_testing(gtest)

# unit tests
file(GLOB_RECURSE sources CONFIGURE_DEPENDS
	"tests/*.hpp"
	"tests/*.cpp"
)

add_executable(tests)
target_sources(tests PRIVATE ${sources})

target_link_libraries(tests
    PRIVATE
		${namespace}::${target_name}
        CONAN_PKG::gtest
)

include(GoogleTest)
gtest_discover_tests(tests)
