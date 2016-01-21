cmake_minimum_required(VERSION 2.8.3)
project(bayes_tracking)
set(CPACK_PACKAGE_NAME "bayes_tracking")
set(VERSION "1.0.3")

## Find catkin macros and libraries if installed
find_package(catkin QUIET)
## Use catkin macros and include dirs
if(catkin_FOUND)
  message(STATUS "Found catkin")
  catkin_package(
    INCLUDE_DIRS include 
    LIBRARIES ${PROJECT_NAME}
  )
  include_directories(
    ${catkin_INCLUDE_DIRS}
  )
else(catkin_FOUND)
    message(STATUS "NOT found catkin")
endif(catkin_FOUND)

find_package(LAPACK)

## Headers
include_directories(
    include
)

OPTION (BAYESTRACKING_BUILD_TRACKINGWIN "Build the tracking window" ON)

if(BAYESTRACKING_BUILD_TRACKINGWIN)
	find_package(OpenCV)

	if(OpenCV_FOUND)
	  set(TRACKWIN "src/bayes_tracking/trackwin.cpp")
	  set(EXCLUDE_TRACKWIN_HEADER "1337")
	else()
	  set(EXCLUDE_TRACKWIN_HEADER "trackwin.h")
	  message(ERROR "Could not find OpenCV! Build with BAYESTRACKING_BUILD_TRACKINGWIN = OFF or give OpenCV_DIR!")
	endif(OpenCV_FOUND)
endif()

## Source files
add_library(${PROJECT_NAME} SHARED
    src/bayes_tracking/associationmatrix.cpp 
    src/bayes_tracking/ekfilter.cpp 
    src/bayes_tracking/ukfilter.cpp 
    src/bayes_tracking/pfilter.cpp 
    src/bayes_tracking/models.cpp
    src/bayes_tracking/BayesFilter/bayesFltAlg.cpp
    src/bayes_tracking/BayesFilter/bayesFlt.cpp
    src/bayes_tracking/BayesFilter/CIFlt.cpp
    src/bayes_tracking/BayesFilter/covFlt.cpp
    src/bayes_tracking/BayesFilter/infFlt.cpp
    src/bayes_tracking/BayesFilter/infRtFlt.cpp
    src/bayes_tracking/BayesFilter/itrFlt.cpp
    src/bayes_tracking/BayesFilter/matSup.cpp
    src/bayes_tracking/BayesFilter/SIRFlt.cpp
    src/bayes_tracking/BayesFilter/UDFlt.cpp
    src/bayes_tracking/BayesFilter/UdU.cpp
    src/bayes_tracking/BayesFilter/unsFlt.cpp
    ${TRACKWIN}
)

target_link_libraries(${PROJECT_NAME} ${OpenCV_LIBS} ${LAPACK_LIBRARIES})

## Link catkin libraries and set install targets
if(catkin_FOUND)
  target_link_libraries(${PROJECT_NAME}
    ${catkin_LIBRARIES}
  )

  install(TARGETS ${PROJECT_NAME}
    ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
    LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
    RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
  )

  install(DIRECTORY include/${PROJECT_NAME}/
    DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION}
    PATTERN ".svn" EXCLUDE
    PATTERN ${EXCLUDE_TRACKWIN_HEADER} EXCLUDE
  )
endif(catkin_FOUND)


## for the rest of the world
if(WIN32 AND NOT CYGWIN)
	set(DEF_INSTALL_CMAKE_DIR CMake)
else()
	set(DEF_INSTALL_CMAKE_DIR lib/Bayestracking-${VERSION})
endif()

set(INSTALL_CMAKE_DIR ${DEF_INSTALL_CMAKE_DIR} CACHE PATH
"Installation directory for CMake files")

install(TARGETS ${PROJECT_NAME}
	EXPORT BayestrackingTargets
	DESTINATION lib)

install(DIRECTORY include/${PROJECT_NAME}
	DESTINATION include/
	PATTERN ".svn" EXCLUDE
	PATTERN ${EXCLUDE_TRACKWIN_HEADER} EXCLUDE)
    

export(TARGETS ${PROJECT_NAME}
	FILE "${PROJECT_BINARY_DIR}/BayestrackingTargets.cmake")

export(PACKAGE ${PROJECT_NAME})

install(EXPORT BayestrackingTargets DESTINATION
  "${INSTALL_CMAKE_DIR}" COMPONENT dev)

configure_file("config/BayestrackingConfig.cmake.in"
       "${PROJECT_BINARY_DIR}/BayestrackingConfig.cmake"
       @ONLY)
configure_file("config/BayestrackingConfigVersion.cmake.in"
       "${PROJECT_BINARY_DIR}/BayestrackingConfigVersion.cmake"
       @ONLY)
configure_file("config/bayestracking.pc.in"
       "${PROJECT_BINARY_DIR}/bayestracking.pc"
       @ONLY)

install(FILES
	"${PROJECT_BINARY_DIR}/BayestrackingConfig.cmake"
	"${PROJECT_BINARY_DIR}/BayestrackingConfigVersion.cmake"
	DESTINATION "${INSTALL_CMAKE_DIR}" COMPONENT dev)

OPTION (BAYESTRACKING_BUILD_EXAMPLES "Build examples" ON)
if(BAYESTRACKING_BUILD_EXAMPLES)
	add_subdirectory(examples)
endif(BAYESTRACKING_BUILD_EXAMPLES)

## Optional builds: documentation
OPTION (BAYESTRACKING_BUILD_DOC "Generates API documentation" OFF)
if(BAYESTRACKING_BUILD_DOC)
  ## Check if doxygen is even installed
  find_package(Doxygen)
  if (DOXYGEN_FOUND STREQUAL "NO")
      message(FATAL_ERROR "Doxygen not found. Please get a copy http://www.doxygen.org")
  endif (DOXYGEN_FOUND STREQUAL "NO")

  ## Prepare doxygen configuration file
  configure_file(${CMAKE_SOURCE_DIR}/Doxyfile.in ${CMAKE_CURRENT_BINARY_DIR}/Doxyfile)

  ## Add doxygen as target
  add_custom_target(doc ALL ${DOXYGEN_EXECUTABLE} ${CMAKE_CURRENT_BINARY_DIR}/Doxyfile
  COMMENT "Generating API documentation with Doxygen" VERBATIM)

  ## Cleanup $build/doc on "make clean"
  set_property(DIRECTORY APPEND PROPERTY
        ADDITIONAL_MAKE_CLEAN_FILES doc)

  ## Install HTML API documentation and manual pages
  set(DOC_PATH "share/doc/${CPACK_PACKAGE_NAME}-${VERSION}")

  install(DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/doc/html
    DESTINATION ${DOC_PATH}
  )

endif(BAYESTRACKING_BUILD_DOC)
