cmake_minimum_required(VERSION 3.0)
project(libimobiledevice)


set(CMAKE_CXX_STANDARD 20)

if(CMAKE_SYSTEM_NAME STREQUAL "Windows")
    add_definitions(-DHAVE_STRNDUP -DHAVE_STPNCPY -DHAVE_MEMCPY -DHAVE_VASPRINTF -DHAVE_ASPRINTF -DHAVE_OPENSSL -DOPENSSL_SHA -DOPENSSL -DSTDC_HEADERS  -DPACKAGE_VERSION="v1.3" -DPACKAGE_STRING="v1.3")
elseif(CMAKE_SYSTEM_NAME STREQUAL "Linux")
    add_definitions(-DHAVE_STRNDUP -DHAVE_STPNCPY -DHAVE_GETIFADDRS -DHAVE_MEMCPY -DHAVE_VASPRINTF -DHAVE_ASPRINTF -DHAVE_OPENSSL -DOPENSSL_SHA -DOPENSSL -DSTDC_HEADERS -DHAVE_UNISTD_H -DHAVE_SYS_TIME_H -DPACKAGE_VERSION="v1.3" -DPACKAGE_STRING="v1.3")
elseif(CMAKE_SYSTEM_NAME STREQUAL "Darwin")
    add_definitions(-DHAVE_STRNDUP -DHAVE_STPNCPY -DHAVE_GETIFADDRS -DHAVE_MEMCPY -DHAVE_VASPRINTF -DHAVE_ASPRINTF -DHAVE_OPENSSL -DOPENSSL_SHA  -DOPENSSL -DSTDC_HEADERS -DHAVE_UNISTD_H -DHAVE_SYS_TIME_H -DPACKAGE_VERSION="v1.3" -DPACKAGE_STRING="v1.3")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -stdlib=libc++ -x objective-c++ ")
    # Execute ./configure --with-openssl
    execute_process(
        COMMAND ${CMAKE_CURRENT_SOURCE_DIR}/configure --with-openssl
        WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
        RESULT_VARIABLE CONFIGURE_RESULT
    )

    # Check the result of ./configure
    if(NOT CONFIGURE_RESULT EQUAL "0")
        message(FATAL_ERROR "Configure failed with error code ${CONFIGURE_RESULT}")
    endif()
    
    # Find OpenSSL package
    find_package(OpenSSL REQUIRED)
endif()




# Execute autogen.sh
# execute_process(
#     COMMAND ${CMAKE_CURRENT_SOURCE_DIR}/autogen.sh
#     WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
#     RESULT_VARIABLE AUTOREGEN_RESULT
# )

# # Check the result of autogen.sh
# if(NOT AUTOREGEN_RESULT EQUAL "0")
#     message(FATAL_ERROR "autogen.sh failed with error code ${AUTOREGEN_RESULT}")
# endif()

add_subdirectory(${CMAKE_CURRENT_SOURCE_DIR}/../libusbmuxd ${CMAKE_CURRENT_SOURCE_DIR}/../libusbmuxd/build)

# Collect all .cpp files recursively
file(GLOB_RECURSE LIBIMOBILEDEVICE_SOURCES  src/*.c src/*.h common/*.c common/*.h 3rd_party/ed25519/*.c 3rd_party/ed25519/*.h 3rd_party/libsrp6a-sha512/*.c 3rd_party/libsrp6a-sha512/*.h)

add_library(libimobiledevice SHARED ${LIBIMOBILEDEVICE_SOURCES})

target_include_directories(libimobiledevice PRIVATE . include src common 3rd_party/ed25519 3rd_party/libsrp6a-sha512 ../libplist/include ../libimobiledevice-glue/include)

target_link_libraries(libimobiledevice PRIVATE libusbmuxd libimobiledevice-glue libplist OpenSSL::SSL OpenSSL::Crypto)

if(CMAKE_SYSTEM_NAME STREQUAL "Darwin")
target_include_directories(libimobiledevice PRIVATE /usr/local/include)
endif()