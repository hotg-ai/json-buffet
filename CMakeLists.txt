cmake_minimum_required(VERSION 3.14)
include(ExternalProject)

set(CMAKE_TOOLCHAIN_FILE "${CMAKE_CURRENT_SOURCE_DIR}/thirdparty/vcpkg/scripts/buildsystems/vcpkg.cmake"
  CACHE STRING "Vcpkg toolchain file")

project(json-buffet LANGUAGES CXX)

#set(DUCKDB_CMAKE_ARGS -DEXTENSION_STATIC_BUILD=TRUE -DBUILD_PARQUET_EXTENSION=TRUE -DBUILD_JSON_EXTENSION=TRUE)

#ExternalProject_Add(duckdb
#    URL                https://github.com/duckdb/duckdb/archive/refs/tags/v0.4.0.tar.gz
#    CONFIGURE_COMMAND "${CMAKE_COMMAND}" -G "${CMAKE_GENERATOR}"  ${DUCKDB_CMAKE_ARGS} "${CMAKE_CURRENT_BINARY_DIR}/duckdb-prefix/src/duckdb/"
#    INSTALL_COMMAND   make DESTDIR=${CMAKE_CURRENT_BINARY_DIR}/duckdb-prefix/src/duckdb-install install
#)


set(CMAKE_INCLUDE_CURRENT_DIR ON)

set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

set(JSON_BUFFET_SRCS jsonbuffet.cpp utils.cpp jsonbuffet.h utils.h)

set(CMAKE_BUILD_TYPE Release)
add_executable(json-buffet main.cpp ${JSON_BUFFET_SRCS})

target_include_directories(json-buffet PRIVATE thirdparty/rapidjson/include/)
target_compile_definitions(json-buffet PRIVATE -DRAPIDJSON_HAS_STDSTRING=1 -DRAPIDJSON_SSE42=1)

add_executable(json-chunker json-chunker.cpp ${JSON_BUFFET_SRCS})
target_include_directories(json-chunker PRIVATE thirdparty/rapidjson/include/)
target_compile_definitions(json-chunker PRIVATE -DRAPIDJSON_HAS_STDSTRING=1 -DRAPIDJSON_SSE42=1)

add_executable(json-indexer json-indexer.cpp ${JSON_BUFFET_SRCS})
target_include_directories(json-indexer PRIVATE thirdparty/rapidjson/include/)
target_compile_definitions(json-indexer PRIVATE -DRAPIDJSON_HAS_STDSTRING=1 -DRAPIDJSON_SSE42=1)

find_package(Arrow CONFIG REQUIRED)
find_package(Parquet CONFIG REQUIRED)

add_executable(json-chunker-pq json-chunker-pq.cpp ${JSON_BUFFET_SRCS} )
target_include_directories(json-chunker-pq PRIVATE thirdparty/rapidjson/include/ ${INCLUDE_DIR})
target_compile_definitions(json-chunker-pq PRIVATE -DRAPIDJSON_HAS_STDSTRING=1 -DRAPIDJSON_SSE42=1)
target_link_libraries(json-chunker-pq ${ARROW_LIBRARIES} ${PARQUET_LIBRARIES})
