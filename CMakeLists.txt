find_package(PythonLibs REQUIRED)

########
# SWIG #
########
find_package(SWIG 4.0.2 REQUIRED)
include(${SWIG_USE_FILE})

include(GenerateExportHeader)

include_directories(
    ${CMAKE_CURRENT_SOURCE_DIR}
    ${PROJECT_BINARY_DIR}
    ${PYTHON_INCLUDE_PATH}
)

###########
# Example #
###########
execute_process(COMMAND ${CMAKE_COMMAND} -E make_directory src/example)
set_source_files_properties(${CMAKE_CURRENT_SOURCE_DIR}/src/example/example.i PROPERTIES CPLUSPLUS ON)
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/src/example)
set(ADDONS_SOURCE_FILES
${CMAKE_CURRENT_SOURCE_DIR}/src/example/example.i
)

set(EXAM_SRCS
    src/example/example.h
)

set(INTERFACE_FILES
    src/example/example.i
)

set_source_files_properties(${INTERFACE_FILES} PROPERTIES CPLUSPLUS   ON)
swig_add_module(example python ${INTERFACE_FILES}
    ${EXAM_SRCS}
)

swig_link_libraries(example
    ${PYTHON_LIBRARIES}
)

##############################################
# Installation directory                     #
# by default, installed to site-packages/OCC #
##############################################
if(NOT DEFINED PYTHONOCC_INSTALL_DIRECTORY)
    if(DEFINED ENV{SP_DIR} AND WIN32)
      # TODO: following hack is azure specific, a recent update in azure
      # prevent cmake to find correct paths of python3 on windows
      # this should be removed as soon as possible
      message(STATUS "conda-build running, using $ENV{SP_DIR} as install dir")
      set(PYTHONOCC_INSTALL_DIRECTORY $ENV{SP_DIR}/OCC CACHE PATH "pythonocc install directory")
    else(DEFINED ENV{SP_DIR} AND WIN32)
      execute_process(COMMAND ${Python3_EXECUTABLE} -c "from distutils.sysconfig import get_python_lib; from os.path import relpath; print(relpath(get_python_lib(1,prefix='${CMAKE_INSTALL_PREFIX}'),'${CMAKE_INSTALL_PREFIX}'))" OUTPUT_VARIABLE python_lib OUTPUT_STRIP_TRAILING_WHITESPACE)
      set(PYTHONOCC_INSTALL_DIRECTORY ${python_lib}/OCC CACHE PATH "pythonocc install directory")
    endif(DEFINED ENV{SP_DIR} AND WIN32)
endif(NOT DEFINED PYTHONOCC_INSTALL_DIRECTORY)

# install addons
install(FILES ${BUILD_DIR}/example.py DESTINATION ${PYTHONOCC_INSTALL_DIRECTORY}/Core )
