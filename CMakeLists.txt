CMAKE_MINIMUM_REQUIRED(VERSION 3.0)
PROJECT (MUMPS VERSION 4.10.0 LANGUAGES C Fortran)

# fortran mangling currently fixed to Add_

option(BUILD_TESTS "${PROJECT_NAME} - Build tests" ON)
#option(USE_METIS "${PROJECT_NAME} - Use METIS 4" ON)
#option(USE_PARMETIS "${PROJECT_NAME} - Use ParMETIS (incl. METIS)" OFF)
option(USE_SCOTCH "${PROJECT_NAME} - Use Scotch" OFF)
option(USE_PTSCOTCH "${PROJECT_NAME} - Use PT-Scotch" ON)
set(BUILD_PRECISION sdcz CACHE STRING "${PROJECT_NAME} - Build precisions (sdcz)")

FIND_PACKAGE(MPI REQUIRED)
SET(CMAKE_C_COMPILER ${MPI_C_COMPILER})
SET(CMAKE_Fortran_COMPILER ${MPI_Fortran_COMPILER})
FIND_PACKAGE(BLAS CONFIG REQUIRED)
FIND_PACKAGE(LAPACK CONFIG REQUIRED)
FIND_PACKAGE(SCALAPACK CONFIG REQUIRED)

#if (USE_PARMETIS)
#    message(STATUS "Use of MUMPS with ParMETIS (incl. METIS) requested. Looking for library...")
#    FIND_PACKAGE(PARMETIS CONFIG REQUIRED)
#elseif(USE_METIS)
#    message(STATUS "Use of MUMPS with METIS 4 requested. Looking for library...")
#    FIND_PACKAGE(METIS 4 CONFIG REQUIRED)
#endif()
if (USE_PTSCOTCH)
    message(STATUS "Use of MUMPS with PT-Scotch (incl. Scotch) requested. Looking for library...")
    FIND_PACKAGE(PTSCOTCH CONFIG REQUIRED)
elseif(USE_SCOTCH)
    message(STATUS "Use of MUMPS with Scotch requested. Looking for library...")
    FIND_PACKAGE(SCOTCH CONFIG REQUIRED)
endif()

add_subdirectory(PORD)

# collect sources
file(GLOB mumps src/mumps*.c src/mumps*.F src/tools*.F)

# Build objects for each precision and combine them into the later mumps target
SET(PREC_LIBS )
foreach(prec s d c z)
    if(BUILD_PRECISION MATCHES [${prec}])
        message(STATUS "Building precision '${prec}'")
        file(GLOB prec_src src/${prec}mu*.F)
        ADD_LIBRARY(${prec}mumps OBJECT ${prec_src} ${mumps})
        target_compile_definitions(${prec}mumps PRIVATE
             Add_ 
             MUMPS_ARITH=MUMPS_ARITH_${prec}
             pord)
        if (USE_PTSCOTCH)
            target_compile_definitions(${prec}mumps PRIVATE ptscotch)
        elseif(USE_SCOTCH)
            target_compile_definitions(${prec}mumps PRIVATE scotch)
        endif()
        list(APPEND PREC_LIBS "$<TARGET_OBJECTS:${prec}mumps>")
        TARGET_INCLUDE_DIRECTORIES(${prec}mumps PRIVATE 
            include PORD/include
            ${MPI_C_INCLUDE_PATH} ${MPI_Fortran_INCLUDE_PATH}
            $<$<BOOL:${USE_PTSCOTCH}>:
                $<TARGET_PROPERTY:scotch,INTERFACE_INCLUDE_DIRECTORIES>>
            $<$<BOOL:${USE_SCOTCH}>:
                $<TARGET_PROPERTY:scotch,INTERFACE_INCLUDE_DIRECTORIES>>
        )
        # Make sure the according header files are installed
        install(DIRECTORY include/ DESTINATION include
            FILES_MATCHING PATTERN "${prec}mu*.h")
    endif()
endforeach()

ADD_LIBRARY(mumps STATIC ${PREC_LIBS})
target_link_libraries(mumps PUBLIC blas scalapack pord
    $<$<BOOL:${USE_PTSCOTCH}>:ptscotch>
    $<$<BOOL:${USE_SCOTCH}>:scotch>)
set_target_properties(mumps PROPERTIES OUTPUT_NAME mumps-${MUMPS_VERSION})

# ParMETIS includes metis
#if (USE_PARMETIS)
#    target_compile_definitions(mumps PRIVATE parmetis)
#    target_link_libraries(mumps PUBLIC parmetis)
#elseif(USE_METIS)
#    target_compile_definitions(mumps PRIVATE metis)
#    target_link_libraries(mumps PUBLIC metis4)
#endif()
if (USE_PTSCOTCH)
    target_link_libraries(mumps PUBLIC ptscotch ptesmumps)
elseif(USE_SCOTCH)
    target_link_libraries(mumps PUBLIC scotch esmumps)
endif()

install(TARGETS mumps EXPORT mumps-config
    DESTINATION lib
    INCLUDES DESTINATION include)
# common headers
install(FILES include/mumps_c_types.h include/mumps_compat.h DESTINATION include)
install(EXPORT mumps-config DESTINATION lib/cmake/mumps-${MUMPS_VERSION})
include(CMakePackageConfigHelpers)
WRITE_BASIC_PACKAGE_VERSION_FILE(${CMAKE_CURRENT_BINARY_DIR}/mumps-config-version.cmake COMPATIBILITY AnyNewerVersion)
install(FILES ${CMAKE_CURRENT_BINARY_DIR}/mumps-config-version.cmake DESTINATION lib/cmake/mumps-${MUMPS_VERSION})

if (BUILD_TESTS)
    enable_testing()
    add_subdirectory(examples)
endif()