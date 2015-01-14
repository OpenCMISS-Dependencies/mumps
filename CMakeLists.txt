CMAKE_MINIMUM_REQUIRED(VERSION 3.0)
PROJECT (MUMPS VERSION 4.10.0 LANGUAGES C Fortran)
SET(CMAKE_NO_SYSTEM_FROM_IMPORTED YES)

option(BUILD_TESTS "${PROJECT_NAME} - Build tests" ON)
#option(USE_METIS "${PROJECT_NAME} - Use METIS 4" ON)
#option(USE_PARMETIS "${PROJECT_NAME} - Use ParMETIS (incl. METIS)" OFF)
option(USE_SCOTCH "${PROJECT_NAME} - Use Scotch" OFF)
option(USE_PTSCOTCH "${PROJECT_NAME} - Use PT-Scotch" ON)
option(FORTRAN_MANGLING "${PROJECT_NAME} - Fortran mangling scheme" Add_)
set(BUILD_PRECISION sdcz CACHE STRING "${PROJECT_NAME} - Build precisions (sdcz)")

FIND_PACKAGE(MPI REQUIRED)
SET(CMAKE_C_COMPILER ${MPI_C_COMPILER})
SET(CMAKE_Fortran_COMPILER ${MPI_Fortran_COMPILER})
find_package(BLAS ${BLAS_VERSION} REQUIRED)
find_package(LAPACK ${LAPACK_VERSION} REQUIRED)
find_package(SCALAPACK ${SCALAPACK_VERSION} REQUIRED)

#if (USE_PARMETIS)
#    message(STATUS "Use of MUMPS with ParMETIS (incl. METIS) requested. Looking for library...")
#    FIND_PACKAGE(PARMETIS REQUIRED)
#elseif(USE_METIS)
#    message(STATUS "Use of MUMPS with METIS 4 requested. Looking for library...")
#    FIND_PACKAGE(METIS 4 REQUIRED)
#endif()
if (USE_PTSCOTCH)
    message(STATUS "Use of MUMPS with PT-Scotch (incl. Scotch) requested. Looking for library...")
    FIND_PACKAGE(PTSCOTCH ${PTSCOTCH_VERSION} REQUIRED)
elseif(USE_SCOTCH)
    message(STATUS "Use of MUMPS with Scotch requested. Looking for library...")
    FIND_PACKAGE(SCOTCH ${SCOTCH_VERSION} REQUIRED)
endif()

add_subdirectory(PORD)

# collect sources
SET(mumps-src
    src/mumps_part9.F
    src/mumps_common
    src/mumps_ooc_common.F
    src/mumps_orderings
    src/mumps_size
    src/mumps_io
    src/mumps_io_basic
    src/mumps_io_thread
    src/mumps_io_err
    src/mumps_static_mapping.F
    src/mumps_sol_es.F
	src/tools_common_mod.F
)
ADD_LIBRARY(mumps_common ${mumps-src})
target_link_libraries(mumps_common PUBLIC blas scalapack pord
    $<$<BOOL:${USE_PTSCOTCH}>:ptscotch>
    $<$<BOOL:${USE_SCOTCH}>:scotch>
    #${MPI_C_LIBRARIES} ${MPI_Fortran_LIBRARIES}
)
target_compile_definitions(mumps_common PRIVATE ${FORTRAN_MANGLING} pord) 
set_target_properties(mumps_common PROPERTIES OUTPUT_NAME mumps_common-${MUMPS_VERSION})
TARGET_INCLUDE_DIRECTORIES(mumps_common PRIVATE include PORD/include)
if (USE_PTSCOTCH)
    target_compile_definitions(mumps_common PRIVATE ptscotch)
elseif(USE_SCOTCH)
    target_compile_definitions(mumps_common PRIVATE scotch)
endif()

# Build objects for each precision and combine them into the later mumps target
SET(PREC_LIBS )
foreach(prec s d c z)
    if(BUILD_PRECISION MATCHES [${prec}])
        message(STATUS "Building precision '${prec}'")
        file(GLOB prec_src src/${prec}mu*.F)
        ADD_LIBRARY(${prec}mumps ${prec_src} src/mumps_c.c)
        set_target_properties(${prec}mumps PROPERTIES
            OUTPUT_NAME ${prec}mumps-${MUMPS_VERSION})
        target_link_libraries(${prec}mumps PUBLIC
            mumps_common pord
            #${MPI_C_LIBRARIES} ${MPI_Fortran_LIBRARIES}
        )
        target_compile_definitions(${prec}mumps PRIVATE
             ${FORTRAN_MANGLING}
             MUMPS_ARITH=MUMPS_ARITH_${prec}
             pord)
        if (USE_PTSCOTCH)
            target_compile_definitions(${prec}mumps PRIVATE ptscotch)
        elseif(USE_SCOTCH)
            target_compile_definitions(${prec}mumps PRIVATE scotch)
        endif()
        list(APPEND PREC_LIBS ${prec}mumps)
        if (USE_PTSCOTCH)
            target_link_libraries(${prec}mumps PUBLIC ptscotch ptesmumps)
        elseif(USE_SCOTCH)
            target_link_libraries(${prec}mumps PUBLIC scotch esmumps)
        endif()
        TARGET_INCLUDE_DIRECTORIES(${prec}mumps PRIVATE 
            include PORD/include
            #${MPI_C_INCLUDE_PATH} ${MPI_Fortran_INCLUDE_PATH}
            #$<$<BOOL:${USE_PTSCOTCH}>:
            #    $<TARGET_PROPERTY:scotch,INTERFACE_INCLUDE_DIRECTORIES>>
            #$<$<BOOL:${USE_SCOTCH}>:
            #    $<TARGET_PROPERTY:scotch,INTERFACE_INCLUDE_DIRECTORIES>>
        )
        # Make sure the according header files are installed
        install(DIRECTORY include/ DESTINATION include
            FILES_MATCHING PATTERN "${prec}mu*.h")
    endif()
endforeach()

# ParMETIS includes metis
#if (USE_PARMETIS)
#    target_compile_definitions(mumps PRIVATE parmetis)
#    target_link_libraries(mumps PUBLIC parmetis)
#elseif(USE_METIS)
#    target_compile_definitions(mumps PRIVATE metis)
#    target_link_libraries(mumps PUBLIC metis)
#endif()
#if (USE_PTSCOTCH)
#    target_link_libraries(mumps PUBLIC ptscotch ptesmumps)
#elseif(USE_SCOTCH)
#    target_link_libraries(mumps PUBLIC scotch esmumps)
#endif()

install(TARGETS mumps_common ${PREC_LIBS} EXPORT mumps-config
    DESTINATION lib
    INCLUDES DESTINATION include/mumps)
# common headers
install(FILES include/mumps_c_types.h include/mumps_compat.h DESTINATION include/mumps)
install(EXPORT mumps-config DESTINATION lib/cmake)
include(CMakePackageConfigHelpers)
WRITE_BASIC_PACKAGE_VERSION_FILE(${CMAKE_CURRENT_BINARY_DIR}/mumps-config-version.cmake COMPATIBILITY AnyNewerVersion)
install(FILES ${CMAKE_CURRENT_BINARY_DIR}/mumps-config-version.cmake DESTINATION lib/cmake)

if (BUILD_TESTS)
    enable_testing()
    add_subdirectory(examples)
endif()