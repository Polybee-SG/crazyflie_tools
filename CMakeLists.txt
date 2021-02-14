cmake_minimum_required(VERSION 3.5)
project(crazyflie_tools)

# Enable C++11 and warnings
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

add_compile_options(-Wall -Wextra -Werror)

# Update the policy setting to avoid an error when loading the ament_cmake package
# at the current cmake version level
if(POLICY CMP0057)
  cmake_policy(SET CMP0057 NEW)
endif()

find_library(USB_LIB usb-1.0)
find_package(Boost REQUIRED COMPONENTS program_options REQUIRED)
find_package(ament_cmake QUIET)

if ( ament_cmake_FOUND )
  message(STATUS "------------------------------------------")
  message(STATUS "crazyflie_tools is being built using AMENT.")
  message(STATUS "------------------------------------------")

  find_package(crazyflie_cpp REQUIRED)

  ament_export_dependencies(crazyflie_cpp)
  ament_package()

  set( CRAZYFLIE_TOOLS_BIN_DESTINATION   lib/${PROJECT_NAME} )

elseif( CATKIN_DEVEL_PREFIX OR CATKIN_BUILD_BINARY_PACKAGE)
  message(STATUS "------------------------------------------")
  message(STATUS "crazyflie_tools is being built using CATKIN.")
  message(STATUS "------------------------------------------")

  find_package(catkin REQUIRED COMPONENTS
    crazyflie_cpp
  )

  catkin_package(
    CATKIN_DEPENDS crazyflie_cpp
  )

  include_directories(
    ${catkin_INCLUDE_DIRS}
  )

  set( CRAZYFLIE_TOOLS_BIN_DESTINATION   ${CATKIN_GLOBAL_BIN_DESTINATION} )

else()
  message(STATUS "------------------------------------------")
  message(STATUS "crazyflie_tools is being built using standard CMake.")
  message(STATUS "------------------------------------------")

  add_subdirectory(crazyflie_cpp)

  include_directories(
    crazyflie_cpp/include
  )

  set( CRAZYFLIE_TOOLS_BIN_DESTINATION   bin )
endif()

# scan
add_executable(scan src/scan.cpp)
list(APPEND executables scan)

# listParams
add_executable(listParams src/listParams.cpp)
list(APPEND executables listParams)

# listLogVariables
add_executable(listLogVariables src/listLogVariables.cpp)
list(APPEND executables listLogVariables)

# listMemories
add_executable(listMemories src/listMemories.cpp)
list(APPEND executables listMemories)

# reboot
add_executable(reboot src/reboot.cpp)
list(APPEND executables reboot)

# battery
add_executable(battery src/battery.cpp)
list(APPEND executables battery)

# version
add_executable(version src/version.cpp)
list(APPEND executables version)

# console
add_executable(console src/console.cpp)
list(APPEND executables console)

# flash
add_executable(flash src/flash.cpp)
list(APPEND executables flash)

# comCheck
add_executable(comCheck src/comCheck.cpp)
list(APPEND executables comCheck)

# log
add_executable(log src/log.cpp)
list(APPEND executables log)

# downloadUSDLogfile
add_executable(downloadUSDLogfile src/downloadUSDLogfile.cpp)
list(APPEND executables downloadUSDLogfile)

# setParam
add_executable(setParam src/setParam.cpp)
list(APPEND executables setParam)

if ( ament_cmake_FOUND )
  foreach(exec ${executables})
    ament_target_dependencies(${exec} crazyflie_cpp)
    target_link_libraries(${exec} ${Boost_LIBRARIES} ${USB_LIB})
  endforeach()
elseif( CATKIN_DEVEL_PREFIX OR CATKIN_BUILD_BINARY_PACKAGE)
  foreach(exec ${executables})
    add_dependencies(${exec} ${${PROJECT_NAME}_EXPORTED_TARGETS} ${catkin_EXPORTED_TARGETS})
    target_link_libraries(${exec} ${catkin_LIBRARIES} ${Boost_LIBRARIES})
  endforeach()
else()
  foreach(exec ${executables})
    target_link_libraries(${exec} crazyflie_cpp ${Boost_LIBRARIES})
  endforeach()
endif()

install(TARGETS
  ${executables}
  RUNTIME DESTINATION ${CRAZYFLIE_TOOLS_BIN_DESTINATION}
)
