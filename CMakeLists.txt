cmake_minimum_required(VERSION 3.14)
project(example)

option(BUILD_METHOD "Build method" 0)

find_package(Protobuf REQUIRED)
message(STATUS "Protobuf_PROTOC_EXECUTABLE = ${Protobuf_PROTOC_EXECUTABLE}")
message(STATUS "Protobuf_INCLUDE_DIRS = ${Protobuf_INCLUDE_DIRS}")
message(STATUS "Protobuf_LIBRARIES = ${Protobuf_LIBRARIES}")

if(BUILD_METHOD STREQUAL 0)
    message(STATUS "Build method: 0")
elseif(BUILD_METHOD STREQUAL 1)
    # 方式一：使用protobuf_generate_cpp
    file(GLOB_RECURSE PROTO_FILES ${CMAKE_CURRENT_SOURCE_DIR}/proto/*.proto)
    message(STATUS "PROTO_FILES = ${PROTO_FILES}")
    set(ALL_PROTO_SRCS)
    set(ALL_PROTO_HDRS)
    foreach(proto_file ${PROTO_FILES})
        protobuf_generate_cpp(PROTO_SRCS PROTO_HDRS ${proto_file})
        message(STATUS "PROTO_SRCS = ${PROTO_SRCS}")
        message(STATUS "PROTO_HDRS = ${PROTO_HDRS}")
        list(APPEND ALL_PROTO_SRCS ${PROTO_SRCS})
        list(APPEND ALL_PROTO_HDRS ${PROTO_HDRS})
    endforeach()
    
    # 生成的文件在build目录下
    message(STATUS "ALL_PROTO_SRCS = ${ALL_PROTO_SRCS}")
    message(STATUS "ALL_PROTO_HDRS = ${ALL_PROTO_HDRS}")
    add_executable(
        write  ${CMAKE_CURRENT_SOURCE_DIR}/write.cpp ${ALL_PROTO_SRCS}
    )
    target_link_libraries(
        write ${Protobuf_LIBRARIES}
    )
    target_include_directories(
        write PUBLIC ${Protobuf_INCLUDE_DIRS} ${CMAKE_CURRENT_BINARY_DIR}
    )

    add_executable(
        read  ${CMAKE_CURRENT_SOURCE_DIR}/read.cpp ${ALL_PROTO_SRCS}
    )
    target_link_libraries(
        read ${Protobuf_LIBRARIES}
    )
    target_include_directories(
        read PUBLIC ${Protobuf_INCLUDE_DIRS} ${CMAKE_CURRENT_BINARY_DIR}
    )

elseif(BUILD_METHOD STREQUAL 2)
    # 方式二：使用protoc
    # protoc -I=../proto --cpp_out=../proto ../proto/addressbook.ptoto
elseif(BUILD_METHOD STREQUAL 3)
    # 方式三：使用add_custom_command
    file(GLOB_RECURSE PROTO_FILES ${CMAKE_CURRENT_SOURCE_DIR}/proto/*.proto)
    message(STATUS "PROTO_FILES = ${PROTO_FILES}")
    set(ALL_PROTO_SRCS)
    set(ALL_PROTO_HDRS)
    foreach(proto_file ${PROTO_FILES})
        get_filename_component(FILE_NAME ${proto_file} NAME_WE)
        set(OUTPUT_CXX_HEADER "${CMAKE_CURRENT_SOURCE_DIR}/proto/${FILE_NAME}.pb.h")
        set(OUTPUT_CXX_SRC "${CMAKE_CURRENT_SOURCE_DIR}/proto/${FILE_NAME}.pb.cc")
        list(APPEND ALL_PROTO_SRCS ${OUTPUT_CXX_SRC})
        list(APPEND ALL_PROTO_HDRS ${OUTPUT_CXX_HEADER})
    endforeach()
    message(STATUS "ALL_PROTO_SRCS = ${ALL_PROTO_SRCS}")
    message(STATUS "ALL_PROTO_HDRS = ${ALL_PROTO_HDRS}")

    add_custom_command(
        OUTPUT ${ALL_PROTO_SRCS} ${ALL_PROTO_HDRS}
        COMMAND ${Protobuf_PROTOC_EXECUTABLE}
        ARGS --cpp_out=${CMAKE_CURRENT_SOURCE_DIR}/proto --proto_path=${CMAKE_CURRENT_SOURCE_DIR}/proto ${PROTO_FILES}
    )

    add_executable(
        write  ${CMAKE_CURRENT_SOURCE_DIR}/write.cpp ${ALL_PROTO_SRCS}
    )
    target_link_libraries(
        write ${Protobuf_LIBRARIES}
    )
    target_include_directories(
        write PUBLIC ${Protobuf_INCLUDE_DIRS} ${CMAKE_CURRENT_SOURCE_DIR}/proto
    )

    add_executable(
        read  ${CMAKE_CURRENT_SOURCE_DIR}/read.cpp ${ALL_PROTO_SRCS}
    )
    target_link_libraries(
        read ${Protobuf_LIBRARIES}
    )
    target_include_directories(
        read PUBLIC ${Protobuf_INCLUDE_DIRS} ${CMAKE_CURRENT_SOURCE_DIR}/proto
    )
endif()


