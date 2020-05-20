cmake_minimum_required(VERSION 2.8.3)
project(roswasm_webgui)

if (DEFINED ENV{EMSDK})

set(CMAKE_C_COMPILER emcc)
set(CMAKE_CXX_COMPILER em++)

## Compile as C++11, supported in ROS Kinetic and newer
# add_compile_options(-std=c++11)

find_package(catkin REQUIRED COMPONENTS roscpp roswasm std_msgs sensor_msgs geometry_msgs nav_msgs)

###################################
## catkin specific configuration ##
###################################
## The catkin_package macro generates cmake config files for your package
## Declare things to be passed to dependent projects
## INCLUDE_DIRS: uncomment this if your package contains header files
## LIBRARIES: libraries you create in this project that dependent projects also need
## CATKIN_DEPENDS: catkin_packages dependent projects also need
## DEPENDS: system dependencies of this project that dependent projects also need
catkin_package(
  INCLUDE_DIRS include
  LIBRARIES roswasm_webgui
  CATKIN_DEPENDS roscpp roswasm
#  DEPENDS system_lib
)

###########
## Build ##
###########

# setting EMSCRIPTEN_GENERATE_BITCODE_STATIC_LIBRARIES=ON here does not
# work, since it will only be taken into account on the next cmake
# configuration. So we explicitly run the code guarded by
# EMSCRIPTEN_GENERATE_BITCODE_STATIC_LIBRARIES=ON in Emscripten.cmake
# platform file.
set(CMAKE_STATIC_LIBRARY_SUFFIX ".bc")
set(CMAKE_C_CREATE_STATIC_LIBRARY "<CMAKE_C_COMPILER> -o <TARGET> <LINK_FLAGS> <OBJECTS>")
set(CMAKE_CXX_CREATE_STATIC_LIBRARY "<CMAKE_CXX_COMPILER> -o <TARGET> <LINK_FLAGS> <OBJECTS>")
# --- end EMSCRIPTEN_GENERATE_BITCODE_STATIC_LIBRARIES

set(EMCC_FLAGS "")
#set(EMCC_FLAGS "${EMCC_FLAGS} -s DISABLE_EXCEPTION_CATCHING=0")
set(EMCC_FLAGS "${EMCC_FLAGS} -s ERROR_ON_UNDEFINED_SYMBOLS=1")
set(EMCC_FLAGS "${EMCC_FLAGS} -s ALLOW_MEMORY_GROWTH=1")
#set(EMCC_FLAGS "${EMCC_FLAGS} -s ASSERTIONS=1")
set(EMCC_FLAGS "${EMCC_FLAGS} -s WASM=1")
set(EMCC_FLAGS "${EMCC_FLAGS} -Wno-warn-absolute-paths")
#set(EMCC_FLAGS "${EMCC_FLAGS} --memory-init-file 0")
set(EMCC_FLAGS "${EMCC_FLAGS} -s USE_BOOST_HEADERS=1")

set(EMCC_FLAGS "${EMCC_FLAGS} -s USE_WEBGL2=1")
set(EMCC_FLAGS "${EMCC_FLAGS} -s USE_GLFW=3")
set(EMCC_FLAGS "${EMCC_FLAGS} -s FULL_ES3=1")
set(EMCC_FLAGS "${EMCC_FLAGS} -s USE_SDL=2")
set(EMCC_FLAGS "${EMCC_FLAGS} -s USE_SDL_IMAGE=2")
set(EMCC_FLAGS "${EMCC_FLAGS} -s SDL2_IMAGE_FORMATS='[\"png\", \"jpg\"]'")
#set(EMCC_FLAGS "${EMCC_FLAGS} --preload-file data")

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${EMCC_FLAGS}")
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${EMCC_FLAGS}")

add_definitions("-g4 -O2 -std=c++11") # -lwebsocket.js -O2 -s WASM=1 -s USE_BOOST_HEADERS=1")

## Specify additional locations of header files
## Your package locations should be listed before other locations
include_directories(
  include
  ${catkin_INCLUDE_DIRS}
)

add_library(roswasm_webgui STATIC src/imgui_impl_glfw.cpp src/imgui_impl_opengl3.cpp src/imgui.cpp src/imgui_draw.cpp src/imgui_demo.cpp src/roswasm_monlaunch.cpp src/roswasm_image.cpp src/roswasm_widget.cpp src/roswasm_examples.cpp) # src/roswasm_sam.cpp)
#)
#set_target_properties(roswasm PROPERTIES SUFFIX ".bc")

add_executable(roswasm_webgui_test src/main.cpp)
set_target_properties(roswasm_webgui_test PROPERTIES OUTPUT_NAME "webgui.js")

target_link_libraries(roswasm_webgui roswasm -lGL -lwebsocket.js)
target_link_libraries(roswasm_webgui_test roswasm_webgui roswasm -lGL -lwebsocket.js)

#############
## Install ##
#############

# all install targets should use catkin DESTINATION variables
# See http://ros.org/doc/api/catkin/html/adv_user_guide/variables.html

## Mark executable scripts (Python etc.) for installation
## in contrast to setup.py, you can choose the destination
# install(PROGRAMS
#   scripts/my_python_script
#   DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
# )

## Mark executables for installation
## See http://docs.ros.org/melodic/api/catkin/html/howto/format1/building_executables.html
# install(TARGETS ${PROJECT_NAME}_node
#   RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
# )

## Mark libraries for installation
## See http://docs.ros.org/melodic/api/catkin/html/howto/format1/building_libraries.html
# install(TARGETS ${PROJECT_NAME}
#   ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
#   LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
#   RUNTIME DESTINATION ${CATKIN_GLOBAL_BIN_DESTINATION}
# )

## Mark cpp header files for installation
# install(DIRECTORY include/${PROJECT_NAME}/
#   DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION}
#   FILES_MATCHING PATTERN "*.h"
#   PATTERN ".svn" EXCLUDE
# )

## Mark other files for installation (e.g. launch and bag files, etc.)
# install(FILES
#   # myfile1
#   # myfile2
#   DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}
# )

#############
## Testing ##
#############

## Add gtest based cpp test target and link libraries
# catkin_add_gtest(${PROJECT_NAME}-test test/test_roswasm_webgui.cpp)
# if(TARGET ${PROJECT_NAME}-test)
#   target_link_libraries(${PROJECT_NAME}-test ${PROJECT_NAME})
# endif()

## Add folders to be run by python nosetests
# catkin_add_nosetests(test)

else() # no emsdk

find_package(catkin REQUIRED COMPONENTS roscpp roscpp_json_serialize std_msgs sensor_msgs geometry_msgs nav_msgs) # rosmon_msgs)
catkin_package()

endif()