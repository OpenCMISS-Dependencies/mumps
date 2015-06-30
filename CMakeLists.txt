CMAKE_MINIMUM_REQUIRED(VERSION 3.0)

project(MUMPS VERSION 5.0.0 LANGUAGES C Fortran)
SET(CMAKE_NO_SYSTEM_FROM_IMPORTED YES)

option(BUILD_TESTS "${PROJECT_NAME} - Build tests" ON)
option(PACKAGE_CONFIG_DIR "Directory for package config files (relative to CMAKE_INSTALL_PREFIX)" "lib/cmake")

# As of MUMPS 5 we can use Metis 5.1 :-)
option(USE_METIS "${PROJECT_NAME} - Use METIS 5" ON)
option(USE_PARMETIS "${PROJECT_NAME} - Use ParMETIS" OFF)
option(USE_SCOTCH "${PROJECT_NAME} - Use Scotch" OFF)
option(USE_PTSCOTCH "${PROJECT_NAME} - Use PT-Scotch" ON)
set(FORTRAN_MANGLING Add_ CACHE STRING "${PROJECT_NAME} - Fortran mangling scheme")
set(BUILD_PRECISION sdcz CACHE STRING "${PROJECT_NAME} - Build precisions (sdcz)")

find_package(MPI REQUIRED)
if (MPI_C_COMPILER)
    SET(CMAKE_C_COMPILER ${MPI_C_COMPILER})
endif()
if (MPI_Fortran_COMPILER)
    SET(CMAKE_Fortran_COMPILER ${MPI_Fortran_COMPILER})
endif()
find_package(BLAS ${BLAS_VERSION} REQUIRED)
find_package(LAPACK ${LAPACK_VERSION} REQUIRED)
find_package(SCALAPACK ${SCALAPACK_VERSION} REQUIRED)

if (USE_PARMETIS)
    message(STATUS "Use of MUMPS with ParMETIS requested. Looking for library...")
    find_package(PARMETIS ${PARMETIS_VERSION} REQUIRED)
elseif(USE_METIS)
    message(STATUS "Use of MUMPS with METIS requested. Looking for library...")
    find_package(METIS ${METIS_VERSION} REQUIRED)
endif()
if (USE_PTSCOTCH)
    message(STATUS "Use of MUMPS with PT-Scotch (incl. Scotch) requested. Looking for library...")
    find_package(PTSCOTCH ${PTSCOTCH_VERSION} REQUIRED)
elseif(USE_SCOTCH)
    message(STATUS "Use of MUMPS with Scotch requested. Looking for library...")
    find_package(SCOTCH ${SCOTCH_VERSION} REQUIRED)
endif()

add_subdirectory(PORD)

# collect mumps_common sources
SET(mumps-src
    src/mumps_common.c
    src/mumps_orderings.c
    src/mumps_ooc_common.F
    src/mumps_size.c
    src/mumps_io.c
    src/mumps_io_basic.c
    src/mumps_io_thread.c
    src/mumps_io_err.c
    src/mumps_static_mapping.F
    src/mumps_sol_es.F
	src/tools_common_mod.F
	src/ana_omp_m.F
	src/tools_common_mod.F
    src/mumps_static_mapping.F
    src/mumps_sol_es.F
	src/fac_future_niv2_mod.F
    src/mumps_comm_ibcast.F
	src/mumps_ooc_common.F
    src/double_linked_list.F
    src/fac_asm_build_sort_index_m.F
    src/fac_asm_build_sort_index_ELT_m.F
    src/omp_tps_common_m.F
    src/mumps_l0_omp_m.F
    src/front_data_mgt_m.F
    src/fac_maprow_data_m.F
    src/fac_descband_data_m.F
    src/fac_ibct_data_m.F
    src/ana_orderings.F
    src/ana_AMDMF.F
    src/bcast_errors.F
    src/estim_flops.F
    src/mumps_type_size.F
    src/mumps_type2_blocking.F
    src/mumps_version.F
    src/tools_common.F
    src/mumps_print_defined.F
    #src/mumps_orderings.c
    #src/mumps_common.c
    #src/mumps_size.c
    #src/mumps_io.c
    #src/mumps_io_basic.c
    #src/mumps_io_thread.c
    #src/mumps_io_err.c
    src/mumps_numa.c
)

