cmake_minimum_required(VERSION 2.6)

if(EIGEN_INCLUDE_DIRS) # if compiling with PCL, it will bring Eigen with it
	message(STATUS "SfMToyLib will use Eigen")
	include_directories(${EIGEN_INCLUDE_DIRS})
	add_definitions( -DUSE_EIGEN )
endif()

set(SSBA_LIBRARY_DIR "${CMAKE_SOURCE_DIR}/3rdparty/SSBA-3.0/build" CACHE PATH "Directory to find SSBA libs")
set(USE_SSBA OFF CACHE BOOL "Build using the SSBA-3.0 library?")

if(USE_SSBA)
	link_directories(${SSBA_LIBRARY_DIR})
	include_directories("${SSBA_LIBRARY_DIR}/../")
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -DHAVE_SSBA")
endif(USE_SSBA)

set( CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wno-sign-compare")
set( CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -D__SFM__DEBUG__" )

# Core functions of SfM
file(GLOB SFM_LIB_SOURCES	
	Distance.* 
	Triangulation.* 
	FindCameraMatrices.*
	MultiCameraDistance.*
	IDistance.*
	MultiCameraPnP.*
	Common.*
	IFeatureMatcher.*
	RichFeatureMatcher.*
	OFFeatureMatcher.*
	BundleAdjuster.*
	GPUSURFFeatureMatcher.*
	AbstractFeatureMatcher.*
	SfMUpdateListener.*
)

add_library(SfMToyLibrary ${SFM_LIB_SOURCES} )
if(MSVC)
	set_target_properties(SfMToyLibrary PROPERTIES COMPILE_FLAGS "/openmp")
	if(USE_SSBA)
		target_link_libraries(SfMToyLibrary V3D COLAMD )
	endif(USE_SSBA)
endif()
if(UNIX)
	if(USE_SSBA)
		# default is to build SSBA as static libs...
		target_link_libraries(SfMToyLibrary 
			"${SSBA_LIBRARY_DIR}/libV3D.a" 
			"${SSBA_LIBRARY_DIR}/libcolamd.a"
			)
	endif(USE_SSBA)
endif()
