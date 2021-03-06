cmake_minimum_required( VERSION 2.8 )
project( SFCGAL )

set( CMAKE_DEBUG_POSTFIX "d" )

#-- include finders and co
set( CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}/cmake/Modules;${CMAKE_MODULE_PATH}" )


#-----------------------------------------------------------
# dependencies
#-----------------------------------------------------------

#-- BOOST --------------------------------------------------
option( Boost_USE_AUTO_LINK "boost use autolink" OFF )
if( NOT ${Boost_USE_AUTO_LINK} )
	add_definitions( "-DBOOST_ALL_NO_LIB" )
endif()

find_package( Boost COMPONENTS unit_test_framework thread REQUIRED )
if( ${Boost_USE_STATIC_LIBS} )
	add_definitions( "-DBOOST_THREAD_USE_LIB" )
endif()


#-- GMP (facultative)  -------------------------------------
find_package( GMP )
if( ${GMP_FOUND} )
	message( STATUS "GMP_INCLUDE_DIRS = ${GMP_INCLUDE_DIRS}" )
	message( STATUS "GMP_LIBRARIES = ${GMP_LIBRARIES}" )
	set( EPLU_WITH_GMP ON )
endif()



#-- MPFR (needs GMP)  --------------------------------------
if( ${GMP_FOUND} )
	find_package( MPFR )
	if( ${MPFR_FOUND} )
		message( STATUS "MPFR_INCLUDE_DIRS = ${MPFR_INCLUDE_DIRS}" )
		message( STATUS "MPFR_LIBRARIES = ${MPFR_LIBRARIES}" )
		set( EPLU_WITH_MPFR ON )
	endif()
endif()

if( NOT ${GMP_FOUND} OR NOT ${MPFR_FOUND} )
	message( WARNING "can't find GMP and MPFR, using default kernel" ) 
	set( EPLU_USE_CARTESIAN_KERNEL ON )
endif()


#-- find CGAL  ---------------------------------------------
option( CGAL_USE_AUTOLINK "disable CGAL autolink" OFF )
if( ${CGAL_USE_AUTOLINK} )
	add_definitions( "-DCGAL_NO_AUTOLINK" )
endif()

find_package( CGAL COMPONENTS Core REQUIRED )
message( STATUS "CGAL ${CGAL_VERSION} found" )

include_directories( 
	include
	${CGAL_INCLUDE_DIRS}
	${Boost_INCLUDE_DIRS}	
)

add_definitions( "-DBOOST_TEST_DYN_LINK" )
if(MSVC)
	# Force to always compile with W4
	if(CMAKE_CXX_FLAGS MATCHES "/W[0-4]")
		string(REGEX REPLACE "/W[0-4]" "/W4" CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS}")
	else()
		set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /W4")
	endif()
elseif(CMAKE_COMPILER_IS_GNUCC OR CMAKE_COMPILER_IS_GNUCXX)
	# Update if necessary
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wno-long-long -pedantic")
endif()


#-- build the library
file( GLOB_RECURSE SFCGAL_HEADERS include/*.h )
file( GLOB_RECURSE SFCGAL_SOURCES src/*.cpp )

if(CMAKE_COMPILER_IS_GNUCXX)
  add_definitions( "-frounding-math" )
endif(CMAKE_COMPILER_IS_GNUCXX)

add_library(
	SFCGAL SHARED
	${SFCGAL_HEADERS}
	${SFCGAL_SOURCES}
)
install( TARGETS SFCGAL DESTINATION lib )
INSTALL(DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/include" DESTINATION .)

#-- build test (todo only if boost use dyn link)

file( GLOB_RECURSE SFCGAL_TEST_SOURCES test/*.cpp )
add_executable( test-SFCGAL ${SFCGAL_TEST_SOURCES} )
target_link_libraries( test-SFCGAL 
	SFCGAL
	${Boost_LIBRARIES}
	${CGAL_LIBRARIES}
)
set_target_properties( test-SFCGAL PROPERTIES DEBUG_POSTFIX "d" )
install( TARGETS test-SFCGAL DESTINATION bin )









