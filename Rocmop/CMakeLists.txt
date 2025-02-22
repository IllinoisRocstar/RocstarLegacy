cmake_minimum_required (VERSION 2.8)
project(Rocmop)

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
set (ROCCOM_DIR "${ROCCOM_INC}/..") 
set(MESQUITE_DIR ${PROJECT_SOURCE_DIR}/External/mesquite_0_9_5)
set(ROCMOP2_SRCS src/Rocmop_2.C src/smooth_mesquite_2.C src/smooth_mesquite_ng_2.C
                 src/smooth_medial_2.C)
set(ROCMOP_SRCS src/Rocmop_1.C src/smooth_mesquite_1.C src/smooth_mesquite_ng_1.C 
                src/smooth_medial_1.C)
set (LIB_SRCS src/geometry.C src/Matrix.C src/Algebraic_Metrics_2.C src/Algebraic_Metrics_3.C
              src/Geometric_Metrics_2.C src/Geometric_Metrics_3.C src/PN_patch.C)
IF(ROCMOP2)
  set(LIB_SRCS ${LIB_SRCS} ${ROCMOP2_SRCS} src/MesqPane_95.C)
ELSE()
  set(LIB_SRCS ${LIB_SRCS} ${ROCMOP_SRCS} src/MesqPane_95.C)
ENDIF()
set (UTIL_SRCS util/chkPconnGRecv.cc util/RocMeshData.cc)
set (TEST_SRCS test/add_aspect_ratios.C test/build_meshes.C test/metric_demo.C 
               test/smooth_volume.C)
set (ALL_SRCS ${LIB_SRCS} ${TEST_SRCS} ${UTIL_SRCS})

set_source_files_properties(${ALL_SRCS} PROPERTIES COMPILE_FLAGS "-fPIC ${MPI_CXX_COMPILE_FLAGS}" )
#set(TEST_SRCS src/MANTest.C)

# rpath settings
SET(CMAKE_INSTALL_RPATH "${CMAKE_INSTALL_PREFIX}/lib")
SET(CMAKE_INSTALL_RPATH_USE_LINK_PATH TRUE)

#MESQUITE        = 1        # this flag is used in Rocmop*.C
#
#OBJS            += MesqPane_95.o
#
#Rocmop_1.o Rocmop_2.o MesqPane_95.o: $(LIBMESQUITE)
#
#Note: use CXXFLAGS instead of CPPFLAGS to avoid include MESQUITE 
#      when generating Rocmop.d and Mesquite.d.
#CXXFLAGS        += -I$(MESQHOME)/include -I$(BUILD_MESQHOME)/includeLinks \
#		   -DMESQUITE
include_directories(include ${ROCCOM_INC} ${ROCCOM_DIR}/Rocmap/include ${ROCCOM_DIR}/Rocsurf/include ${ROCCOM_DIR}/Rocmap/test 
                            ${MESQUITE_DIR}/include ${MESQUITE_DIR}/includeLinks ${ROCCOM_DIR}/Rocsurf/test ${ROCCOM_DIR}/Rocblas/include)
add_definitions(-DMESQUITE)

add_subdirectory(External/mesquite_0_9_5)

IF(NOT BUILD_STATIC)
  add_library(Rocmop SHARED ${LIB_SRCS})
ELSE()
  add_library(Rocmop STATIC ${LIB_SRCS})
  add_definitions( -DSTATIC_LINK )
ENDIF()
add_dependencies(Rocmop MSQ_MV)
target_link_libraries(Rocmop Rocsurf Rocmap Rocblas Rocout Rocin Roccom mesquite_95 m ${MPI_CXX_LIBRARIES})

#set (UTIL_SRCS util/chkPconnGRecv.cc util/RocMeshData.cc)
#set (TEST_SRCS test/add_aspect_ratios.C test/build_meshes.C test/metric_demo.C 
#               test/smooth_volume.C)

add_executable(chkPconnGRecv ${UTIL_SRCS})
add_executable(add_aspect_ratios test/add_aspect_ratios.C)
add_executable(build_meshes test/build_meshes.C)
add_executable(metric_demo test/metric_demo.C)
add_executable(smooth_volume test/smooth_volume.C)
target_link_libraries(chkPconnGRecv Rocmop ${HDF5_LIB})
target_link_libraries(add_aspect_ratios Rocmop ${HDF5_LIB})
target_link_libraries(build_meshes Rocmop ${HDF5_LIB})
target_link_libraries(metric_demo Rocmop ${HDF5_LIB})
target_link_libraries(smooth_volume Rocmop ${HDF5_LIB})

SET_TARGET_PROPERTIES(Rocmop PROPERTIES LINK_FLAGS "${MPI_CXX_LINK_FLAGS}")
SET_TARGET_PROPERTIES(add_aspect_ratios PROPERTIES LINK_FLAGS "${MPI_CXX_LINK_FLAGS}")
SET_TARGET_PROPERTIES(build_meshes PROPERTIES LINK_FLAGS "${MPI_CXX_LINK_FLAGS}")
SET_TARGET_PROPERTIES(metric_demo PROPERTIES LINK_FLAGS "${MPI_CXX_LINK_FLAGS}")
SET_TARGET_PROPERTIES(smooth_volume PROPERTIES LINK_FLAGS "${MPI_CXX_LINK_FLAGS}")

#ADD_TEST(RunAllTests ${EXECUTABLE_OUTPUT_PATH}/testx testresults.txt)
#ADD_TEST(TestObject:ValidHandle ${TEST_RESULTS} TestObject:ValidHandle testresults.txt)
#ADD_TEST(TestObject:Exists ${TEST_RESULTS} TestObject:Exists testresults.txt)

INSTALL(TARGETS Rocmop RUNTIME DESTINATION bin LIBRARY DESTINATION lib ARCHIVE DESTINATION lib)
