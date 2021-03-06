cmake_minimum_required(VERSION 3.5.1)

function(find_python_exec DST)
  if (WIN32)
    set(CMD where.exe python3)
  elseif (UNIX OR APPLE)
    set(CMD which python3)
  endif()
  execute_process(
    COMMAND ${CMD}
    OUTPUT_VARIABLE OUT
    RESULT_VARIABLE RES
    OUTPUT_STRIP_TRAILING_WHITESPACE
    )
  set(${DST} ${OUT} PARENT_SCOPE)
endfunction()

function(build_test SRCFILE)
  get_filename_component(src_name ${SRCFILE} NAME_WE)
  set(target "${src_name}")
  add_executable(${target} ${SRCFILE})
  target_link_libraries(
    ${target}
    PRIVATE
    gtn
    )
  target_include_directories(
    ${target}
    PRIVATE
    ${PROJECT_SOURCE_DIR}/..
    )
  add_test(${target} ${target})
endfunction(build_test)

if (GTN_BUILD_TESTS)
  build_test(${PROJECT_SOURCE_DIR}/test/autograd_test.cpp)
  build_test(${PROJECT_SOURCE_DIR}/test/creations_test.cpp)
  build_test(${PROJECT_SOURCE_DIR}/test/criterion_test.cpp)
  build_test(${PROJECT_SOURCE_DIR}/test/functions_test.cpp)
  build_test(${PROJECT_SOURCE_DIR}/test/parallel_test.cpp)
  build_test(${PROJECT_SOURCE_DIR}/test/graph_test.cpp)
  build_test(${PROJECT_SOURCE_DIR}/test/utils_test.cpp)
  build_test(${PROJECT_SOURCE_DIR}/test/rand_test.cpp)
  if (GTN_BUILD_PYTHON_BINDINGS)
    find_python_exec(PYTHON_EXEC_PATH)
    add_test(
      NAME PythonBindingsTest
      COMMAND ${PYTHON_EXEC_PATH} -m unittest discover -v ${PROJECT_SOURCE_DIR}/bindings/python/test
    )
  endif()
endif ()
