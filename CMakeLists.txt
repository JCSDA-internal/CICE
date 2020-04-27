# (C) Copyright 2018 UCAR.
#
# This software is licensed under the terms of the Apache Licence Version 2.0
# which can be obtained at http://www.apache.org/licenses/LICENSE-2.0.

################################################################################
# CICE6
################################################################################

cmake_minimum_required( VERSION 3.3.2 FATAL_ERROR )

project( cice6 VERSION 6.1.0 LANGUAGES C Fortran )

set( CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/cmake;${CMAKE_MODULE_PATH} )
set( CMAKE_DIRECTORY_LABELS "cice6")

set( ECBUILD_DEFAULT_BUILD_TYPE Release )
set( ENABLE_OS_TESTS           OFF CACHE BOOL "Disable OS tests" FORCE )
set( ENABLE_LARGE_FILE_SUPPORT OFF CACHE BOOL "Disable testing of large file support" FORCE )
set( ENABLE_MPI ON CACHE BOOL "Compile with MPI" )

include( ecbuild_system NO_POLICY_SCOPE )

ecbuild_requires_macro_version( 2.7 )

################################################################################
# Project
################################################################################

ecbuild_declare_project()

ecbuild_enable_fortran( REQUIRED )

set( CICE6_LINKER_LANGUAGE Fortran )

################################################################################
# Dependencies
################################################################################

# MPI
ecbuild_add_option( FEATURE MPI DEFAULT ON
                    DESCRIPTION "Support for MPI distributed parallelism"
                    REQUIRED_PACKAGES "MPI COMPONENTS Fortran" )
ecbuild_find_mpi( COMPONENTS Fortran REQUIRED )
ecbuild_include_mpi()
link_libraries(${MPI_Fortran_LIBRARIES})

# OpenMP
if( HAVE_OMP )
  ecbuild_enable_omp()
else()
  ecbuild_enable_ompstubs()
endif()

# NetCDF
set( NETCDF_F90 ON CACHE BOOL "Compile with Fortran NetCDF" )
find_package( NetCDF REQUIRED COMPONENTS Fortran )
include_directories( ${NETCDF_INCLUDE_DIRS} )

# icepack
ecbuild_use_package( PROJECT icepack REQUIRED)
include_directories( ${ICEPACK_INCLUDE_DIRS})

################################################################################
# Definitions
################################################################################

################################################################################
# Export package info
################################################################################

set( CICE6_INCLUDE_DIRS ${CMAKE_Fortran_MODULE_DIRECTORY} )
set( CICE6_LIBRARIES cice6 )

get_directory_property( CICE6_DEFINITIONS COMPILE_DEFINITIONS )

foreach( _tpl ${CICE6_TPLS} )
  string( TOUPPER ${_tpl} TPL )
  list( APPEND CICE6_EXTRA_DEFINITIONS   ${${TPL}_DEFINITIONS}  ${${TPL}_TPL_DEFINITIONS}  )
  list( APPEND CICE6_EXTRA_INCLUDE_DIRS  ${${TPL}_INCLUDE_DIRS} ${${TPL}_TPL_INCLUDE_DIRS} )
  list( APPEND CICE6_EXTRA_LIBRARIES     ${${TPL}_LIBRARIES}    ${${TPL}_TPL_LIBRARIES}    )
endforeach()

################################################################################
# Sources
################################################################################
include( cice6_compiler_flags )

include_directories( ${CICE6_INCLUDE_DIRS} ${CICE6_EXTRA_INCLUDE_DIRS} )

add_subdirectory( cicecore )

list( APPEND cice6_src_files
    ${cicecore_files}
)

if (ECBUILD_INSTALL_FORTRAN_MODULES)
  install (DIRECTORY ${CMAKE_Fortran_MODULE_DIRECTORY}/${CMAKE_CFG_INTDIR}
    DESTINATION ${INSTALL_INCLUDE_DIR} )
endif()

################################################################################
# Library
################################################################################

ecbuild_add_library( TARGET   cice6
                     SOURCES  ${cice6_src_files}
                     LIBS     ${NETCDF_LIBRARIES} ${ICEPACK_LIBRARIES}
                     INSTALL_HEADERS LISTED
                     LINKER_LANGUAGE ${CICE6_LINKER_LANGUAGE}
                    )

################################################################################
# executable
################################################################################
set ( cice6_exe_files
    cicecore/drivers/standalone/cice/CICE.F90
)

ecbuild_add_executable( TARGET   cice6.x
                        SOURCES  ${cice6_exe_files}
                        LIBS     ${CICE6_LIBRARIES} ${ICEPACK_LIBRARIES}
                        INCLUDES ${CICE6_INCLUDES}
                        LINKER_LANGUAGE ${CICE6_LINKER_LANGUAGE}
                       )

################################################################################
# Finalise configuration
################################################################################

# prepares a tar.gz of the sources and/or binaries
ecbuild_install_project( NAME cice6 )

# print the summary of the configuration
ecbuild_print_summary()
