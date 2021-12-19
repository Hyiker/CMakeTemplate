cmake_minimum_required(VERSION 3.13)

set(LANG CXX C)
project(CMakeTemplate VERSION 0.0.1 LANGUAGES ${LANG})
set(CMAKE_CXX_STANDARD 20)

##################################################
# HELPERS TOOLS
##################################################
list(APPEND CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/cmake)
include(GitUtility)

set(GIT_HASH "")
get_git_hash(GIT_HASH)
message(STATUS "Git hash is ${GIT_HASH}")

set(GIT_BRANCH "")
get_git_branch(GIT_BRANCH)
message(STATUS "Git branch is ${GIT_BRANCH}")
configure_file(
  ${CMAKE_CURRENT_SOURCE_DIR}/helpers/git_helper.h.in
  ${CMAKE_CURRENT_SOURCE_DIR}/helpers/git_helper.h
  @ONLY
)

##################################################
# SOURCES AND HEADERS
##################################################
set(TARGET target_name)
set(SOURCES
    proj/main.cpp
)
set(HEADERS
   helpers/git_helper.h
)
add_executable(${TARGET} ${HEADERS} ${SOURCES})

##################################################
# THIRD PARTY LIBRARY
##################################################
set(LIBS_PUB)
set(LIBS_PRI)
# find_package(OpenMP)
# set(LIBS_PUB ${LIBS_PUB OpenMP::OpenMP_CXX})
target_link_libraries(${TARGET} PRIVATE ${LIBS_PRI} PUBLIC ${LIBS_PUB})

##################################################
# ASSETS PACKING
##################################################
set(ASSETS_SRC "${CMAKE_SOURCE_DIR}/assets")
set(ASSETS_DST "$<TARGET_FILE_DIR:${TARGET}>/assets")

if(WIN32)
    set(ASSETS_CMD copy_directory)
    set(ASSETS_MSG "Copy assets")
else()
    set(ASSETS_CMD create_symlink)
    set(ASSETS_MSG "Link assets")
endif()
add_custom_command(
    TARGET ${TARGET} POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E echo
    "${ASSETS_MSG}: ${ASSETS_SRC} to ${ASSETS_DST}"
)
add_custom_command(
    TARGET ${TARGET} POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E ${ASSETS_CMD}
    "${ASSETS_SRC}" "${ASSETS_DST}"
)

##################################################
# UNIT TESTS
##################################################
string(TOUPPER ${CMAKE_BUILD_TYPE} CBT)
if(CBT MATCHES DEBUG)
    message(STATUS "Building unit tests")
    set(TEST_TARGET ${TARGET}_test)
    set(TEST_SOURCES
        tests/main.cpp
    )
    set(TEST_HEADERS
       tests/one_add_one_test.hpp
       helpers/git_helper.h
    )
    add_executable(${TEST_TARGET} ${TEST_HEADERS} ${TEST_SOURCES})
endif()
