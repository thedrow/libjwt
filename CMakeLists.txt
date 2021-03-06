project(libjwt C)
cmake_minimum_required(VERSION 2.8)

set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wall -Ofast -Wextra -pedantic -fPIE")

set(LIBRARY_NAME jwt)

set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR}/lib)# Static lib
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR}/lib)# Shared lib
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR}/bin)# executables

#Library used
set(LIBS ssl crypto jansson check m pthread rt subunit)

add_definitions(-DUSE_CMAKE)

include_directories(include)

add_library(${LIBRARY_NAME} SHARED libjwt/jwt.c)
add_library(${LIBRARY_NAME}_static STATIC libjwt/jwt.c)
set_target_properties(${LIBRARY_NAME}_static PROPERTIES OUTPUT_NAME ${LIBRARY_NAME})

target_link_libraries(${LIBRARY_NAME} ${LIBS})

install(TARGETS ${LIBRARY_NAME} ${LIBRARY_NAME}_static
        LIBRARY DESTINATION /usr/lib
        ARCHIVE DESTINATION /usr/lib
        OPTIONAL)

install(FILES include/jwt.h DESTINATION "/usr/include/jwt")

enable_testing()
add_custom_target(tests COMMAND ${CMAKE_CTEST_COMMAND} OPTIONAL)

function(InitialiseTest test_name file)
    add_executable(${test_name} ${file})
    target_link_libraries(${test_name} ${LIBS} ${LIBRARY_NAME})
    add_test(NAME ${test_name} COMMAND ${test_name} OPTIONAL)
    add_dependencies(tests ${test_name})
endfunction()

InitialiseTest(jwt_dump tests/jwt_dump.c)
InitialiseTest(jwt_encode tests/jwt_encode.c)
InitialiseTest(jwt_grant tests/jwt_grant.c)
InitialiseTest(jwt_new tests/jwt_new.c)
