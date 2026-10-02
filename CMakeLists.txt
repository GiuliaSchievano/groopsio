cmake_minimum_required(VERSION 3.16)

project(groopsio LANGUAGES CXX Fortran)
set(CMAKE_CXX_STANDARD 17)

# Find Python 3 and its development files
find_package(Python3 REQUIRED COMPONENTS Interpreter Development)

# Find NumPy using the Python interpreter
execute_process(COMMAND ${Python3_EXECUTABLE} -c "import numpy; print(numpy.get_include())" OUTPUT_VARIABLE NUMPY_INCLUDE_DIR OUTPUT_STRIP_TRAILING_WHITESPACE)

# Find EXPAT
find_package(EXPAT REQUIRED)
include_directories(${EXPAT_INCLUDE_DIRS})

# Find BLAS
find_package(BLAS REQUIRED)

# Find zlib
find_library(LIB_Z z)

if(LIB_Z)
    find_path(ZLIB_INCLUDE_DIR NAMES zlib.h)
    include_directories(${ZLIB_INCLUDE_DIR})
else()
    add_definitions(-DNOLIB_Z)
    message(WARNING "Z library NOT found (https://www.zlib.net). groopsio will not be able to read/write compressed *.gz files.")
endif()


set(BASE_LIBRARIES ${EXPAT_LIBRARIES} ${BLAS_LIBRARIES} ${LIB_Z} ${Python_LIBRARIES} stdc++fs)

# Read the list of GROOPS source files
include(sources.list)

set(GROOPS_SOURCE_DIR ""CACHE PATH "path to GROOPS source directory")

include_directories(${GROOPS_SOURCE_DIR})

list(TRANSFORM SOURCES PREPEND ${GROOPS_SOURCE_DIR}//)

# Name of the Python extension library
set(EXTENSION_LIBRARY_NAME "groopsiobase" CACHE STRING "name of the Python extension library")

# Build the shared library
add_library(${EXTENSION_LIBRARY_NAME} SHARED ${PROJECT_SOURCE_DIR}/src/groopsio.cpp ${SOURCES})

# Make the library name "groopsiobase" without lib prefix or .so suffix
SET_TARGET_PROPERTIES(${EXTENSION_LIBRARY_NAME} PROPERTIES SUFFIX "")
SET_TARGET_PROPERTIES(${EXTENSION_LIBRARY_NAME} PROPERTIES PREFIX "")

# Add NumPy headers
target_include_directories(${EXTENSION_LIBRARY_NAME} PRIVATE ${NUMPY_INCLUDE_DIR})

# Link against Python, EXPAT, BLAS and zlib
target_link_libraries(${EXTENSION_LIBRARY_NAME} PRIVATE ${EXPAT_LIBRARIES} ${BLAS_LIBRARIES} ${LIB_Z} Python3::Python)
