# (C) Copyright 2022 UCAR.
#
# This software is licensed under the terms of the Apache Licence Version 2.0
# which can be obtained at http://www.apache.org/licenses/LICENSE-2.0.

list( APPEND socacice_model_param
  ice_in
)

list( APPEND socacice_model_restarts
  grid/gx3/grid_gx3.nc
  grid/gx3/kmt_gx3.nc
  grid/gx3/global_gx3.bathy.nc
  ic/gx3/iced_gx3_v5.nc
  forcing/gx3/gx3_forcing_fields.nc
)

# forcing stuff
#├── INPUT
#│   ├── 4XDAILY -> /home/gvernier/sandboxes/CICE/test/ucar-crap/CICE_data/forcing/gx3/NCAR_bulk/4XDAILY/
#│   ├── global_gx3.bathy.nc -> /home/gvernier/sandboxes/CICE/bundle/cice6/test/grid/gx3/global_gx3.bathy.nc
#│   ├── grid_gx3.nc -> /home/gvernier/sandboxes/CICE/bundle/cice6/test/grid/gx3/grid_gx3.nc
#│   ├── gx3_forcing_fields.nc -> /home/gvernier/sandboxes/CICE/bundle/cice6/test/forcing/gx3/gx3_forcing_fields.nc
#│   ├── iced_gx3_v5.nc -> /home/gvernier/sandboxes/CICE/bundle/cice6/test/ic/gx3/iced_gx3_v5.nc
#│   ├── kmt_gx3.nc -> /home/gvernier/sandboxes/CICE/bundle/cice6/test/grid/gx3/kmt_gx3.nc
#│   ├── MONTHLY -> /home/gvernier/sandboxes/CICE/test/ucar-crap/CICE_data/forcing/gx3/NCAR_bulk/MONTHLY/



# Create Data directory for model input and symlink all files
file(MAKE_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/INPUT)
foreach(FILENAME ${socacice_model_restarts})
     get_filename_component(filename ${FILENAME} NAME )
     execute_process( COMMAND ${CMAKE_COMMAND} -E create_symlink
           ${CMAKE_CURRENT_SOURCE_DIR}/${FILENAME}
           ${CMAKE_CURRENT_BINARY_DIR}/INPUT/${filename} )
endforeach(FILENAME)

# CICE's resource files
foreach(FILENAME ${socacice_model_param})
     get_filename_component(filename ${FILENAME} NAME )
     execute_process( COMMAND ${CMAKE_COMMAND} -E create_symlink
           ${CMAKE_CURRENT_SOURCE_DIR}/${FILENAME}
           ${CMAKE_CURRENT_BINARY_DIR}/${filename} )
endforeach(FILENAME)


# CICE's output directory
file(MAKE_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/history)

# Regression test
add_test(NAME test_cice_forecast
         COMMAND mpirun -np 4 ${PROJECT_BINARY_DIR}/../bin/cice6.x
         WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR})
