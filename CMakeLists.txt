cmake_minimum_required(VERSION 3.21)

project(jnihook LANGUAGES C CXX)

set (CMAKE_CXX_STANDARD 23)

# Используем встроенный поиск Java
find_package(Java REQUIRED)
find_package(JNI REQUIRED)

option(JNIHOOK_BUILD_TESTS "Enable building of tests" OFF)

set(JNIHOOK_DIR "${PROJECT_SOURCE_DIR}/src")
set(JNIHOOK_INC "${PROJECT_SOURCE_DIR}/include")
file(GLOB_RECURSE JNIHOOK_SRC "${JNIHOOK_DIR}/*.cpp")

add_library(jnihook STATIC ${JNIHOOK_SRC})
target_include_directories(jnihook PUBLIC ${JNIHOOK_INC} ${JNI_INCLUDE_DIRS})
target_link_libraries(jnihook PUBLIC ${JNI_LIBRARIES})
set_target_properties(jnihook PROPERTIES POSITION_INDEPENDENT_CODE True)

if(JNIHOOK_BUILD_TESTS)
	# Build Java classes
	file(COPY "${PROJECT_SOURCE_DIR}/tests/dummy" DESTINATION .)
	execute_process(COMMAND "${Java_JAVAC_EXECUTABLE}" "dummy/Dummy.java")

	# Build library to inject
	set(TESTS_DIR "${PROJECT_SOURCE_DIR}/tests")
	set(TESTS_SRC "${TESTS_DIR}/test.cpp")
	add_library(test SHARED ${TESTS_SRC})
	target_link_libraries(test PRIVATE jnihook)
	set_target_properties(test PROPERTIES POSITION_INDEPENDENT_CODE True)
endif()
