cmake_minimum_required(VERSION 2.8.3)
project(libdlib)

find_package(catkin REQUIRED)

catkin_destinations() # set-up destination variables
file(MAKE_DIRECTORY ${CATKIN_DEVEL_PREFIX}/${CATKIN_PACKAGE_INCLUDE_DESTINATION})

set(VERSION 19.9)

include(ExternalProject)
ExternalProject_Add(EP_${PROJECT_NAME}
    GIT_REPOSITORY "https://github.com/davisking/dlib.git"
    GIT_TAG "v${VERSION}"

    SOURCE_DIR ${PROJECT_BINARY_DIR}/${PROJECT_NAME}-src
    BINARY_DIR ${PROJECT_BINARY_DIR}/${PROJECT_NAME}-build
    
    CONFIGURE_COMMAND ${CMAKE_COMMAND} ${PROJECT_BINARY_DIR}/${PROJECT_NAME}-src/dlib
    BUILD_COMMAND ${CMAKE_COMMAND} --build . --config Release
        # copy headers to devel space (catkin does not like headers in source space)
        COMMAND ${CMAKE_COMMAND} -E copy_directory ${PROJECT_BINARY_DIR}/${PROJECT_NAME}-src/dlib ${CATKIN_DEVEL_PREFIX}/${CATKIN_PACKAGE_INCLUDE_DESTINATION}
        # copy libs, set-up soname chain
	COMMAND ${CMAKE_COMMAND} -E copy_if_different ${PROJECT_BINARY_DIR}/${PROJECT_NAME}-build/libdlib.so ${CATKIN_DEVEL_PREFIX}/${CATKIN_PACKAGE_LIB_DESTINATION}
	COMMAND ${CMAKE_COMMAND} -E copy_if_different ${PROJECT_BINARY_DIR}/${PROJECT_NAME}-build/libdlib.so.${VERSION}.0 ${CATKIN_DEVEL_PREFIX}/${CATKIN_PACKAGE_LIB_DESTINATION}
    INSTALL_COMMAND ""
)

catkin_package(
    CFG_EXTRAS libdlib-extras.cmake # catkin_package does not support artifacts in devel space
    EXPORTED_TARGETS EP_${PROJECT_NAME}
)

install(DIRECTORY ${CATKIN_DEVEL_PREFIX}/${CATKIN_PACKAGE_INCLUDE_DESTINATION}/
    DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION}
    FILES_MATCHING PATTERN "*.h"
)

install(FILES ${CATKIN_DEVEL_PREFIX}/${CATKIN_PACKAGE_LIB_DESTINATION}/libdlib.a
    DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
)
