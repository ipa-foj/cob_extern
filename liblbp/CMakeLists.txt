cmake_minimum_required(VERSION 2.8.3)
project(liblbp)

find_package(catkin REQUIRED)

catkin_destinations() # set-up destination variables
file(MAKE_DIRECTORY ${CATKIN_DEVEL_PREFIX}/${CATKIN_PACKAGE_INCLUDE_DESTINATION})

include(ExternalProject)
ExternalProject_Add(EP_${PROJECT_NAME}
    GIT_REPOSITORY "https://github.com/carolinepacheco/lbplibrary"

    SOURCE_DIR ${PROJECT_BINARY_DIR}/${PROJECT_NAME}-src
    BINARY_DIR ${PROJECT_BINARY_DIR}/${PROJECT_NAME}-build
    
    CONFIGURE_COMMAND ${CMAKE_COMMAND} ${PROJECT_BINARY_DIR}/${PROJECT_NAME}-src
    BUILD_COMMAND ${CMAKE_COMMAND} --build . --config Release
        # copy headers to devel space (catkin does not like headers in source space)
        COMMAND cd ${PROJECT_BINARY_DIR}/${PROJECT_NAME}-src/ && find . -name *.h -mindepth 3 | cut -d/ -f3 | xargs -i mkdir -p ${CATKIN_DEVEL_PREFIX}/${CATKIN_PACKAGE_INCLUDE_DESTINATION}/package_lbp/{}
        COMMAND cd ${PROJECT_BINARY_DIR}/${PROJECT_NAME}-src/ && find . -name *.h* | cut -c 2- | xargs -i cp ${PROJECT_BINARY_DIR}/${PROJECT_NAME}-src/{} ${CATKIN_DEVEL_PREFIX}/${CATKIN_PACKAGE_INCLUDE_DESTINATION}{}
        # copy lib, set-up soname chain
        COMMAND ${CMAKE_COMMAND} -E copy_if_different ${PROJECT_BINARY_DIR}/${PROJECT_NAME}-build/liblbp.so ${CATKIN_DEVEL_PREFIX}/${CATKIN_PACKAGE_LIB_DESTINATION}
    INSTALL_COMMAND ""
)

catkin_package(
    CFG_EXTRAS liblbp-extras.cmake # catkin_package does not support artifacts in devel space
    EXPORTED_TARGETS EP_${PROJECT_NAME}
)

install(DIRECTORY ${CATKIN_DEVEL_PREFIX}/${CATKIN_PACKAGE_INCLUDE_DESTINATION}/
    DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION}
    FILES_MATCHING PATTERN "*.h*"
)

install(FILES ${CATKIN_DEVEL_PREFIX}/${CATKIN_PACKAGE_LIB_DESTINATION}/liblbp.so
    DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
)
