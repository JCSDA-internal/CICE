# (C) COPYRIGHT 2018-2020 UCAR
#
# THIS SOFTWARE IS LICENSED UNDER THE TERMS OF THE APACHE LICENCE VERSION 2.0
# WHICH CAN BE OBTAINED AT HTTP://WWW.APACHE.ORG/LICENSES/LICENSE-2.0.

#
# bundle for CICE6
#
cmake_minimum_required( VERSION 3.12 )
project( cice6-bundle VERSION 0.0.0 LANGUAGES C CXX Fortran )

find_package( ecbuild 3.3.2 REQUIRED)
include( ecbuild_bundle )

ecbuild_bundle_initialize()

ecbuild_bundle( PROJECT jedicmake GIT "https://github.com/jcsda-internal/jedi-cmake.git" UPDATE BRANCH develop )
include( jedicmake/cmake/Functions/git_functions.cmake  )


# required repositories

ecbuild_bundle( PROJECT icepack GIT "https://github.com/jcsda-internal/icepack.git" UPDATE BRANCH feature/ecbuild-new )
ecbuild_bundle( PROJECT cice6            SOURCE "../" )

ecbuild_bundle_finalize()
