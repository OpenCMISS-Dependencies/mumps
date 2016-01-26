if (UNIX OR MINGW)
    INCLUDE( ProcessorCount )
    ProcessorCount(SYSTEM_PROCESSOR_COUNT)
    
    SET(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR})
    SET(TEST_FILE ${CMAKE_CURRENT_SOURCE_DIR}/input_simpletest_real)
    
    # Fortran double precision test
    add_executable(simpletest dsimpletest.F)
    target_include_directories(simpletest PRIVATE 
        ../include 
        ${MPI_C_INCLUDE_PATH} ${MPI_Fortran_INCLUDE_PATH}
    )
    target_link_libraries(simpletest PRIVATE 
        dmumps ${MPI_C_LIBRARIES} ${MPI_Fortran_LIBRARIES}
    )
    
    SET(simpletest_wrap ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/simpletest_wrap.sh)
    add_custom_target(simpletest_wrap COMMAND simpletest < ${TEST_FILE})
    file(WRITE ${simpletest_wrap} "#!/bin/sh\n${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/simpletest < ${TEST_FILE}")
    execute_process(COMMAND chmod +x ${simpletest_wrap})
    add_test(simple ${simpletest_wrap})
    add_test(simple_par ${MPIEXEC} 
        ${MPIEXEC_NUMPROC_FLAG} ${SYSTEM_PROCESSOR_COUNT}
        ${simpletest_wrap})
else()
    message(WARNING "Simple Fortran test not implemented on non-UNIX/non-shell environments")        
endif()

# C test
add_executable(ctest c_example.c)
target_compile_definitions(ctest PRIVATE ${FORTRAN_MANGLING})
target_include_directories(ctest PRIVATE ../include)
target_link_libraries(ctest PRIVATE mumps)
add_test(c_test ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/ctest)
add_test(c_test_par ${MPIEXEC} 
    ${MPIEXEC_NUMPROC_FLAG} ${SYSTEM_PROCESSOR_COUNT}
    ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/ctest)