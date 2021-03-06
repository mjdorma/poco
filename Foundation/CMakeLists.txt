set(LIBNAME "PocoFoundation")

# Sources
file(GLOB SRCS_G "src/*.cpp")
POCO_SOURCES_AUTO( SRCS ${SRCS_G})

# Headers
file(GLOB_RECURSE HDRS_G "include/*.h" )
POCO_HEADERS_AUTO( SRCS ${HDRS_G})

# Platform Specific
POCO_SOURCES_AUTO_PLAT( SRCS OPENVMS src/OpcomChannel.cpp )
POCO_HEADERS_AUTO( SRCS include/Poco/OpcomChannel.h )

POCO_SOURCES_AUTO_PLAT( SRCS UNIX src/SyslogChannel.cpp )
POCO_HEADERS_AUTO( SRCS include/Poco/SyslogChannel.h )

POCO_SOURCES_AUTO_PLAT( SRCS WIN32
    src/WindowsConsoleChannel.cpp
    src/EventLogChannel.cpp
    )

POCO_HEADERS_AUTO( SRCS
    include/Poco/WindowsConsoleChannel.h
    include/Poco/EventLogChannel.h
    )

# Messages
POCO_MESSAGES( SRCS Logging src/pocomsg.mc)

# If POCO_UNBUNDLED is enabled we try to find the required packages
# The configuration will fail if the packages are not found
if (POCO_UNBUNDLED)
    find_package(PCRE REQUIRED)
    set(SYSLIBS ${SYSLIBS} ${PCRE_LIBRARIES})
    include_directories(${PCRE_INCLUDE_DIRS})

    #HACK: Unicode.cpp requires functions from these files. The can't be taken from the library
    POCO_SOURCES( SRCS RegExp
        src/pcre_ucd.c
        src/pcre_tables.c
    )

    find_package(ZLIB REQUIRED)
    set(SYSLIBS ${SYSLIBS} ${ZLIB_LIBRARIES})
    include_directories(${ZLIB_INCLUDE_DIRS})
else()
    # pcre
    POCO_SOURCES( SRCS pcre
        src/pcre_chartables.c
        src/pcre_compile.c
        src/pcre_exec.c
        src/pcre_fullinfo.c
        src/pcre_globals.c
        src/pcre_maketables.c
        src/pcre_newline.c
        src/pcre_ord2utf8.c
        src/pcre_study.c
        src/pcre_tables.c
        src/pcre_try_flipped.c
        src/pcre_ucd.c
        src/pcre_valid_utf8.c
        src/pcre_xclass.c
    )

    # zlib
    POCO_HEADERS( SRCS zlib
        include/Poco/zconf.h
        include/Poco/zlib.h
        )

    POCO_SOURCES( SRCS zlib
        src/adler32.c
        src/compress.c
        src/crc32.c
        src/deflate.c
        src/infback.c
        src/inffast.c
        src/inflate.c
        src/inftrees.c
        src/trees.c
        src/zutil.c
    )
endif (POCO_UNBUNDLED)

if(CMAKE_SYSTEM MATCHES "Windows")
    add_definitions( -DPCRE_STATIC)
    set(SYSLIBS ${SYSLIBS} iphlpapi)
else (CMAKE_SYSTEM MATCHES "Windows")
    add_definitions( -DPCRE_STATIC)
endif(CMAKE_SYSTEM MATCHES "Windows")

add_library( ${LIBNAME} ${LIB_MODE} ${SRCS}) 
set_target_properties( ${LIBNAME} 
    PROPERTIES
    VERSION ${SHARED_LIBRARY_VERSION} SOVERSION ${SHARED_LIBRARY_VERSION}
    DEFINE_SYMBOL Foundation_EXPORTS)
if (${CMAKE_CXX_COMPILER_ID} MATCHES "SunPro")
    set_target_properties( ${LIBNAME} PROPERTIES LINK_FLAGS "-library=stlport4")
endif (${CMAKE_CXX_COMPILER_ID} MATCHES "SunPro")
target_link_libraries( ${LIBNAME} ${SYSLIBS})

install(
    DIRECTORY include/Poco
    DESTINATION include
    PATTERN ".svn" EXCLUDE
    )

install(
    TARGETS ${LIBNAME}
    LIBRARY DESTINATION lib${LIB_SUFFIX}
    ARCHIVE DESTINATION lib${LIB_SUFFIX}
    RUNTIME DESTINATION bin
    )

if (ENABLE_TESTS)
    add_subdirectory( samples )
    add_subdirectory( testsuite )
endif ()

