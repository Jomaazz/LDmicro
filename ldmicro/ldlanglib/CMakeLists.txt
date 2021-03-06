cmake_minimum_required(VERSION 3.11)

project(ldlanglib LANGUAGES CXX)

if( NOT LDLANG )
    set(LDLANG "EN")
endif()

message(STATUS "ldlang lang : ${LDLANG}")

find_package(Perl REQUIRED)

set(LDLANG_SRC_LIST
    ../lang.h
    ../lang.cpp

    "${CMAKE_CURRENT_SOURCE_DIR}/obj/helptext.cpp"
    "${CMAKE_CURRENT_SOURCE_DIR}/obj/lang-tables.h"
    )

file(MAKE_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/obj)

execute_process(
    COMMAND ${PERL_EXECUTABLE} txt2c.pl
    WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/../
    OUTPUT_FILE      "${CMAKE_CURRENT_SOURCE_DIR}/obj/helptext.cpp"
    )

execute_process(
    COMMAND ${PERL_EXECUTABLE} lang-tables.pl
    WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/../
    OUTPUT_FILE      "${CMAKE_CURRENT_SOURCE_DIR}/obj/lang-tables.h"
    )

if(MSVC)
    string(APPEND CMAKE_CXX_FLAGS " /J")
    string(APPEND CMAKE_C_FLAGS " /J")

    if(CMAKE_CXX_FLAGS MATCHES "/W[0-4]")
        string(REGEX REPLACE "/W[0-4]" "/W4" CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS}")
    else()
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /W4")
    endif()

    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /wd4127")

elseif(CMAKE_COMPILER_IS_GNUCXX)
    string(APPEND CMAKE_CXX_FLAGS "-Wall -Wextra ")
    string(APPEND CMAKE_C_FLAGS "-Wall -Wextra ")
endif()

add_definitions(
    "-DISOLATION_AWARE_ENABLED"
    "-DWIN32_LEAN_AND_MEAN"
    "-D_CRT_SECURE_NO_WARNINGS"
    "-D_CRT_SECURE_NO_DEPRECATE"
    "-DNOMINMAX"
    )
INCLUDE_DIRECTORIES("../../common/win32" "../../common/gsl-lite/include" "../../common/stringarray" "../../common/utils")

if(${LDLANG} STREQUAL "ALL")
    foreach(lang_id ${LDLANG_ALL})
        if(${lang_id} STREQUAL "EN")
            add_library               (${PROJECT_NAME} ${LDLANG_SRC_LIST})
            target_compile_definitions(${PROJECT_NAME} PUBLIC -DLDLANG_${lang_id})
            target_include_directories(${PROJECT_NAME} PRIVATE ${CMAKE_CURRENT_SOURCE_DIR})
        else()
            string(TOLOWER ${lang_id} ldlang_lower)
            add_library               (${PROJECT_NAME}-${ldlang_lower} ${LDLANG_SRC_LIST})
            target_compile_definitions(${PROJECT_NAME}-${ldlang_lower} PUBLIC -DLDLANG_${lang_id})
            target_include_directories(${PROJECT_NAME}-${ldlang_lower} PRIVATE ${CMAKE_CURRENT_SOURCE_DIR})
        endif()
    endforeach()

else()

    if(${LDLANG} STREQUAL "EN")
        add_library          (${PROJECT_NAME} ${LDLANG_SRC_LIST})
        target_compile_definitions(${PROJECT_NAME} PUBLIC -DLDLANG_${LDLANG})
        target_include_directories(${PROJECT_NAME} PRIVATE ${CMAKE_CURRENT_SOURCE_DIR})
    elseif( NOT (${LDLANG} STREQUAL "ALL"))
        string(TOLOWER ${LDLANG} ldlang_lower)
        add_library               (${PROJECT_NAME}-${ldlang_lower} ${LDLANG_SRC_LIST})
        target_compile_definitions(${PROJECT_NAME}-${ldlang_lower} PUBLIC -DLDLANG_${LDLANG})
        target_include_directories(${PROJECT_NAME}-${ldlang_lower} PRIVATE ${CMAKE_CURRENT_SOURCE_DIR})
    endif()

endif()
