cmake_minimum_required(VERSION 3.10)
project(git-crypt)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

if(MSVC)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /W4 /permissive- /EHsc")
else()
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -pedantic -Wno-long-long -O2")
endif()

if(WIN32)
    add_definitions(-DWIN32)
endif()

# Set installation directories
set(PREFIX "/usr/local" CACHE STRING "Installation prefix")
set(BINDIR "${PREFIX}/bin" CACHE STRING "Binary directory")
set(MANDIR "${PREFIX}/share/man" CACHE STRING "Man page directory")

# Source files
set(SOURCES
    git-crypt.cpp
    commands.cpp
    crypto.cpp
    gpg.cpp
    key.cpp
    parse_options.cpp
    fhstream.cpp
    crypto-openssl-10.cpp
    crypto-openssl-11.cpp
	util.cpp
	coprocess.cpp
)

if(WIN32)
    list(REMOVE_ITEM SOURCES util-unix.cpp coprocess-unix.cpp)
else()
    list(REMOVE_ITEM SOURCES util-win32.cpp coprocess-win32.cpp)
endif()

# Add executable
add_executable(git-crypt ${SOURCES})

# Link libraries
find_package(OpenSSL REQUIRED)
target_link_libraries(git-crypt OpenSSL::Crypto)

# Installation
install(TARGETS git-crypt DESTINATION ${BINDIR})

option(ENABLE_MAN "Enable man page installation" OFF)
if(ENABLE_MAN AND NOT WIN32)
    find_program(XSLTPROC xsltproc)
    if(XSLTPROC)
        set(DOCBOOK_XSL "http://docbook.sourceforge.net/release/xsl/current/manpages/docbook.xsl")
        add_custom_command(
            OUTPUT man/man1/git-crypt.1
            COMMAND ${XSLTPROC} --param man.output.in.separate.dir 1 
                                 --stringparam man.output.base.dir man/ 
                                 --param man.output.subdirs.enabled 1 
                                 --param man.authors.section.enabled 1 
                                 ${DOCBOOK_XSL} man/git-crypt.xml
            DEPENDS man/git-crypt.xml
        )
        add_custom_target(build-man DEPENDS man/man1/git-crypt.1)
        install(FILES man/man1/git-crypt.1 DESTINATION ${MANDIR}/man1)
    endif()
endif()

# Clean target
add_custom_target(clean-bin COMMAND ${CMAKE_COMMAND} -E remove git-crypt ${CMAKE_CURRENT_BINARY_DIR}/*.o)
if(ENABLE_MAN AND NOT WIN32)
    add_custom_target(clean-man COMMAND ${CMAKE_COMMAND} -E remove man/man1/git-crypt.1)
endif()
#add_custom_target(clean ALL DEPENDS clean-bin clean-man)
