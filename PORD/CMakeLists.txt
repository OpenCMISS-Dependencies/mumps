file(GLOB pord_src lib/*.c)
add_library(pord ${pord_src})
set_target_properties(pord PROPERTIES OUTPUT_NAME pord-${MUMPS_VERSION})
target_include_directories(pord PRIVATE include)
install(TARGETS pord EXPORT mumps-config
    DESTINATION lib
    INCLUDES DESTINATION include/mumps) 