set(mumps-src-arith
    ana_aux_par.F
    ana_lr.F
    fac_asm_master_m.F
    fac_asm_master_ELT_m.F
    omp_tps_m.F
    mumps_comm_buffer.F
    mumps_load.F
    mumps_lr_data_m.F
    mumps_ooc_buffer.F
    mumps_ooc.F
    mumps_struc_def.F
    static_ptr_m.F
    lr_core.F
    fac_lr.F
    fac_omp_m.F
    fac_par_m.F
    fac_front_LU_type1.F
    fac_front_LU_type2.F
    fac_front_LDLT_type1.F
    fac_front_LDLT_type2.F
    fac_front_aux.F
    fac_front_type2_aux.F
    ini_driver.F
    ana_driver.F
    fac_driver.F
    sol_driver.F
    end_driver.F
    ana_aux_ELT.F
    ana_aux.F
    ana_dist_m.F
    ana_LDLT_preprocess.F
    ana_reordertree.F
    arrowheads.F
    bcast_int.F
    fac_asm_ELT.F
    fac_asm.F
    fac_b.F
    fac_distrib_distentry.F
    fac_distrib_ELT.F
    fac_lastrtnelind.F
    fac_mem_alloc_cb.F
    fac_mem_compress_cb.F
    fac_mem_free_block_cb.F
    fac_mem_stack_aux.F
    fac_mem_stack.F
    fac_process_band.F
    fac_process_blfac_slave.F
    fac_process_blocfacto_LDLT.F
    fac_process_blocfacto.F
    fac_process_bf.F
    fac_process_end_facto_slave.F
    fac_process_contrib_type1.F
    fac_process_contrib_type2.F
    fac_process_contrib_type3.F
    fac_process_maprow.F
    fac_process_master2.F
    fac_process_message.F
    fac_process_root2slave.F
    fac_process_root2son.F
    fac_process_rtnelind.F
    fac_root_parallel.F
    fac_scalings.F
    fac_determinant.F
    fac_scalings_simScaleAbs.F
    fac_scalings_simScale_util.F
    fac_sol_pool.F
    fac_type3_symmetrize.F
    ini_defaults.F
    mumps_driver.F
    mumps_f77.F
    mumps_iXamax.F
    ana_mtrans.F
    ooc_panel_piv.F
    rank_revealing.F
    sol_aux.F
    sol_bwd_aux.F
    sol_bwd.F
    sol_c.F
    sol_fwd_aux.F
    sol_fwd.F
    sol_matvec.F
    sol_root_parallel.F
    tools.F
    type3_root.F
    mumps_restart.F
)
ADD_LIBRARY(mumps_common ${mumps-src})
target_link_libraries(mumps_common PUBLIC blas scalapack pord
    $<$<BOOL:${USE_PTSCOTCH}>:ptesmumps>
    $<$<BOOL:${USE_SCOTCH}>:esmumps>
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
        # Compile sources for current arith
        set(mumps-src-arith-${prec} )
        foreach(entry ${mumps-src-arith})
            list(APPEND mumps-src-arith-${prec} src/${prec}${entry})
        endforeach()
        ADD_LIBRARY(${prec}mumps ${mumps-src-arith-${prec}} src/mumps_c.c)
        set_target_properties(${prec}mumps PROPERTIES
            OUTPUT_NAME ${prec}mumps-${MUMPS_VERSION})
        target_link_libraries(${prec}mumps PUBLIC mumps_common pord)
        target_compile_definitions(${prec}mumps PRIVATE
             ${FORTRAN_MANGLING}
             MUMPS_ARITH=MUMPS_ARITH_${prec}
             pord)
        if (USE_PARMETIS)
            target_compile_definitions(${prec}mumps PRIVATE parmetis)
            target_link_libraries(${prec}mumps PUBLIC parmetis)
        elseif(USE_METIS)
            target_compile_definitions(${prec}mumps PRIVATE metis)
            target_link_libraries(${prec}mumps PUBLIC metis)
        endif()
        if (USE_PTSCOTCH)
            target_compile_definitions(${prec}mumps PRIVATE ptscotch)
            target_link_libraries(${prec}mumps PUBLIC ptscotch ptesmumps)
        elseif(USE_SCOTCH)
            target_compile_definitions(${prec}mumps PRIVATE scotch)
            target_link_libraries(${prec}mumps PUBLIC scotch esmumps)
        endif()
        list(APPEND PREC_LIBS ${prec}mumps)
        TARGET_INCLUDE_DIRECTORIES(${prec}mumps PRIVATE 
            include PORD/include
        )
        # Make sure the according header files are installed
        install(DIRECTORY include/ DESTINATION include
            FILES_MATCHING PATTERN "${prec}mu*.h")
    endif()
endforeach()

# Add global interface target for easy linking if the build precisions are not known.
add_library(mumps INTERFACE)
target_link_libraries(mumps INTERFACE mumps_common ${PREC_LIBS} pord)

install(TARGETS mumps_common mumps ${PREC_LIBS} EXPORT mumps-config
    DESTINATION lib
    INCLUDES DESTINATION include/mumps)
# common headers
install(FILES include/mumps_c_types.h include/mumps_compat.h DESTINATION include/mumps)
install(EXPORT mumps-config DESTINATION ${PACKAGE_CONFIG_DIR})
# Fortran modules
SET(CMAKE_Fortran_MODULE_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/fortran_modules)
install(DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/fortran_modules/ DESTINATION include/mumps)

include(CMakePackageConfigHelpers)
WRITE_BASIC_PACKAGE_VERSION_FILE(${CMAKE_CURRENT_BINARY_DIR}/mumps-config-version.cmake COMPATIBILITY AnyNewerVersion)
file(WRITE ${CMAKE_CURRENT_BINARY_DIR}/mumps-config-dependencies.cmake 
    "include(CMakeFindDependencyMacro)\r\n"
    	"set(${PROJECT_NAME}_IMPORT_PREFIX \${_IMPORT_PREFIX})\r\n"
)
if (USE_PARMETIS)
    file(APPEND ${CMAKE_CURRENT_BINARY_DIR}/mumps-config-dependencies.cmake "find_dependency(PARMETIS ${PARMETIS_VERSION})\r\n")
endif()
if (USE_SCOTCH)
    file(APPEND ${CMAKE_CURRENT_BINARY_DIR}/mumps-config-dependencies.cmake "find_dependency(SCOTCH ${SCOTCH_VERSION})\r\n")
endif()
if (USE_PTSCOTCH)
    file(APPEND ${CMAKE_CURRENT_BINARY_DIR}/mumps-config-dependencies.cmake "find_dependency(PTSCOTCH ${PTSCOTCH_VERSION})\r\n")
endif()
file(APPEND ${CMAKE_CURRENT_BINARY_DIR}/mumps-config-dependencies.cmake "set(_IMPORT_PREFIX \${${PROJECT_NAME}_IMPORT_PREFIX})")
install(FILES 
    ${CMAKE_CURRENT_BINARY_DIR}/mumps-config-version.cmake
    ${CMAKE_CURRENT_BINARY_DIR}/mumps-config-dependencies.cmake
    DESTINATION ${PACKAGE_CONFIG_DIR})

if (BUILD_TESTS)
    enable_testing()
    add_subdirectory(examples)
endif()