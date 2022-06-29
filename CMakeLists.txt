# (C) Copyright 2018 UCAR.
#
# This software is licensed under the terms of the Apache Licence Version 2.0
# which can be obtained at http://www.apache.org/licenses/LICENSE-2.0.

################################################################################
# CICE6
################################################################################

cmake_minimum_required( VERSION 3.12 )
project( cice6 VERSION 6.1.0 LANGUAGES C Fortran )

find_package(ecbuild 3.3.2 REQUIRED)
include( ecbuild_system NO_POLICY_SCOPE )
ecbuild_declare_project()

list( APPEND CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/cmake)
include( cice6_compiler_flags )

################################################################################
# Dependencies
################################################################################
find_package(OpenMP COMPONENTS C Fortran)
find_package(MPI REQUIRED COMPONENTS C Fortran)
find_package(NetCDF REQUIRED COMPONENTS Fortran )
find_package(icepack 1.2.0 REQUIRED)


################################################################################
# Sources
################################################################################
set( CICE6_LINKER_LANGUAGE Fortran)

add_subdirectory( cicecore )

list( APPEND cice6_src_files
    ${cicecore_files}
)

ecbuild_add_library( TARGET   cice6
                     SOURCES  ${cice6_src_files}
                     INSTALL_HEADERS LISTED
                     LINKER_LANGUAGE ${CICE6_LINKER_LANGUAGE}
                    )

target_link_libraries(cice6 PUBLIC NetCDF::NetCDF_Fortran)
target_link_libraries(cice6 PUBLIC MPI::MPI_Fortran)
target_link_libraries(cice6 PUBLIC OpenMP::OpenMP_C OpenMP::OpenMP_Fortran)
target_link_libraries(cice6 PUBLIC icepack)

# Fortran module output directory for build and install interfaces
set(MODULE_DIR module/${PROJECT_NAME}/${CMAKE_Fortran_COMPILER_ID}/${CMAKE_Fortran_COMPILER_VERSION})
set_target_properties(${PROJECT_NAME} PROPERTIES Fortran_MODULE_DIRECTORY ${CMAKE_BINARY_DIR}/${MODULE_DIR})
install(DIRECTORY ${CMAKE_BINARY_DIR}/${MODULE_DIR}/ DESTINATION ${MODULE_DIR})
target_include_directories(${PROJECT_NAME} INTERFACE
                            $<BUILD_INTERFACE:${CMAKE_BINARY_DIR}/${MODULE_DIR}>
                            $<INSTALL_INTERFACE:${MODULE_DIR}>)

# Installed .h include locations
target_include_directories(cice6 PUBLIC
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/cicecore>
  $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>)


set ( cice6_exe_files
    cicecore/drivers/standalone/cice/CICE.F90
)

ecbuild_add_executable( TARGET   cice6.x
                        SOURCES  ${cice6_exe_files}
                        LIBS     cice6 icepack
                        LINKER_LANGUAGE ${CICE6_LINKER_LANGUAGE}
                       )

################################################################################
# Finalise configuration
################################################################################
ecbuild_install_project( NAME cice6 )
ecbuild_print_summary()
