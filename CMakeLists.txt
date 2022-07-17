cmake_minimum_required(VERSION 3.14)
include(ExternalProject)

project(json-buffet LANGUAGES CXX)



set(ARROW_VERSION "apache-arrow-0.17.0")

if (NOT "$ENV{PARQUET_ARROW_VERSION}" STREQUAL "")
    set(ARROW_VERSION "$ENV{PARQUET_ARROW_VERSION}")
endif()

message(STATUS "Using Apache Arrow version: ${ARROW_VERSION}")

set(ARROW_URL "https://github.com/apache/arrow/archive/${ARROW_VERSION}.tar.gz")

set(ARROW_CMAKE_ARGS
    #Arrow dependencies
    -DARROW_WITH_LZ4=OFF
    -DARROW_WITH_ZSTD=OFF
    -DARROW_WITH_BROTLI=OFF
    -DARROW_WITH_SNAPPY=OFF
    -DARROW_WITH_ZLIB=OFF

    #Build settings
    -DARROW_BUILD_STATIC=ON
    -DARROW_BUILD_SHARED=OFF
    -DARROW_BOOST_USE_SHARED=ON
    -DARROW_BUILD_TESTS=OFF
    -DARROW_TEST_MEMCHECK=OFF
    -DARROW_BUILD_BENCHMARKS=OFF

    #Arrow modules
    -DARROW_IPC=ON
    -DARROW_COMPUTE=OFF
    -DARROW_GPU=OFF
    -DARROW_JEMALLOC=OFF
    -DARROW_BOOST_VENDORED=OFF
    -DARROW_PYTHON=OFF
    -DCMAKE_INSTALL_PREFIX=/

)

if (${ARROW_VERSION} STREQUAL "apache-arrow-0.15.0")
  # Keep ARROW_HDFS=ON to workaround arrow-0.9 bug that disables
  # boost_regex. See https://issues.apache.org/jira/browse/ARROW-2903
else ()
  set(ARROW_CMAKE_ARGS ${ARROW_CMAKE_ARGS} -DARROW_HDFS=OFF -DARROW_PARQUET=ON -DARROW_FILESYSTEM=ON -DARROW_DATASET=ON -DARROW_IPC=ON -DARROW_JSON=ON -DARROW_CSV=ON)
endif()

ExternalProject_Add(arrow
    URL                ${ARROW_URL}
    CONFIGURE_COMMAND "${CMAKE_COMMAND}" -G "${CMAKE_GENERATOR}" ${ARROW_CMAKE_ARGS} "${CMAKE_CURRENT_BINARY_DIR}/arrow-prefix/src/arrow/cpp/"
    INSTALL_COMMAND   make DESTDIR=${CMAKE_CURRENT_BINARY_DIR}/arrow-prefix/src/arrow-install install
)


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


set(INCLUDE_DIR ${CMAKE_CURRENT_BINARY_DIR}/arrow-prefix/src/arrow-install/usr/include/)
set(ARROW_INCLUDE ${INCLUDE_DIR}/arrow )
set(PARQUET_INCLUDE ${INCLUDE_DIR}/parquet )
set(ARROW_LIB ${CMAKE_CURRENT_BINARY_DIR}/arrow-prefix/src/arrow-install/usr/lib )

message(STATUS "ARROW_INC " ${INCLUDE_DIR})
add_executable(json-chunker-pq json-chunker-pq.cpp ${JSON_BUFFET_SRCS} )
target_include_directories(json-chunker-pq PRIVATE thirdparty/rapidjson/include/ ${INCLUDE_DIR})
target_compile_definitions(json-chunker-pq PRIVATE -DRAPIDJSON_HAS_STDSTRING=1 -DRAPIDJSON_SSE42=1)
target_link_libraries(json-chunker-pq ${ARROW_LIB}/libarrow.a ${ARROW_LIB}/libparquet.a)