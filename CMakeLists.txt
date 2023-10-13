cmake_minimum_required(VERSION 3.8)
project(tracy_profiler)

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

# find dependencies
find_package(ament_cmake REQUIRED)

option(TRACY_ENABLE "Enable profiling" ON)

# Make library available
add_library(${PROJECT_NAME} SHARED
  tracy/public/TracyClient.cpp
)

set(TRACY_INLCUDE_DIR tracy/public)

target_include_directories(${PROJECT_NAME} PUBLIC
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/${TRACY_INLCUDE_DIR}>
  $<INSTALL_INTERFACE:${TRACY_INLCUDE_DIR}>
)

target_compile_definitions(${PROJECT_NAME} PUBLIC
  TRACY_ENABLE
)

ament_export_libraries(${PROJECT_NAME})
ament_export_include_directories(${TRACY_INLCUDE_DIR})

ament_export_targets(export_${PROJECT_NAME} HAS_LIBRARY_TARGET)

# Compile server
add_custom_target(build_tracy_server
    COMMAND sh -c "CC=clang CXX=clang++ make release LEGACY=1 -C profiler/build/unix -j `nproc`"
    WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/tracy
)

add_dependencies(${PROJECT_NAME} build_tracy_server)

set(TRACY_SERVER_PATH ${CMAKE_CURRENT_SOURCE_DIR}/tracy/profiler/build/unix)
file(GLOB TRACY_SERVER_FILES "${TRACY_SERVER_PATH}/Tracy-*")

# Install

install(TARGETS 
  ${PROJECT_NAME}
  EXPORT export_${PROJECT_NAME}
  LIBRARY DESTINATION lib
  ARCHIVE DESTINATION lib
  RUNTIME DESTINATION bin
)

install(FILES ${TRACY_SERVER_FILES}
  DESTINATION bin
)

install(
  DIRECTORY ${TRACY_INLCUDE_DIR}/
  DESTINATION ${TRACY_INLCUDE_DIR}
)

ament_package()
