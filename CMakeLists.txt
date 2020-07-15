
cmake_minimum_required (VERSION 3.1)
#set_target_properties(xbeep PROPERTIES PUBLIC_HEADER include/xbeep.h)
set(LIBMAJ 3)
set(LIBMIN 0)
set(LIBREV 12)
set(LIBFULLREV ${LIBMAJ}.${LIBMIN}.${LIBREV})
set(MODELIST  xbee1 xbee2 xbee3 xbee5 xbee6b xbeeZB net debug)
set(CMAKE_C_FLAGS "-fvisibility=hidden -Wno-variadic-macros -Wstrict-prototypes -MMD")
set(CMAKE_C_COMPILER "gcc")
set(CMAKE_CXX_FLAGS "-fvisibility=hidden -Wno-variadic-macros -MMD")
set(CMAKE_CXX_COMPILER "g++")
file(GLOB xbee_srcs *.c modes/*.c modes/*/*.c)
list(REMOVE_ITEM xbee_srcs 
	${CMAKE_SOURCE_DIR}/xsys_darwin.c
	${CMAKE_SOURCE_DIR}/xsys_linux.c)
list(REVERSE MODELIST)
set(CMODELIST NULL)
foreach(MODE IN LISTS MODELIST)
	file(GLOB mode_srcs modes/${MODE}/*.c)
	list(APPEND xbee_srcs ${mode_srcs})
	set(CMODELIST "&mode_${MODE}, ${CMODELIST}")
endforeach(MODE)
execute_process(
	COMMAND git log -1 --format="%H"
  	WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
	OUTPUT_VARIABLE COMMIT
  	OUTPUT_STRIP_TRAILING_WHITESPACE)
execute_process(
	COMMAND git log -1 --format="%cn <%ce>"
  	WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
	OUTPUT_VARIABLE COMMITTER
  	OUTPUT_STRIP_TRAILING_WHITESPACE)
execute_process(
	COMMAND date
  	WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
	OUTPUT_VARIABLE BUILDTIME
  	OUTPUT_STRIP_TRAILING_WHITESPACE)
set_property(SOURCE ver.c PROPERTY COMPILE_DEFINITIONS
	LIB_REVISION="${LIBFULLREV}" LIB_COMMIT=${COMMIT}
	LIB_COMMITTER=${COMMITTER} LIB_BUILDTIME="${BUILDTIME}")
set_property(SOURCE mode.c PROPERTY COMPILE_DEFINITIONS MODELIST=${CMODELIST})
include_directories(include)
add_library(xbee-static STATIC ${xbee_srcs})
add_library(xbee-shared SHARED ${xbee_srcs})
target_link_libraries(xbee-static pthread rt)
target_link_libraries(xbee-shared pthread rt)
set_property(TARGET xbee-static xbee-shared PROPERTY PUBLIC_HEADER include/xbee.h)
set_property(TARGET xbee-static xbee-shared PROPERTY OUTPUT_NAME xbee)
add_library(xbeep-static STATIC xbeep.cpp)
target_link_libraries(xbeep-static xbee-static)
add_library(xbeep-shared SHARED xbeep.cpp)
target_link_libraries(xbeep-shared xbee-shared)
set_property(TARGET xbeep-static xbeep-shared PROPERTY PUBLIC_HEADER include/xbeep.h)
set_property(TARGET xbeep-static xbeep-shared PROPERTY OUTPUT_NAME xbeep)
set_property(TARGET xbee-static xbee-shared xbeep-static xbeep-shared PROPERTY VERSION ${LIBFULLREV})
set_property(TARGET xbee-static xbee-shared xbeep-static xbeep-shared PROPERTY COMPILE_DEFINITIONS
	XBEE_LOG_RX_DEFAULT_OFF
	XBEE_LOG_TX_DEFAULT_OFF
	XBEE_FRAME_TIMEOUT_ENABLED
	LIBXBEE_BUILD)
SET(GCC_COVERAGE_LINK_FLAGS    "-r")
