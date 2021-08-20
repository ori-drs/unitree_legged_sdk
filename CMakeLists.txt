cmake_minimum_required(VERSION 3.0.2)
project(unitree_legged_sdk)

# Compile settings
add_compile_options(-std=c++11)
set(CMAKE_BUILD_TYPE "RelWithDebInfo")
set(CMAKE_CXX_FLAGS "-O3")

# LCM
find_package(lcm REQUIRED)
include(${LCM_USE_FILE})
get_target_property(lcm_INCLUDE_DIRS lcm INTERFACE_INCLUDE_DIRECTORIES)

# pthreads
find_package(Threads REQUIRED)


# Create interface target for binary library
set(ARCHITECTURE amd64) # TODO: Auto-detect
add_library(unitree_legged_sdk SHARED IMPORTED GLOBAL)
set_target_properties(unitree_legged_sdk PROPERTIES
  IMPORTED_LOCATION ${CMAKE_CURRENT_SOURCE_DIR}/lib/libunitree_legged_sdk_${ARCHITECTURE}.so
  INTERFACE_INCLUDE_DIRECTORIES ${CMAKE_CURRENT_SOURCE_DIR}/include
)
add_library(${PROJECT_NAME}::${PROJECT_NAME} ALIAS ${PROJECT_NAME})

set(EXTRA_LIBS unitree_legged_sdk::unitree_legged_sdk Threads::Threads lcm)

# Configure build:
include_directories(include ${lcm_INCLUDE_DIRS})

# one pc one process
add_executable(example_position examples/example_position.cpp)
target_link_libraries(example_position ${EXTRA_LIBS})

add_executable(example_velocity examples/example_velocity.cpp)
target_link_libraries(example_velocity ${EXTRA_LIBS})

add_executable(example_torque examples/example_torque.cpp)
target_link_libraries(example_torque ${EXTRA_LIBS})

add_executable(example_walk examples/example_walk.cpp)
target_link_libraries(example_walk ${EXTRA_LIBS})

add_executable(example_wirelessHandle examples/example_wirelessHandle.cpp)
target_link_libraries(example_wirelessHandle ${EXTRA_LIBS})

# one pc multi process
add_executable(lcm_server_high examples/lcm_server_high.cpp)
target_link_libraries(lcm_server_high ${EXTRA_LIBS})

add_executable(lcm_server_low examples/lcm_server_low.cpp)
target_link_libraries(lcm_server_low ${EXTRA_LIBS})

# multi pc
add_executable(udp_send_test examples/multi_pc_udp_send.cpp)
target_link_libraries(udp_send_test ${EXTRA_LIBS})

add_executable(udp_recv_test examples/multi_pc_udp_recv.cpp)
target_link_libraries(udp_recv_test ${EXTRA_LIBS})

# lcm server
add_executable(lcm_server examples/lcm_server.cpp)
target_link_libraries(lcm_server ${EXTRA_LIBS})

# NB: We are not installing the examples...

# Installation
include(GNUInstallDirs)

# install library
install(
  FILES
    lib/libunitree_legged_sdk_amd64.so
    lib/libunitree_legged_sdk_arm32.so
    lib/libunitree_legged_sdk_arm64.so
  DESTINATION ${CMAKE_INSTALL_LIBDIR}
)
install(DIRECTORY include/${PROJECT_NAME} DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})

# export
set(INCLUDE_INSTALL_DIR ${CMAKE_INSTALL_INCLUDEDIR})
set(LIB_INSTALL_DIR ${CMAKE_INSTALL_LIBDIR})

include(CMakePackageConfigHelpers)
configure_package_config_file(cmake/unitree_legged_sdkConfig.cmake.in
  ${CMAKE_CURRENT_BINARY_DIR}/unitree_legged_sdkConfig.cmake
  INSTALL_DESTINATION ${LIB_INSTALL_DIR}/unitree_legged_sdk/cmake
  PATH_VARS INCLUDE_INSTALL_DIR LIB_INSTALL_DIR)
write_basic_package_version_file(
  ${CMAKE_CURRENT_BINARY_DIR}/unitree_legged_sdkConfigVersion.cmake
  VERSION 0.0.0
  COMPATIBILITY SameMajorVersion)
install(FILES ${CMAKE_CURRENT_BINARY_DIR}/unitree_legged_sdkConfig.cmake
              ${CMAKE_CURRENT_BINARY_DIR}/unitree_legged_sdkConfigVersion.cmake
        DESTINATION ${LIB_INSTALL_DIR}/unitree_legged_sdk/cmake )
