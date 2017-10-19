cmake_minimum_required(VERSION 3.3)

project(MUMPS VERSION 5.0.0 LANGUAGES C Fortran)
set(CMAKE_NO_SYSTEM_FROM_IMPORTED YES)

option(BUILD_TESTS "${PROJECT_NAME} - Build tests" ON)
set(PACKAGE_CONFIG_DIR "lib/cmake" CACHE STRING "Directory for package config files (relative to CMAKE_INSTALL_PREFIX)")

# As of MUMPS 5 we can use Metis 5.1 :-)
option(USE_METIS "${PROJECT_NAME} - Use METIS 5" OFF)
option(USE_PARMETIS "${PROJECT_NAME} - Use ParMETIS" ON)
option(USE_SCOTCH "${PROJECT_NAME} - Use Scotch" OFF)
option(USE_PTSCOTCH "${PROJECT_NAME} - Use PT-Scotch" ON)
set(FORTRAN_MANGLING Add_ CACHE STRING "${PROJECT_NAME} - Fortran mangling scheme")
set(BUILD_PRECISION sdcz CACHE STRING "${PROJECT_NAME} - Build precisions (sdcz)")
# The code implements "UPPER" as mangling definition - we use UpCase as this
# is most common.
if (FORTRAN_MANGLING STREQUAL UpCase)
    set(FORTRAN_MANGLING UPPER)
endif()

find_package(LAPACK ${LAPACK_VERSION} REQUIRED)
find_package(SCALAPACK ${SCALAPACK_VERSION} REQUIRED)
find_package(MPI REQUIRED)

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

set(CMAKE_Fortran_MODULE_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/fortran_modules)

add_subdirectory(PORD)

# collect mumps_common sources
set(mumps-src-f
    src/ana_AMDMF.F
    src/ana_omp_m.F
    src/ana_orderings.F
    src/bcast_errors.F
    src/double_linked_list.F
    src/estim_flops.F
    src/fac_asm_build_sort_index_ELT_m.F
    src/fac_asm_build_sort_index_m.F
    src/fac_descband_data_m.F
    src/fac_future_niv2_mod.F
    src/fac_ibct_data_m.F
    src/fac_maprow_data_m.F
    src/front_data_mgt_m.F
    src/mumps_comm_ibcast.F
	src/mumps_l0_omp_m.F
	src/mumps_ooc_common.F
    src/mumps_ooc_common.F
    src/mumps_print_defined.F
    src/mumps_sol_es.F
    src/mumps_static_mapping.F
	src/mumps_type_size.F
    src/mumps_type2_blocking.F
    src/mumps_version.F
	src/omp_tps_common_m.F
	src/tools_common.F
	src/tools_common_mod.F
)
set(mumps-src-c
    src/mumps_common.c
	src/mumps_io.c
    src/mumps_io_basic.c
    src/mumps_io_thread.c
    src/mumps_io_err.c
	src/mumps_numa.c
    src/mumps_orderings.c
    src/mumps_size.c
)

set(mumps-src-arith
    ana_aux.F
    ana_aux_ELT.F
    ana_aux_par.F
    ana_dist_m.F
    ana_driver.F
    ana_LDLT_preprocess.F
    ana_lr.F
    ana_mtrans.F
    ana_reordertree.F
    arrowheads.F
    bcast_int.F
    end_driver.F
    fac_asm.F
    fac_asm_ELT.F
    fac_asm_master_m.F
    fac_asm_master_ELT_m.F
    fac_b.F
    fac_determinant.F
    fac_distrib_distentry.F
    fac_distrib_ELT.F
    fac_driver.F
    fac_front_LU_type1.F
    fac_front_LU_type2.F
    fac_front_LDLT_type1.F
    fac_front_LDLT_type2.F
    fac_front_aux.F
    fac_front_type2_aux.F
    fac_lastrtnelind.F
    fac_lr.F
    fac_mem_alloc_cb.F
    fac_mem_compress_cb.F
    fac_mem_free_block_cb.F
    fac_mem_stack_aux.F
    fac_mem_stack.F
    fac_omp_m.F
    fac_par_m.F
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
    fac_scalings_simScaleAbs.F
    fac_scalings_simScale_util.F
    fac_sol_pool.F
    fac_type3_symmetrize.F
    ini_defaults.F
    ini_driver.F
    lr_core.F
    mumps_comm_buffer.F
    mumps_driver.F
    mumps_f77.F
    mumps_iXamax.F
    mumps_load.F
    mumps_lr_data_m.F
    mumps_ooc_buffer.F
    mumps_ooc.F
    mumps_restart.F
    mumps_struc_def.F
    omp_tps_m.F
    ooc_panel_piv.F
    rank_revealing.F
    sol_aux.F
    sol_bwd_aux.F
    sol_bwd.F
    sol_c.F
    sol_driver.F
    sol_fwd_aux.F
    sol_fwd.F
    sol_matvec.F
    sol_root_parallel.F
    static_ptr_m.F
    tools.F
    type3_root.F
)

