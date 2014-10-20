PROJECT (MUMPS C Fortran)
CMAKE_MINIMUM_REQUIRED(VERSION 2.8 FATAL_ERROR)

SET(CMAKE_PREFIX_PATH ${OPENCMISS_DEPENDENCIES_CONFIGS_DIR} ${CMAKE_PREFIX_PATH})
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${OPENCMISS_DEPENDENCIES_LIBRARIES})

if (OCM_USE_PARMETIS)
    FIND_PACKAGE(PARMETIS CONFIG REQUIRED)
endif()
FIND_PACKAGE(MPI REQUIRED)

# create mtocpp
file(GLOB mumps src/mumps*.c src/mumps*.F src/tools*.F)
file(GLOB smumps src/smu*.F)
file(GLOB dmumps src/dmu*.F)
file(GLOB cmumps src/cmu*.F)
file(GLOB zmumps src/zmu*.F)

# Create only double prec version
#ADD_LIBRARY(mumps STATIC ${mumps} ${smumps} ${dmumps} ${cmumps} ${zmumps})
ADD_LIBRARY(mumps STATIC ${mumps} ${dmumps})

target_compile_definitions(mumps PRIVATE MUMPS_ARITH=MUMPS_ARITH_d)
# Only add parmetis if wanted
if (OCM_USE_PARMETIS)
    target_compile_definitions(mumps PRIVATE parmetis)
    target_link_libraries(mumps parmetis)
    TARGET_INCLUDE_DIRECTORIES(mumps PRIVATE 
        $<TARGET_PROPERTY:parmetis,INTERFACE_INCLUDE_DIRECTORIES>
    )
endif()

TARGET_INCLUDE_DIRECTORIES(mumps PRIVATE 
    include
    ${MPI_C_INCLUDE_PATH} ${MPI_Fortran_INCLUDE_PATH}
    #$<TARGET_PROPERTY:ocm_mpi,INTERFACE_INCLUDE_DIRECTORIES>
)