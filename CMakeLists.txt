cmake_minimum_required(VERSION 3.22)

project(r.in.kinect LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

include(build_addon)

find_M()
find_OpenMP()

# Find external dependencies specified in original Makefile
find_package(PCL 1.15 REQUIRED COMPONENTS
  common
  io
  kdtree
  octree
  search
  surface
  sample_consensus
  filters
  features
  keypoints
  segmentation
  tracking)

find_package(Eigen3 REQUIRED)
find_package(Boost REQUIRED COMPONENTS system)
find_library(K4A_LIB k4a REQUIRED)
find_library(FLANN_STATIC_LIB flann_cpp_s REQUIRED)

# Bundle PCL into an interface target
add_library(PCL_ALL INTERFACE IMPORTED)
set_target_properties(PCL_ALL PROPERTIES
  INTERFACE_INCLUDE_DIRECTORIES "${PCL_INCLUDE_DIRS}"
  INTERFACE_LINK_LIBRARIES      "${PCL_LIBRARIES}"
  INTERFACE_COMPILE_DEFINITIONS "${PCL_DEFINITIONS}")

# Define source files
file(GLOB sources "*.cpp" "*.cxx")

# Build GRASS addon using custom macro
build_addon(
  ${PROJECT_NAME}
  SOURCES ${sources}
  GRASSLIBS
    gis
    raster
    imagery
    vector
    dbmibase
    interpfl
    interpdata
    gmath
    qtree
  DEPENDS
    PCL_ALL
    Eigen3::Eigen
    Boost::system
    ${K4A_LIB}
    ${FLANN_STATIC_LIB}
    LIBM
  OPTIONAL_DEPENDS
    OpenMP::OpenMP_CXX
  DOCFILES
    ${PROJECT_NAME}.html
    ${PROJECT_NAME}.md)

# Apply extra compile flags, export settings, and RPATH from original Makefile
target_compile_options(${PROJECT_NAME} PRIVATE -O0 -march=native -Wno-deprecated)

set_target_properties(${PROJECT_NAME} PROPERTIES
  ENABLE_EXPORTS TRUE
  BUILD_RPATH "/usr/local/lib"
  INSTALL_RPATH "/usr/local/lib"
)
