# Created by the script cgal_create_cmake_script
# This is the CMake script for compiling a CGAL application.


project( Sweep_line_2_Tests )

cmake_minimum_required(VERSION 2.8.10)

find_package(CGAL QUIET COMPONENTS Core )

set(CGAL_SEGMENT_TRAITS 1)
set(CGAL_SEGMENT_LEDA_TRAITS 2)
set(CGAL_POLYLINE_TRAITS 11)
set(CGAL_CONIC_TRAITS 21)

set(TRAP 1)   # Trapezoidal decomposition
set(NAIVE 2)
set(WALK 3)

if ( CGAL_FOUND )

  include( ${CGAL_USE_FILE} )

  include( CGAL_CreateSingleSourceCGALProgram )

  include_directories (BEFORE "../../include")

  function(compile name source_file point_location traits)
#    message(compile ${source_file})
    add_executable(${name} ${source_file})
    add_to_cached_list( CGAL_EXECUTABLE_TARGETS ${name} )
    if(TARGET CGAL::CGAL_Core)
      target_link_libraries(${name} CGAL::CGAL CGAL::CGAL_Core)
    endif()
    target_compile_definitions(${name} PRIVATE
      -DCGAL_ARR_TEST_POINT_LOCATION=${point_location}
      -DCGAL_ARR_TEST_TRAITS=${traits}
      )
  endfunction()

  function(compile_and_run_sweep name source_file point_location traits data_set)
    compile(${ARGV})
    if(NOT BUILD_TESTING)
      return()
    endif()
    cgal_add_compilation_test(${name})
    file(GLOB files  RELATIVE ${CMAKE_CURRENT_SOURCE_DIR} "${CMAKE_CURRENT_SOURCE_DIR}/${data_set}/*")
    foreach(file ${files})
#      message("test ${source_file} ${file}")
      string(MAKE_C_IDENTIFIER "${name}  ${file}" test_name)
#      message(" --> ${test_name}")
      cgal_add_test(${name} TEST_NAME ${test_name} ARGUMENTS ${file})
    endforeach()
  endfunction()

  compile_and_run_sweep(test_sweep test_sweep.cpp ${NAIVE} ${CGAL_SEGMENT_TRAITS} "DATA/segments_tight")
  compile_and_run_sweep(test_sweep_conic test_sweep_conic.cpp ${NAIVE} ${CGAL_CONIC_TRAITS} "DATA/conics")

else()
  
    message(STATUS "This program requires the CGAL library, and will not be compiled.")
  
endif()

