cmake_minimum_required (VERSION 2.8)
project(Rocon)

load_cache(../)
IF(NOT ENABLE_MPI)
  add_definitions( -DDUMMY_MPI )
ELSE()
  FIND_PACKAGE(MPI REQUIRED)
  add_definitions( -DMPICH_IGNORE_CXX_SEEK )
  INCLUDE_DIRECTORIES(${MPI_INCLUDE_PATH})
ENDIF()

#INCLUDE(CTest)
find_path(ROCCOM_INC roccom.h HINTS ../Roccom/include)

set (LIB_SRCS src/Rocon.C src/ray-triangle.C src/GEM.C
              src/GeoPartFunctions.C src/Partition.C
              src/RemeshFunctions.C src/RocfluFunctions.C
              src/TRAIL.C src/TRAIL_UnixUtils.C src/TransferFunctions.C
              src/PMesh.C src/BSMesh.C src/Mesh.C src/GeoPrimitives.C)
set (UTIL_SRCS src/flotsam.C src/genflu.C src/wrl2hdf.C)
set (ALL_SRCS ${LIB_SRCS} ${UTIL_SRCS})

set_source_files_properties(${ALL_SRCS} PROPERTIES COMPILE_FLAGS "-fPIC ${MPI_CXX_COMPILE_FLAGS}" )
#set(TEST_SRCS src/MANTest.C)

# rpath settings
SET(CMAKE_INSTALL_RPATH "${CMAKE_INSTALL_PREFIX}/lib")
SET(CMAKE_INSTALL_RPATH_USE_LINK_PATH TRUE)

include_directories(include ${ROCCOM_INC})


IF(NOT BUILD_STATIC)
  add_library(Rocon SHARED ${LIB_SRCS})
ELSE()
  add_library(Rocon STATIC ${LIB_SRCS})
  add_definitions( -DSTATIC_LINK )
ENDIF()
target_link_libraries(Rocon Roccom ${IRAD_LIB} ${MPI_CXX_LIBRARIES})

add_executable(flotsam src/flotsam.C)
add_executable(genflu  src/genflu.C )
#add_executable(genstar src/genstar.C)
add_executable(wrl2hdf src/wrl2hdf.C)
#add_executable(profane src/profane.C)
target_link_libraries(flotsam Rocon)
target_link_libraries(genflu  Rocon)
#target_link_libraries(genstar Rocon)
target_link_libraries(wrl2hdf Rocon)
#target_link_libraries(profane Rocon)
SET_TARGET_PROPERTIES(Rocon PROPERTIES LINK_FLAGS "${MPI_CXX_LINK_FLAGS}")
SET_TARGET_PROPERTIES(flotsam PROPERTIES LINK_FLAGS "${MPI_CXX_LINK_FLAGS}")
SET_TARGET_PROPERTIES(wrl2hdf PROPERTIES LINK_FLAGS "${MPI_CXX_LINK_FLAGS}")

#ADD_TEST(RunAllTests ${EXECUTABLE_OUTPUT_PATH}/testx testresults.txt)
#ADD_TEST(TestObject:ValidHandle ${TEST_RESULTS} TestObject:ValidHandle testresults.txt)
#ADD_TEST(TestObject:Exists ${TEST_RESULTS} TestObject:Exists testresults.txt)

INSTALL(TARGETS Rocon flotsam genflu wrl2hdf RUNTIME DESTINATION bin LIBRARY DESTINATION lib ARCHIVE DESTINATION lib)
