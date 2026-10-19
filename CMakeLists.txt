cmake_minimum_required(VERSION 3.16)
project(SuperJupiterConfigStore VERSION 0.1.0 LANGUAGES CXX)

set(_superjupiter_config_store_top_level OFF)
if(CMAKE_CURRENT_SOURCE_DIR STREQUAL CMAKE_SOURCE_DIR)
  set(_superjupiter_config_store_top_level ON)
endif()
option(SUPERJUPITER_CONFIG_STORE_BUILD_TESTS "Build ConfigStore tests" ${_superjupiter_config_store_top_level})
option(SUPERJUPITER_CONFIG_STORE_BUILD_EXAMPLES "Build ConfigStore host example" ${_superjupiter_config_store_top_level})
option(SUPERJUPITER_CONFIG_STORE_INSTALL "Generate ConfigStore install rules" ${_superjupiter_config_store_top_level})

add_library(SuperJupiterConfigStore INTERFACE)
add_library(SuperJupiter::ConfigStore ALIAS SuperJupiterConfigStore)
set_target_properties(SuperJupiterConfigStore PROPERTIES EXPORT_NAME ConfigStore)
target_include_directories(SuperJupiterConfigStore INTERFACE
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src>
  $<INSTALL_INTERFACE:include>)
target_compile_features(SuperJupiterConfigStore INTERFACE cxx_std_11)

if(SUPERJUPITER_CONFIG_STORE_BUILD_TESTS)
  enable_testing()
  add_executable(superjupiter-config-store-tests tests/test.cpp)
  target_link_libraries(superjupiter-config-store-tests PRIVATE SuperJupiter::ConfigStore)
  add_test(NAME superjupiter-config-store-tests COMMAND superjupiter-config-store-tests)
  add_executable(superjupiter-config-store-namespace-test tests/namespace_isolation.cpp)
  target_link_libraries(superjupiter-config-store-namespace-test PRIVATE SuperJupiter::ConfigStore)
  add_test(NAME superjupiter-config-store-namespace-test COMMAND superjupiter-config-store-namespace-test)
endif()
if(SUPERJUPITER_CONFIG_STORE_BUILD_EXAMPLES)
  add_executable(superjupiter-config-store-replay tools/replay.cpp)
  target_link_libraries(superjupiter-config-store-replay PRIVATE SuperJupiter::ConfigStore)
endif()
if(SUPERJUPITER_CONFIG_STORE_INSTALL)
  include(CMakePackageConfigHelpers)
  set(_superjupiter_config_store_package_dir lib/cmake/SuperJupiterConfigStore)
  install(TARGETS SuperJupiterConfigStore EXPORT SuperJupiterConfigStoreTargets)
  install(FILES src/SuperJupiterConfigStore.h DESTINATION include)
  install(EXPORT SuperJupiterConfigStoreTargets NAMESPACE SuperJupiter::
    DESTINATION ${_superjupiter_config_store_package_dir})
  configure_package_config_file(cmake/SuperJupiterConfigStoreConfig.cmake.in
    ${CMAKE_CURRENT_BINARY_DIR}/SuperJupiterConfigStoreConfig.cmake
    INSTALL_DESTINATION ${_superjupiter_config_store_package_dir})
  write_basic_package_version_file(${CMAKE_CURRENT_BINARY_DIR}/SuperJupiterConfigStoreConfigVersion.cmake
    VERSION ${PROJECT_VERSION} COMPATIBILITY SameMinorVersion ARCH_INDEPENDENT)
  install(FILES ${CMAKE_CURRENT_BINARY_DIR}/SuperJupiterConfigStoreConfig.cmake
    ${CMAKE_CURRENT_BINARY_DIR}/SuperJupiterConfigStoreConfigVersion.cmake
    DESTINATION ${_superjupiter_config_store_package_dir})
  install(FILES LICENSE DESTINATION share/licenses/SuperJupiterConfigStore)
endif()
