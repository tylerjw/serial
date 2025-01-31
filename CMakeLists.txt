cmake_minimum_required(VERSION 3.8)
project(serial)

# Find ROS 2 packages
find_package(ament_cmake REQUIRED)
find_package(rclcpp REQUIRED)

# Platform-specific settings
if(APPLE)
    find_library(IOKIT_LIBRARY IOKit)
    find_library(FOUNDATION_LIBRARY Foundation)
endif()

if(UNIX AND NOT APPLE)
    set(rt_LIBRARIES rt)
    set(pthread_LIBRARIES pthread)
endif()

# Sources
set(serial_SRCS
    src/serial.cc
    include/serial/serial.h
    include/serial/v8stdint.h
)

if(APPLE)
    list(APPEND serial_SRCS src/impl/unix.cc src/impl/list_ports/list_ports_osx.cc)
elseif(UNIX)
    list(APPEND serial_SRCS src/impl/unix.cc src/impl/list_ports/list_ports_linux.cc)
else()
    list(APPEND serial_SRCS src/impl/win.cc src/impl/list_ports/list_ports_win.cc)
endif()

# Add serial library
add_library(${PROJECT_NAME} SHARED ${serial_SRCS})

if(WIN32)
    target_link_libraries(${PROJECT_NAME} setupapi)
elif(APPLE)
    target_link_libraries(${PROJECT_NAME} ${FOUNDATION_LIBRARY} ${IOKIT_LIBRARY})
elif(UNIX)
    target_link_libraries(${PROJECT_NAME} rt pthread)
endif()

# Include directories
target_include_directories(${PROJECT_NAME}
  PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
    $<INSTALL_INTERFACE:include>
)

# Install library
install(TARGETS ${PROJECT_NAME}
  ARCHIVE DESTINATION lib
  LIBRARY DESTINATION lib
  RUNTIME DESTINATION bin
)

# Install headers
install(DIRECTORY include/serial/
  DESTINATION include/serial
)

# Build example
add_executable(serial_example examples/serial_example.cc)
ament_target_dependencies(serial_example rclcpp)
target_link_libraries(serial_example ${PROJECT_NAME})

# Install example
install(TARGETS serial_example
  DESTINATION lib/${PROJECT_NAME}
)

# # Unit tests
# if(BUILD_TESTING)
#   find_package(ament_cmake_gtest REQUIRED)
#   add_subdirectory(tests)
# endif()

# Export package
ament_export_dependencies(rclcpp)
ament_export_include_directories(include)
ament_export_libraries(${PROJECT_NAME})
ament_package()
