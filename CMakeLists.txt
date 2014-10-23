cmake_policy(SET CMP0048 NEW) # Allows VERSION in project definitions
CMAKE_MINIMUM_REQUIRED(VERSION 3.0 FATAL_ERROR)

PROJECT (MUMPS VERSION 4.10.0 LANGUAGES C Fortran)

if (OPENCMISS_DEPENDENCIES_LIBRARIES)
    SET(CMAKE_PREFIX_PATH ${OPENCMISS_DEPENDENCIES_CONFIGS_DIR} ${CMAKE_PREFIX_PATH})
    set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${OPENCMISS_DEPENDENCIES_LIBRARIES})
    set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${OPENCMISS_DEPENDENCIES_LIBRARIES})
else()
    SET(CMAKE_PREFIX_PATH ../cmake ${CMAKE_PREFIX_PATH})
    SET(OPENCMISS_DEPENDENCIES_CONFIGS_DIR ${CMAKE_CURRENT_BINARY_DIR})
    SET(OCM_USE_METIS YES)
    SET(OCM_USE_PARMETIS NO)
endif()

FIND_PACKAGE(BLAS CONFIG REQUIRED)
FIND_PACKAGE(LAPACK CONFIG REQUIRED)
FIND_PACKAGE(SCALAPACK CONFIG REQUIRED)
if (OCM_USE_METIS)
    message(STATUS "Use of MUMPS with METIS requested. Looking for library...")
    FIND_PACKAGE(METIS 4 CONFIG REQUIRED)
endif()
if (OCM_USE_PARMETIS)
    message(STATUS "Use of MUMPS with ParMETIS requested. Looking for library...")
    FIND_PACKAGE(PARMETIS CONFIG REQUIRED)
endif()
FIND_PACKAGE(MPI REQUIRED)

# create mtocpp
file(GLOB mumps src/mumps*.c src/mumps*.F src/tools*.F)
#file(GLOB smumps src/smu*.F)
file(GLOB dmumps src/dmu*.F)
#file(GLOB cmumps src/cmu*.F)
#file(GLOB zmumps src/zmu*.F)

# Create only double prec version
#ADD_LIBRARY(mumps STATIC ${mumps} ${smumps} ${dmumps} ${cmumps} ${zmumps})
ADD_LIBRARY(mumps STATIC ${mumps} ${dmumps})

target_compile_definitions(mumps PRIVATE Add_ MUMPS_ARITH=MUMPS_ARITH_d)
target_link_libraries(mumps PUBLIC blas scalapack)
set_target_properties(mumps PROPERTIES OUTPUT_NAME mumps-${MUMPS_VERSION})
# Only add parmetis if wanted
if (OCM_USE_METIS)
    target_compile_definitions(mumps PRIVATE metis)
    target_link_libraries(mumps PUBLIC metis)
    TARGET_INCLUDE_DIRECTORIES(mumps PRIVATE 
        $<TARGET_PROPERTY:metis,INTERFACE_INCLUDE_DIRECTORIES>
    )
endif()
# Only add parmetis if wanted
if (OCM_USE_PARMETIS)
    target_compile_definitions(mumps PRIVATE parmetis)
    target_link_libraries(mumps PUBLIC parmetis)
    TARGET_INCLUDE_DIRECTORIES(mumps PRIVATE 
        $<TARGET_PROPERTY:parmetis,INTERFACE_INCLUDE_DIRECTORIES>
    )
endif()

TARGET_INCLUDE_DIRECTORIES(mumps PRIVATE 
    include
    ${MPI_C_INCLUDE_PATH} ${MPI_Fortran_INCLUDE_PATH}
    #$<TARGET_PROPERTY:ocm_mpi,INTERFACE_INCLUDE_DIRECTORIES>
)

export(TARGETS mumps FILE ${OPENCMISS_DEPENDENCIES_CONFIGS_DIR}/mumps-config.cmake)
include(CMakePackageConfigHelpers)
WRITE_BASIC_PACKAGE_VERSION_FILE(${OPENCMISS_DEPENDENCIES_CONFIGS_DIR}/mumps-${MUMPS_VERSION}/mumps-config-version.cmake
    VERSION ${MUMPS_VERSION} COMPATIBILITY AnyNewerVersion)

enable_testing()
add_subdirectory(examples)