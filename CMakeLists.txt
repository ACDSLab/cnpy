CMAKE_MINIMUM_REQUIRED(VERSION 3.0 FATAL_ERROR)
if(COMMAND cmake_policy)
	cmake_policy(SET CMP0003 NEW)
endif(COMMAND cmake_policy)

project(cnpy)

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
set(LIBRARY_NAME ${PROJECT_NAME})
option(ENABLE_STATIC "Build static (.a) library" ON)

find_package(ZLIB REQUIRED)

include_directories(${ZLIB_INCLUDE_DIRS})

add_library(${LIBRARY_NAME} SHARED "cnpy.cpp")
target_link_libraries(cnpy ${ZLIB_LIBRARIES})

if (WIN32)
	install(TARGETS ${LIBRARY_NAME}
			EXPORT ${PROJECT_NAME}-targets
			RUNTIME DESTINATION lib
			PERMISSIONS OWNER_READ OWNER_WRITE OWNER_EXECUTE GROUP_READ GROUP_EXECUTE WORLD_READ WORLD_EXECUTE)
else()
	install(TARGETS ${LIBRARY_NAME}
			EXPORT ${PROJECT_NAME}-targets
			LIBRARY DESTINATION lib
			PERMISSIONS OWNER_READ OWNER_WRITE OWNER_EXECUTE GROUP_READ GROUP_EXECUTE WORLD_READ WORLD_EXECUTE)
endif()
if(ENABLE_STATIC)
    add_library(${LIBRARY_NAME}-static STATIC "cnpy.cpp")
    set_target_properties(cnpy-static PROPERTIES OUTPUT_NAME "cnpy")
    install(TARGETS "cnpy-static" ARCHIVE DESTINATION lib)
endif(ENABLE_STATIC)

install(FILES "cnpy.h"
		DESTINATION include)
install(FILES "mat2npz" "npy2mat" "npz2mat"
		DESTINATION bin
		PERMISSIONS OWNER_READ OWNER_WRITE OWNER_EXECUTE GROUP_READ GROUP_EXECUTE WORLD_READ WORLD_EXECUTE)

add_executable(example1 example1.cpp)
target_link_libraries(example1 cnpy)

# Export project export set to the build tree
export(
  EXPORT ${PROJECT_NAME}-targets
  FILE "${PROJECT_BINARY_DIR}/${PROJECT_NAME}-targets.cmake"
  )

# Export the package for use from the build-tree
# (this registers the build-tree with a global CMake-registry ~/.cmake)
export(PACKAGE ${PROJECT_NAME})

include(CMakePackageConfigHelpers)

set(TMP_INCLUDE_INSTALL_DIR ${PROJECT_SOURCE_DIR})
configure_package_config_file(
  ${PROJECT_SOURCE_DIR}/cmake/Modules/project-config.cmake.in
  ${PROJECT_BINARY_DIR}/${PROJECT_NAME}-config.cmake
  INSTALL_DESTINATION ${PROJECT_BINARY_DIR}
  PATH_VARS TMP_INCLUDE_INSTALL_DIR
  )
