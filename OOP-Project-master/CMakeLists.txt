cmake_minimum_required(VERSION 3.10)

project(OOP-project)
add_definitions(-Wall)
set(CMAKE_CXX_STANDARD 11)

#include project headers
include_directories(
        ${CMAKE_CURRENT_SOURCE_DIR}/utils
        ${CMAKE_CURRENT_SOURCE_DIR}/src
)

set(source_vo
        src/vo/People.cpp
        src/vo/Storage.cpp
        src/vo/Student.cpp
        src/vo/Subject.cpp
        src/vo/Teacher.cpp
        src/vo/Tutor.cpp
        src/vo/Message.cpp
        )
set(source_dao
        src/dao/PeopleDao.cpp
        src/dao/SubjectDao.cpp
        )
set(source_service
        src/service/TutorServices.cpp
        src/service/StudentServices.cpp
        src/service/PeopleServices.cpp
        src/service/SubjectServices.cpp
        src/service/TeacherServices.cpp
        src/service/PeopleCommunications.cpp
        )

set(source_test
        test/PeopleServices_test.cpp
        test/database_test.cpp
        test/PeopleDao_test.cpp
        test/SubjectDao_test.cpp
        test/SubjectServices_test.cpp
        utils/database_connection.cpp
        )

set(major_project
        ${source_test}
        ${source_service}
        ${source_dao}
        ${source_vo}
        )
add_executable(GTestSelf ${major_project})

set(input_test
        test/input_tests/input_test.cpp
        utils/database_connection.cpp
        )
set(input_test
        ${input_test}
        ${source_service}
        ${source_dao}
        ${source_vo}
        )
add_executable(GTestSelfInput ${input_test})

set(main_cpp
        ${source_service}
        ${source_vo}
        ${source_dao}
        src/main.cpp
        utils/database_connection.cpp
        )
add_executable(MainSelf ${main_cpp})

#automatic dependency management

if(NOT EXISTS "${CMAKE_BINARY_DIR}/conan.cmake")
    message(STATUS "Downloading conan.cmake from https://github.com/conan-io/cmake-conan")
    file(DOWNLOAD "https://raw.githubusercontent.com/conan-io/cmake-conan/v0.16.1/conan.cmake"
            "${CMAKE_BINARY_DIR}/conan.cmake"
            EXPECTED_HASH SHA256=396e16d0f5eabdc6a14afddbcfff62a54a7ee75c6da23f32f7a31bc85db23484
            TLS_VERIFY ON)
endif()

#including dependencies headers
include(${CMAKE_BINARY_DIR}/conan.cmake)
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/utils/sqlpp11-connector-mysql/installed/x64-osx/include)

#link dependencies libraries
conan_cmake_run(REQUIRES

#        openssl/1.0.2u
        mariadb-connector-c/3.1.12
        gtest/1.10.0
        sqlpp11/0.60
        libcurl/7.78.0

        BASIC_SETUP CMAKE_TARGETS
        BUILD missing)

target_link_libraries(GTestSelf
        CONAN_PKG::gtest
#        CONAN_PKG::openssl
        CONAN_PKG::mariadb-connector-c
        CONAN_PKG::sqlpp11
        )


target_link_libraries(GTestSelfInput
        CONAN_PKG::gtest
#        CONAN_PKG::openssl
        CONAN_PKG::mariadb-connector-c
        CONAN_PKG::sqlpp11
        )

target_link_libraries(MainSelf
        CONAN_PKG::gtest
#        CONAN_PKG::openssl
        CONAN_PKG::mariadb-connector-c
        CONAN_PKG::sqlpp11
        )

#use different sqlpp11-mysql-connector library depending platform

#https://stackoverflow.com/questions/9160335/os-specific-instructions-in-cmake-how-to

if(APPLE)
    target_link_libraries(GTestSelf
            ${CMAKE_CURRENT_SOURCE_DIR}/utils/sqlpp11-connector-mysql/installed/x64-osx/lib/libsqlpp-mysql.a
            )
    target_link_libraries(GTestSelfInput
            ${CMAKE_CURRENT_SOURCE_DIR}/utils/sqlpp11-connector-mysql/installed/x64-osx/lib/libsqlpp-mysql.a
            )
    target_link_libraries(MainSelf
            ${CMAKE_CURRENT_SOURCE_DIR}/utils/sqlpp11-connector-mysql/installed/x64-osx/lib/libsqlpp-mysql.a
            )
endif(APPLE)

if(UNIX AND NOT APPLE)
    target_link_libraries(GTestSelf
            ${CMAKE_CURRENT_SOURCE_DIR}/utils/sqlpp11-connector-mysql/installed/x64-linux/lib/libsqlpp-mysql.a
            )
    target_link_libraries(GTestSelfInput
            ${CMAKE_CURRENT_SOURCE_DIR}/utils/sqlpp11-connector-mysql/installed/x64-linux/lib/libsqlpp-mysql.a
            )
    target_link_libraries(MainSelf
            ${CMAKE_CURRENT_SOURCE_DIR}/utils/sqlpp11-connector-mysql/installed/x64-linux/lib/libsqlpp-mysql.a
            )
    set_target_properties(GTestSelf
            PROPERTIES
            COMPILE_FLAGS "-ftest-coverage -fprofile-arcs"
            LINK_FLAGS    "-lgcov --coverage"
            )
    set_target_properties(GTestSelfInput
            PROPERTIES
            COMPILE_FLAGS "-ftest-coverage -fprofile-arcs"
            LINK_FLAGS    "-lgcov --coverage"
            )
    set_target_properties(MainSelf
            PROPERTIES
            COMPILE_FLAGS "-ftest-coverage -fprofile-arcs"
            LINK_FLAGS    "-lgcov --coverage"
            )
endif(UNIX AND NOT APPLE)