foreach(lang f c)
    # Need to split c and fortran sources for VS
    add_library(mumps_common_${lang} ${mumps-src-${lang}})
    target_link_libraries(mumps_common_${lang} PUBLIC scalapack pord)
    target_include_directories(mumps_common_${lang} PRIVATE include)
    target_compile_definitions(mumps_common_${lang} PRIVATE ${FORTRAN_MANGLING})
    if (USE_PTSCOTCH)
        target_compile_definitions(mumps_common_${lang} PRIVATE ptscotch)
        target_link_libraries(mumps_common_${lang} PUBLIC ptesmumps)
    elseif(USE_SCOTCH)
        target_compile_definitions(mumps_common_${lang} PRIVATE scotch)
        target_link_libraries(mumps_common_${lang} PUBLIC esmumps)
    endif()
    if (USE_PARMETIS)
        target_compile_definitions(mumps_common_${lang} PRIVATE parmetis)
        target_link_libraries(mumps_common_${lang} PUBLIC parmetis)
    elseif(USE_METIS)
        target_compile_definitions(mumps_common_${lang} PRIVATE metis)
        target_link_libraries(mumps_common_${lang} PUBLIC metis)
    endif()
endforeach()

# Interface library
add_library(mumps_common INTERFACE)
target_link_libraries(mumps_common INTERFACE mumps_common_f mumps_common_c)
#set_target_properties(mumps_common PROPERTIES OUTPUT_NAME mumps_common-${MUMPS_VERSION})

# Build objects for each precision and combine them into the later mumps target
set(PREC_LIBS )
foreach(prec s d c z)
    if(BUILD_PRECISION MATCHES [${prec}])
        message(STATUS "Building precision '${prec}'")
        # Compile sources for current arith
        set(mumps-src-arith-${prec} )
        foreach(entry ${mumps-src-arith})
            list(APPEND mumps-src-arith-${prec} src/${prec}${entry})
        endforeach()
        
        # Arithmetic-dependent mumps f libraries
        add_library(${prec}mumps ${mumps-src-arith-${prec}})
        set_target_properties(${prec}mumps PROPERTIES
            OUTPUT_NAME ${prec}mumps-${MUMPS_VERSION})
        target_include_directories(${prec}mumps PRIVATE include)
        target_link_libraries(${prec}mumps PUBLIC mumps_common)
        target_compile_definitions(${prec}mumps PRIVATE
            MUMPS_ARITH=MUMPS_ARITH_${prec}
        )
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
        
        # Arithmetic-dependent mumps c libraries
        add_library(${prec}mumps_c src/mumps_c.c)
        target_link_libraries(${prec}mumps_c PUBLIC mumps_common ${prec}mumps)
        target_include_directories(${prec}mumps_c PRIVATE include)
        target_compile_definitions(${prec}mumps_c PRIVATE
            MUMPS_ARITH=MUMPS_ARITH_${prec}
            ${FORTRAN_MANGLING}
        )
        
        list(APPEND PREC_LIBS ${prec}mumps ${prec}mumps_c)
        # Make sure the according header files are installed
        install(DIRECTORY include/ DESTINATION include
            FILES_MATCHING PATTERN "${prec}mu*.h")
    endif()
endforeach()

# Add global interface target for easy linking if the build precisions are not known.
add_library(mumps INTERFACE)
target_link_libraries(mumps INTERFACE mumps_common ${PREC_LIBS})

install(TARGETS mumps_common mumps_common_c mumps_common_f mumps ${PREC_LIBS} EXPORT mumps-config
    DESTINATION lib
    INCLUDES DESTINATION include/mumps)
# common headers
install(FILES include/mumps_c_types.h include/mumps_compat.h DESTINATION include/mumps)
install(EXPORT mumps-config DESTINATION ${PACKAGE_CONFIG_DIR})

# Fortran modules
install(DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/fortran_modules/ DESTINATION include/mumps)

include(CMakePackageConfigHelpers)
WRITE_BASIC_PACKAGE_VERSION_FILE(${CMAKE_CURRENT_BINARY_DIR}/mumps-config-version.cmake COMPATIBILITY AnyNewerVersion)
file(WRITE ${CMAKE_CURRENT_BINARY_DIR}/mumps-config-dependencies.cmake 
    "include(CMakeFindDependencyMacro)\r\n"
    	"set(${PROJECT_NAME}_IMPORT_PREFIX \${_IMPORT_PREFIX})\r\n"
)
file(APPEND ${CMAKE_CURRENT_BINARY_DIR}/mumps-config-dependencies.cmake "find_dependency(LAPACK)\r\n")
file(APPEND ${CMAKE_CURRENT_BINARY_DIR}/mumps-config-dependencies.cmake "find_dependency(SCALAPACK)\r\n")
if (USE_PARMETIS)
    file(APPEND ${CMAKE_CURRENT_BINARY_DIR}/mumps-config-dependencies.cmake "find_dependency(PARMETIS)\r\n")
elseif(USE_METIS)
    file(APPEND ${CMAKE_CURRENT_BINARY_DIR}/mumps-config-dependencies.cmake "find_dependency(METIS)\r\n")    
endif()
if (USE_PTSCOTCH)
    file(APPEND ${CMAKE_CURRENT_BINARY_DIR}/mumps-config-dependencies.cmake "find_dependency(PTSCOTCH)\r\n")
elseif(USE_SCOTCH)
    file(APPEND ${CMAKE_CURRENT_BINARY_DIR}/mumps-config-dependencies.cmake "find_dependency(SCOTCH)\r\n")    
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
