cmake_minimum_required(VERSION 3.12)

message(STATUS
        "------------------------------------------------------------\n"
        "--\n"
        "--             Configuring Acrotensor Unit Tests\n"
        "--\n"
        "-- ------------------------------------------------------------"
        )


project(tests CXX)
file(GLOB_RECURSE SOURCE_TEST "*.cpp")
add_custom_target(all-tests)

add_executable(acrotensor_unit_tests ${SOURCE_TEST})
target_link_libraries(acrotensor_unit_tests PRIVATE acrotensor_obj)
target_include_directories(acrotensor_unit_tests PRIVATE ${CMAKE_CURRENT_LIST_DIR})
target_include_directories(acrotensor_unit_tests PRIVATE ${CMAKE_SOURCE_DIR} ${CMAKE_SOURCE_DIR}/exec ${CMAKE_SOURCE_DIR}/kernel ${CMAKE_SOURCE_DIR}/ops ${CMAKE_SOURCE_DIR}/tensor ${CMAKE_SOURCE_DIR}/util)
add_test(NAME acrotensor_unit_tests WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR} COMMAND acrotensor_unit_tests)
if (MSVC)
    set_target_properties(acrotensor_unit_tests PROPERTIES LINK_FLAGS "/ignore:4099")
endif()
add_dependencies(all-tests acrotensor_unit_tests)

add_custom_command(
        TARGET all-tests
        POST_BUILD
        COMMENT "Running Tests"
        WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
        DEPENDS acrotensor_obj
        COMMAND ${CMAKE_CTEST_COMMAND} -C $<CONFIG> --output-on-failure)



