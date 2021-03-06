cmake_minimum_required(VERSION 3.5)

project(yasmine_model)

set(CMAKE_VERBOSE_MAKEFILE, 1)

if(SX_RAPIDJSON)
	message("SX_RAPIDJSON is set by user.")
else()
	message("SX_RAPIDJSON is not set by user. Setting default value.")
	if("${CMAKE_CXX_COMPILER_ID}" STREQUAL "GNU" OR "${CMAKE_CXX_COMPILER_ID}" STREQUAL "Clang")
		set(SX_RAPIDJSON "/usr/include/rapidjson")
	elseif("${CMAKE_CXX_COMPILER_ID}" STREQUAL "MSVC")
		set(SX_RAPIDJSON "C:\Program Files\rapidjson")
	endif()
endif()
message(STATUS ${SX_RAPIDJSON})

file(GLOB yasmine_model_SRC "include/*.hpp" "source/*.cpp")

set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)

if("${CMAKE_CXX_COMPILER_ID}" STREQUAL "GNU" OR "${CMAKE_CXX_COMPILER_ID}" STREQUAL "Clang")
	if( "${CMAKE_CXX_FLAGS}" STREQUAL "-m64" )
		set(extra_flags "${extra_flags} -m64")
		message(STATUS "setting platform x64")
	elseif("${CMAKE_CXX_FLAGS}" STREQUAL "-m32")
		set(extra_flags "${extra_flags} -m32")
		message(STATUS "setting platform x86")
	endif()	
	

	if("${CPP_VERSION}" STREQUAL "03")		
		set(extra_flags "${extra_flags} -std=c++03 -Wall -Wpedantic -g -Wno-unknown-pragmas -D \"SX_CPP03_BOOST\" -D \"SX_NO_VARIADIC_MACRO\"")
	elseif("${CPP_VERSION}" STREQUAL "14")
		set(extra_flags "${extra_flags} -std=c++14 -Wall -Wpedantic -g")
	else()
		set(extra_flags "${extra_flags} -std=c++11 -Wall -Wpedantic -g -D \"SX_NO_STD_MAKE_UNIQUE\"")
	endif()

endif()


if( "${CMAKE_CXX_COMPILER_ID}" STREQUAL "MSVC" )
	# for release
	if( "${CMAKE_BUILD_TYPE}" STREQUAL "Release" )	
		#Creates fast code.
		set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} /O2")
		set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} /PDB-")
		
		#Controls LINK optimizations.
		set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} /OPT:REF /OPT:ICF")
		
	# for debug
	elseif( "${CMAKE_BUILD_TYPE}" STREQUAL "Debug" )	
		#Creates a program database (PDB) file.
		set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} /PDB")
	endif()
endif()


set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${extra_flags}")

include_directories(${SX_RAPIDJSON})
include_directories("./../yasmine_model/include")
include_directories("./../externals/essentials/include")

if("${CMAKE_CXX_COMPILER_ID}" STREQUAL "GNU" OR "${CMAKE_CXX_COMPILER_ID}" STREQUAL "Clang")
	add_library(libyasmine_model SHARED ${yasmine_model_SRC})
elseif("${CMAKE_CXX_COMPILER_ID}" STREQUAL "MSVC")
	add_library(yasmine_model STATIC ${yasmine_model_SRC})
endif()

if("${CMAKE_CXX_COMPILER_ID}" STREQUAL "GNU" OR "${CMAKE_CXX_COMPILER_ID}" STREQUAL "Clang")
	if("${CPP_VERSION}" STREQUAL "03")
		target_link_libraries (libyasmine_model LINK_PUBLIC pthread boost_system boost_thread boost_chrono)
	else()
		target_link_libraries (libyasmine_model LINK_PUBLIC pthread)
	endif()
elseif("${CMAKE_CXX_COMPILER_ID}" STREQUAL "MSVC")
	target_link_libraries (yasmine_model LINK_PUBLIC)
endif()

install(TARGETS libyasmine_model DESTINATION lib)
file(GLOB yasmine_model_hpp "include/*.hpp")
install(FILES ${yasmine_model_hpp} DESTINATION include/yasmine_model/include)
set_target_properties(libyasmine_model PROPERTIES PREFIX "")